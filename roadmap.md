# Roadmap of Nebulas

## Milestones

* In 2017 December, Nebulas test-net will be online.
* In 2018 Q1, Nebulas v1.0 will be released and main-net will be online (ahead of the original schedules).

### v1.0 (2018 Q1)

* Fully functional blockchain, with JavaScript and TypeScript as the languages of Smart Contract.
* A user-friendly Nebulas Wallet for both desktop and mobile device to manage their own assets on Nebulas.
* A web-based Nebulas Block Explorer to let developers and users search and view all the data on Nebulas.

### v2.0 (2018 Q4)

* Add Nebulas Rank (NR) to each addresses on Nebulas, help users and developers finding more values inside.
* Implement Developer Incentive Protocol (DIP) to encourage developers build more valuable decentralized applications on Nebulas.

### v3.0 (2019 Q4)

* Fully functional Nebulas Force and PoD implementation.

## Long term goals

* Scalability for large transaction volume.
* Subchain support.
* Zero-knowledge Proof integration.

## Versions

### v0.1.0 [done]

Goals

* Implement a nebulas kernel.
* In-memory blockchain with PoW consensus.
* Fully P2P network support.

Download [here](https://github.com/nebulasio/go-nebulas/releases/tag/v0.1.0).

### v0.2.0 [done]

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
* Support multi key files management in KeyStore.

Download [here](https://github.com/nebulasio/go-nebulas/releases/tag/v0.2.0).

### v0.3.0 [done]

Goals

* Support disk storage for all blockchain data.
* Add smart contract execution engine, based on Chrome V8.

Core

* Add disk storage with a middleware of storage.
* Implement smart contract transaction.

NVM

* Integrate Chrome V8 as Smart Contract execution engine.

Download [here](https://github.com/nebulasio/go-nebulas/releases/tag/v0.3.0).

### v0.4.0 [done]

Goals

* Implement Gas calculating in Smart Contract Execution Engine.
* Support more API.
* Add repl in neb application.
* Add metrics and reporting capability.

Core

* Add Gas related fields in Transaction.
* Implemented Gas calculation mechanism.

NVM

* Add execution limits to V8 Engine.
* Add Gas calculation mechanism.

CMD

* Add repl in neb application

Misc
* Add more API.
* Add metrics and reporting capability.

Download [here](https://github.com/nebulasio/go-nebulas/releases/tag/v0.4.0).

### v0.5.0

Goals

* Prepare for test-net releasing, improve stability.

Core

* Improve stability and missing functions if we miss anything.

Consensus

* Implement DPoS consensus algorithm and keep developing PoD algorithm.

NVM

* Finalize the Gas Cost Matrix.
* Support Event liked pubsub functionality.

Misc

* Add more metrics to monitor the stability of neb applications.

### v0.8.0

Goals

* New Nebulas Block Explorer.
* New Nebulas Wallet.
* New web-based Playground tools to interactive with Nebulas.

### v1.0.0

Goals

* Ready for main-net.
* Support JavaScript and TypeScript as Smart Contract Language.
* Stable and high performance blockchain system.
* Release new Nebulas Block Explorer.
* Release new Nebulas Wallet for both desktop and mobile device.
* A web-based playground tools for developer.
