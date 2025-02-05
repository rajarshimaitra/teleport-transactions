<div align="center">

<h1><img alt="/logo" src="https://raw.githubusercontent.com/utxo-teleport/teleport-transactions/master/assets/logo.png" width="25" style="margin:-4px 4px" />Teleport Transactions</h1>

<p>
    A Taker library with minimal API for performing coinswaps. A Maker binary with minimal config to deploy swap-service demons.
  </p>

<p>
    <!--
    <a href="https://crates.io/crates/coinswap"><img alt="Crate Info" src="https://img.shields.io/crates/v/coinswap.svg"/></a>
    <a href="https://docs.rs/coinswap"><img alt="API Docs" src="https://img.shields.io/badge/docs.rs-coinswap-green"/></a>
    -->
    <a href="https://github.com/utxo-teleport/teleport-transactions/blob/master/LICENSE.md"><img alt="CC0 1.0 Universal Licensed" src="https://img.shields.io/badge/license-CC0--1.0-blue.svg"/></a>
    <a href="https://github.com/utxo-teleport/teleport-transactions/actions/workflows/build.yaml"><img alt="CI Status" src="https://github.com/utxo-teleport/teleport-transactions/actions/workflows/build.yaml/badge.svg"></a>
    <a href="https://github.com/utxo-teleport/teleport-transactions/actions/workflows/lint.yaml"><img alt="CI Status" src="https://github.com/utxo-teleport/teleport-transactions/actions/workflows/lint.yaml/badge.svg"></a>
    <a href="https://github.com/utxo-teleport/teleport-transactions/actions/workflows/test.yaml"><img alt="CI Status" src="https://github.com/utxo-teleport/teleport-transactions/actions/workflows/test.yaml/badge.svg"></a>
    <a href="https://codecov.io/github/utxo-teleport/teleport-transactions?branch=master">
    <img alt="Coverage" src="https://codecov.io/github/utxo-teleport/teleport-transactions/coverage.svg?branch=master">
    </a>
    <a href="https://blog.rust-lang.org/2023/12/28/Rust-1.75.0.html"><img alt="Rustc Version 1.75.0+" src="https://img.shields.io/badge/rustc-1.75.0%2B-lightgrey.svg"/></a>
  </p>
</div>

> \[!WARNING\]
> This library is currently under beta development and at an experimental stage. There are known and unknown bugs. Mainnet use is strictly NOT recommended.

## Table of Contents

- [Table of Contents](#table-of-contents)
- [About](#about)
- [Architecture](#architecture)
- [Build and Run](#build-and-run)
- [Project Status](#project-status)
- [Roadmap](#roadmap)
  - [V 0.1.0](#v-010)
  - [V 0.1.\*](#v-01)
- [Community](#community)

## About

Teleport Transactions is a rust implementation of a variant of atomic-swap protocol, using HTLCs on Bitcoin. Read more at:

* [Mailing list post](https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2020-October/018221.html)
* [Detailed design](https://gist.github.com/chris-belcher/9144bd57a91c194e332fb5ca371d0964)
* [Developer's resources](/docs/dev-book.md)
* [Run demo](/docs/demo.md)

## Architecture

The project is divided into distinct modules, each focused on specific functionalities.

```console
docs/
src/
├─ bin/
├─ maker/
├─ market/
├─ protocol/
├─ scripts/
├─ taker/
├─ wallet/
├─ watchtower/
tests/
```
| Directory           | Description |
|---------------------|-------------|
| **`doc`**           | Contains all the project-related docs. The [dev-book](./docs/dev-book.md) includes major developer salient points and the [demo doc](./docs/demo.md) describes how to run the `teleport` binary and perform a swap in regtest.|
| **`tests`**         | Contains integration tests. Describes behavior of various abort/malice cases.|
| **`src/taker`**     | Taker module houses its core logic in `src/taker/api.rs` and handles both Taker-related behaviors and most of the protocol-related logic. |
| **`src/maker`**     | Encompasses Maker-specific logic and plays a relatively passive role compared to Taker. |
| **`src/wallet`**    | Manages wallet-related operations, including storage and blockchain interaction. |
| **`src/market`**    | Handles market-related logic, where Makers post their offers. |
| **`src/watchtower`**| Provides a Taker-offloadable watchtower implementation for monitoring contract transactions. |
| **`src/scripts`**   | Offers simple scripts to utilize library APIs in the `teleport` app. |
| **`src/bin`**       | Houses deployed project binaries. |
| **`src/protocol`**  | Contains utility functions, error handling, and messages for protocol communication. |


## Build and Run

The project follows the standard Rust build workflow and generates a CLI app named `teleport`.

```console
$ cargo build
```

The project includes both integration and unit tests. The integration tests simulates various edge cases of the coinswap protocol.

To run the tests:

```console
$ cargo test -- --nocapture
```

For manual swaps using the `teleport` app, follow the instructions in [demo](/docs/demo.md).

For in-depth developer documentation on protocol workflow and implementation, consult the [dev book](/docs/dev-book.md).

## Project Status

The project is currently in a pre-alpha stage, intended for demonstration and prototyping. The protocol has various hard-coded configuration variables and known/unknown bugs. Basic swap protocol functionality works on `regtest` and `signet` networks, but it's not recommended for `mainnet` use.

If you're interested in contributing to the project, explore the [open issues](https://github.com/utxo-teleport/teleport-transactions/issues) and submit a PR.

## Roadmap

### V 0.1.0

- [X] Basic protocol workflow with integration tests.
- [X] Modularize protocol components.
- [X] Refine logging information.
- [X] Abort 1: Taker aborts after setup. Makers identify this, and gets their fund back via contract tx.
- [X] Abort 2: One Maker aborts **before setup**. Taker retaliates by banning the maker, moving on with other makers, if it can't find enough makers, then recovering via contract transactions.
  - [X] Case 1: Maker drops **before** sending sender's signature. Taker tries with another Maker and moves on.
  - [X] Case 2: Maker drops **before** sending sender's signature. Taker doesn't have any new Maker. Recovers from swap.
  - [X] Case 3: Maker drops **after** sending sender's signatures. Taker doesn't have any new Maker. Recovers from swap.
- [X] Build a flexible Test-Framework with `bitcoind` backend.
- [X] Abort 3: Maker aborts **after setup**. Taker and other Makers identify this and recovers back via contract tx. Taker bans the aborting Maker's fidelity bond.
  - [X] Case 1: Maker Drops at `ContractSigsForRecvrAndSender`. Does not broadcasts the funding txs. Taker and Other Maker recovers. Maker gets banned.
  - [X] Case 2: Maker drops at `ContractSigsForRecvr` after broadcasting funding txs. Taker and other Makers recover. Maker gets banned.
  - [X] Case 3: Maker Drops at `HashPreimage` message and doesn't respond back with privkeys. Taker and other Maker recovers. Maker gets banned.
- [X] Malice 1: Taker broadcasts contract immaturely. Other Makers identify this, get their funds back via contract tx.
- [X] Malice 2: One of the Makers broadcast contract immaturely. The Taker identify this, bans the Maker's fidelity bond, other Makers get back funds via contract tx.
- [X] Fix all clippy warnings.
- [x] Implement configuration file i/o support for Takers and Makers.
- [x] Switch to binary encoding for network messages.
- [x] Switch to binary encoding for wallet data.
- [ ] Make tor detectable and connectable by default for Maker and Taker. And Tor configs to their config lists.
- [ ] Complete all unit tests in modules.
- [ ] Achieve >80% crate level test coverage ratio (including integration tests).
- [x] Clean up and integrate fidelity bonds with maker banning.
- [ ] Sketch a simple `AddressBook` server. Tor must. This is for MVP. Later on we will move to more decentralized address server architecture.
- [ ] Turn maker server into a `makerd` binary, and a `maker-cli` rpc controller app, with MVP API.
- [ ] Finalize the Taker API for downstream wallet integration.
- [ ] Develop an example web Taker client with a downstream wallet.
- [ ] Package `makerd` and `maker-cli` in a downstream node.
- [ ] Release v0.1.0 in Signet for beta testing.

### V 0.1.*

- [ ] Implement UTXO merging and branch-out via swap for improved UTXO management.
- [ ] Describe contract and funding transactions via miniscript, using BDK for wallet management.
- [ ] Enable wallet syncing via CBF (BIP157/158).
- [ ] Transition to taproot outputs for the entire protocol, enhancing anonymity and obfuscating contract transactions.
- [ ] Implement customizable wallet data storage (SQLite, Postgres).
- [ ] Optional: Payjoin integration via coinswap.

# Contributing

The project is under active development by a few motivated Rusty Bitcoin devs. Any contribution for features, tests, docs and other fixes/upgrades is encouraged and welcomed. The maintainers will use the PR thread to provide quick reviews and suggestions and are generally proactive at merging good contributions.

Few directions for new contributors:

- The list of [issues](https://github.com/utxo-teleport/teleport-transactions/issues) is a good place to look for contributable tasks and open problems.

- Issues marked with [`good first issue`](https://github.com/utxo-teleport/teleport-transactions/issues?q=is%3Aopen+is%3Aissue+label%3A%22good+first+issue%22) are good places to get started for newbie Rust/Bitcoin devs.

- The [docs](./docs) are a good place to start reading up on the protocol.

- Reviewing [open PRs](https://github.com/utxo-teleport/teleport-transactions/pulls) are a good place to start gathering a contextual understanding of the codebase.

- Search for `TODO`s in the codebase to find in-line marked code todos and smaller improvements.

## Community

The dev community lurks in a small corner of Discord [here](https://discord.gg/TSSAB3g4Zf) (say holla, if you drop there from this readme).

Dev discussions predominantly happen via FOSS best practices, and by using Github as the Community Forum.

The Issues, PRs and Discussions are where all the hard lifting happening.
