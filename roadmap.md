# Roadmap of Nebulas

## Goals

### v1.0

* Fully functional blockchain, with NR, PoD and Smart Contract (Solidity-compatible).

### v2.0

* Fully functional NF, with NVM, Protocol Code Upgrading, DIP and more smart contract languages support.

### Long term goals

* Scalability for large transaction volume.
* Subchain support.
* Zero-knowledge Proof integration.

## v1.0

* Test-net is online in 2018 Q1.
* Main-net is online in 2018 Q2.


### v0.1.0

Goals

* Implement a nebulas kernel.
* In-memory blockchain with PoW consensus.
* Fully P2P network support.


### v0.2.0

Goals

* Provide (RPC) API to submit/query transaction externally.
* Implement Sync Protocol to bootstrap any nodes that join into nebulas network at any time, from any tail.

Core

* Implement transaction pool.
* Prevent record-replay attack of transaction.
* Integrate Protocol Buffer for serialization.

Net

* Refactor the design of network.
* Implement Sync Protocol.
* Implement Broadcast and Relay function.

API

* Add Balance API.
* Add Transaction API.
* Add some debugging API, eg “Dump Chain”, “Dump Block”.

Crypto

* Support Ethereum-keystore file.
* Support multi keystore files management in KeyStore.


### v0.3.0

Goals

* Support disk storage for all blockchain data.
* Add smart contract execution engine, based on LLVM.
* Support Solidity language.

Core

* Add disk storage with a middleware of storage.
* Implement smart contract transaction.

NVM

* Integrate LLVM as Smart Contract execution engine.


### v0.5.0

Goals

* Implement NR algorithm.
* Implement PoD consensus algorithm.
* Support more API.
* Add repl in neb application.

Core

* Implement Vote mechanism and NR transactions.
* Implement NR algorithm.

Consensus

* Implement PoD algorithm.

CMD

* Add repl in neb application.

API

* Add more API.
* Support JSON-RPC.


### v0.8.0

Expected release date: 2018 Q1

Goals

* Ready for Test-net.
* Core features are all implemented except fully functional NVM.
* Release a simple wallet of Nebulas.

Core

* Implement DIP algorithm.

Debugging

*  Anonymous log collection and analysis system.

Wallet

* A simple wallet.


### v1.0.0

Expected release date: 2018 Q2

Goals

* Ready for Main-net.
* Achieve production quality, especially in robust performance and stability.

neb (nebulas client)

* Fully functional blockchain with local storage.
* Implement NR, PoD and DIP algorithm and protocols.
* Safety smart contract execution env with Solidity language support.
* Fully functional API (RPC and JSON-RPC) and repl.

Wallet

* A simple wallet client for nebulas.
