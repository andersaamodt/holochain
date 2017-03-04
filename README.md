# Holochain

 ![Code Status](https://img.shields.io/badge/Code-Pre--Alpha-orange.svg) [![Travis](https://img.shields.io/travis/metacurrency/holochain.svg)](https://travis-ci.org/metacurrency/holochain) [![Go Report Card](https://goreportcard.com/badge/github.com/metacurrency/holochain)](https://goreportcard.com/report/github.com/metacurrency/holochain) [![Gitter](https://badges.gitter.im/metacurrency/holochain.svg)](https://gitter.im/metacurrency/holochain?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=body_badge) [![In Progress](https://badge.waffle.io/metacurrency/holochain.svg?label=in%20progress&title=In%20Progress)](http://waffle.io/metacurrency/holochain)

**Holographic storage for distributed applications.** A holochain is a monotonic distributed hash table (DHT) where every node enforces validation rules on data before publishing that data against the signed chains where the data originated.

In other words, a holochain functions very much **like a blockchain without bottlenecks** when it comes to enforcing validation rules, but is designed to  be fully distributed with each node only needing to hold a small portion of the data instead of everything needing a full copy of a global ledger. This makes it feasible to run blockchain-like applications on devices as lightweight as mobile phones.

**[Code Status:](https://github.com/metacurrency/holochain/milestones?direction=asc&sort=due_date&state=all)** Active development for **proof-of-concept stage**. Pre-alpha. Not for production use. We still expect to destructively restructure data chains at this time.
<br/>

| Holochain Links: | [FAQ](https://github.com/metacurrency/holochain/wiki/FAQ) | [Developer Wiki](https://github.com/metacurrency/holochain/wiki) | [White Paper](http://ceptr.org/projects/holochain) | [GoDocs](https://godoc.org/github.com/metacurrency/holochain) |
|---|---|---|---|---|

**Table of Contents**
<!-- TOC START min:2 max:4 link:true update:true -->
  - [Installation](#installation)
  - [Usage](#usage)
  - [Architecture](#architecture)
    - [Functional Domains](#functional-domains)
      - [Group DNA / Holochain configuration](#group-dna--holochain-configuration)
      - [Individuals Authoring Content](#individuals-authoring-content)
      - [Application API](#application-api)
    - [Two Distinct SubSystems](#two-distinct-subsystems)
      - [1. Authoring your Local Chain](#1-authoring-your-local-chain)
      - [2. Running a DHT Node](#2-running-a-dht-node)
  - [Documentation](#documentation)
  - [Testing](#testing)
  - [Development --](#development---)
    - [Contributor Guidelines](#contributor-guidelines)
      - [Tech](#tech)
      - [Social --](#social---)
  - [License](#license)
  - [Acknowledgements](#acknowledgements)

<!-- TOC END -->

## Installation

1. Make sure you have a working environment set up for the Go language version 1.7 or later. [See the installation instructions for Go](http://golang.org/doc/install.html). Follow their instructions for setting your $GOPATH too.
2. Install the [gx package manager](https://github.com/whyrusleeping/gx):
```
$ go get -u github.com/whyrusleeping/gx
```
3. Then you can install the holochain command line interface with:
```
$ go get -d github.com/metacurrency/holochain
$ cd $GOPATH/src/github.com/metacurrency/holochain
$ make
```

Make sure your `PATH` includes the `$GOPATH/bin` directory so the program it builds can be easily called:
```
$ export PATH=$PATH:$GOPATH/bin
```
## Installation on Windows

1. Install Go 1.7.5](https://golang.org/dl/).
2. Install [Windows git](https://git-scm.com/downloads). Be sure to select the appropriate options so that git is accessible from the Windows command line.
3. Install [GnuWin32 make](http://gnuwin32.sourceforge.net/packages/make.htm).Add C:\Program Files (x86)\GnuWin32\bin to your PATHS directory. (Make sure C:\go\bin is in your PATHS directory already, too)
4. Click Start, type "System" and press Enter. Click "Advanced system settings" in the sidebar. Click "Environment Variables...". Under System Variables, click New..., and put GOPATH as the name, and the path to your Go installation in the value (usually C:\go).
5. Now, double-click PATH under System Variables, and click New in the window that pops up. Add the path to go's bin directory as the value (usually C:\go\bin). This will allow you to run compiled executables from anywhere in the Windows command line.
6. Now click New again time and add the path to your GnuWin32 make bin directory (usually C:\Program Files (x86)\GnuWin32\bin).
5. Follow the remaining instructions starting at step 2 above. You should be able to use 'go' and 'make' from the Windows command line. (Add -x to the Go 'get' command to see verbose output as the packages download.)

## Usage

Once you've gotten everything working as described above you can execute some basic holochain commands from the command line like this:

    hc help

Since holochain is basically a distributed database engine, you will probably only do some basic maintenance through the command line. To initialize holochain service and build the directories, files, and generates public/private keys:

    hc init '"Fred Flinstone" <fred@flintsone.com>'

You can use a pre-existing holochain configuration by replacing SOURCE with path for loading existing DNA.  You can use a live chain from your .holochain directory, or one of the templates in the examples directory:

    hc gen from <SOURCE_PATH> <NAME>

If you are a developer and want to build your own group configuration, data schemas, and validation rules for a holochain you can set up the initial scaffolding and files with:

    hc gen dev <NAME> [<FORMAT>]

`<FORMAT>` is the encoding type of your DNA file which can be one of `yaml`, `json` or `toml`.

To aid development, the `gen dev` command also produces a sample `test` sub-directory with exposed function calls of the format `<test_num>.json`

The command:

    hc test <NAME>

runs those test functions.  Thus you can use this command as you make changes to your holochain DNA files to aid in test-driven-development.

After you have cloned or completed development for a chain, you can start the chain (i.e. create the genesis entries) with:

    hc gen chain <NAME>

Then you serve it via http on localhost with:

    hc serve <NAME> [<PORT>]

To view all the chains on your system and their status, use:

    hc status

You can inspect the contents of a particular chain with:

    hc dump <NAME>

By default `hc` stores all holochain data and configuration files to the `~/.holochain` directory.  You can override this with the -path flag or by setting the `HOLOPATH` environment variable, e.g.:

    hc -path ~/mychains init '<my@other.identity>'
    HOLOPATH=~/mychains hc

Note that you can use the form:

    hc -path=/your/path/here

but beware as that form will not do shell substitutions (i.e. ~ won't get converted to your home dir)

## Architecture
### Functional Domains
Holochains, by design, should be used in the context of a group operating by a shared set of agreements. Generally speaking, you don't need a holochain if you are just managing your own data.

These agreements are encoded in the validation rules which are checked before authoring to one's local chain, and are also checked by every DHT node asked to publish the new data.

In essence these ensure holochain participants operate according the same rules. Just like in blockchains, if you collude to break validation rules, you essentially have forked the chain. If you commit things to your chain, or try to publish things which don't comply with the validation rules, the rest of the network/DHT rejects it.

#### Group DNA / Holochain configuration
At this stage, a developer needs to set up the technical configuration of the collective agreements enforced by a holochain. This includes such things as: the holochain name, UUID, address & name spaces, data schemas, validation rules for chain entries and data propagation on the DHT,

#### Individuals Authoring Content
As an individual, you can join a holochain by installing its holochain configuration and configuring your ID, keys, chain, and DHT node in accord with the DNA specs.

#### Application API
Holochains function like a database. They don't have much end-user interface, and are primarily used by an application or program to store data. Unless you're a developer building one of these applications, you're not likely to interact directly with a holochain. Hopefully, you install an application that does all that for you and the holochain stays nice and invisible enabling the application to store its information in a decentralized manner.

### Two Distinct SubSystems
There are two modes to participate in a holochain: as a **chain author**, and as a **DHT node**. We expect most installations will be doing both things and acting as full peers in a P2P data system. However, each could be run in a separate container, communicating only by network interface.

#### 1. Authoring your Local Chain
Your chain is your signed, sequential record of the data you create to share on the holochain. Depending on the holochain's validation rules, this data may also be immutable and non-repudiable. Your local chain/data-store follows this pattern:

1. Validates your new data
2. Stores the data in a new chain entry
3. Signs it to your chain
4. Indexes the content
5. Shares it to the DHT
6. Responds to validation requests from DHT nodes

#### 2. Running a DHT Node
For serving data shared across the network. When your node receives a request from another node to publish DHT data, it will first validate the signatures, chain links, and any other application specific data integrity in the entity's source chain who is publishing the data.

## Documentation

Find additional documentation in the [Holochain Wiki](https://github.com/metacurrency/holochain/wiki).

You can also find the [auto-generated Reference API for Holochain on GoDocs](https://godoc.org/github.com/metacurrency/holochain)

## Development -- [![In Progress](https://badge.waffle.io/metacurrency/holochain.svg?label=in%20progress&title=In%20Progress)](http://waffle.io/metacurrency/holochain)
We accept Pull Requests and welcome your participation.
 * [Milestones](https://github.com/metacurrency/holochain/milestones?direction=asc&sort=due_date&state=all) and progress on Roadmap
 * Kanban on [Waffle](https://waffle.io/metacurrency/holochain) of GitHub issues
 * Or [chat with us on gitter](https://gitter.im/metacurrency/holochain)

[![Throughput Graph](http://graphs.waffle.io/metacurrency/holochain/throughput.svg)](https://waffle.io/metacurrency/holochain/metrics)

### Dependencies

This project depends on various parts of [libp2p](https://github.com/libp2p/go-libp2p), which uses the [gx](https://github.com/whyrusleeping/gx) package manager.  This means that installation doesn't follow the normal "go get" process but instead also requires a make step.  Thus, to install the code and dependencies run:

    go get github.com/metacurrency/holochain/
    make deps

If you already installed the hc command line interface the dependencies will have been installed, and this step is unnecessary.

Note that `make` and `make deps` have a side-effect of re-writing some of the imports in various files.  This is how `gx` handles dependencies on specific versions of go imports.  But this means that when you are ready to make commits to your repo, you must undo these re-writes so they don't get committed to the repo.  You can do this with:

    make publish

After you have made your commit and are ready to continue working, you can redo those rewrites without re-running the full dependency install with:

    make work

### Tests

To compile and run all the tests:

    cd $GOPATH/github.com/metacurrency/holochain
    make test

Or if you have already done the initial `make` or `make deps` step, you can simply use `go test` as usual.

### Contributor Guidelines

#### Tech
* We use **test driven development**. Adding a new function or feature, should mean you've added the tests that make sure it works.
* Set your editor to automatically use [gofmt](https://blog.golang.org/go-fmt-your-code) on save so there's no wasted discussion on proper indentation of brace style!
* [Contact us](https://gitter.im/metacurrency/holochain) to set up a **pair coding session** with one of our developers to learn the lay of the land
* **join our dev documentation calls** twice weekly on Tuesdays and Fridays.

#### Social -- [![Twitter Follow](https://img.shields.io/twitter/follow/holochain.svg?style=social&label=Follow)](https://twitter.com/holochain)
<!-- * Protocols for Inclusion. -->
We are committed to foster a vibrant thriving community, including growing a culture that breaks cycles of marginalization and dominance behavior. In support of this, some open source communities adopt [Codes of Conduct](http://contributor-covenant.org/version/1/3/0/).  We are still working on our social protocols, and empower each team to describe its own <i>Protocols for Inclusion</i>.  Until our teams have published their guidelines, please use the link above as a general guideline.

## License
[![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](http://www.gnu.org/licenses/gpl-3.0)  Copyright (C) 2017, The MetaCurrency Project (Eric Harris-Braun, Arthur Brock, et. al.)

This program is free software: you can redistribute it and/or modify it under the terms of the license provided in the LICENSE file (GPLv3).  This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

**Note:** We are considering other 'looser' licensing options (like MIT license) but at this stage are using GPL while we're getting the matter sorted out.

## Acknowledgements
* **MetaCurrency & Ceptr**: Holochains are a sub-project of [Ceptr](http://ceptr.org) which is a semantic, distributed computing platform under development by the [MetaCurrency Project](http://metacurrency.org).
&nbsp;
* **Ian Grigg**: Some of our initial plans for this architecture were inspired in 2006 by [his paper about Triple Entry Accounting](http://iang.org/papers/triple_entry.html) and his work on [Ricardian Contracts](http://iang.org/papers/ricardian_contract.html).
<!-- * **Juan Benet**: For all his work on IPFS and being a generally cool guy. The libP2P library has been extremely helpful in getting our peered node communications running. -->
* **Crypto Pioneers** And of course the people who paved the road before us by writing good crypto libraries and **preaching the blockchain gospel**. Nobody understood what we were talking about when we started sharing our designs. The main reason people want it now, is because blockchains have opened their eyes to new patterns of power available from decentralized architectures.
