# How to Join Nebulas Testnet

## Introduction

We are glad to release Nebulas Testnet here. It simulate the Nebulas network and NVM, and allow developers to interact with Nebulas without paying the cost of gas. 

#### Configuration

If you want to run your own nodes, please use testnet configuration to join Nebulas Testnet.

> Genesis.conf

Genesis.conf should be as same as [here](resources/conf/testnet-genesis.conf) exactly.

> neb.conf

Refer to neb.conf [here](resources/conf/neb.conf).

Notice:

* testnet seed node:

```
seed:["/ip4/13.56.18.241/tcp/8680/ipfs/QmYcBY52pnuNQNMtsLUdKYQeLzDHZqfTj1RQGYs4Gujuqi", "/ip4/54.206.110.30/tcp/8680/ipfs/QmcFzHfFRHbp6o2WbTYvxv7uLH5mjSJXpRDMc3jKfy5ze4", "/ip4/54.238.223.81/tcp/8680/ipfs/Qmac11jvtGpFt9Ptevn4SHHQpvJjNsC17ZX7VmuHvsHM8o", "/ip4/13.250.10.239/tcp/8680/ipfs/QmY6d8qdHaa1XoMs76uQt8UpCcNJL77kx9R5ACwQPhZCF4", "/ip4/47.52.174.176/tcp/8680/ipfs/QmUQ77Jmqs99R8gjrJHNmz8LEf6HQMghUxbZNzwviR1LJn", "/ip4/35.182.48.19/tcp/8680/ipfs/QmW5HY9ef16pGvdryyJSDCz42ZiHEEmpmFuzYHpEBWvySG", "/ip4/35.177.86.207/tcp/8680/ipfs/QmYpPgrwzxcE1jbVfwqQmM7eSGd6LufpRmV76nGKT2kY7M"]
```
* `chain_id` should be 1001, as same as [genesis.conf](resources/conf/testnet-genesis.conf).
* `datadir` should be different with private chain.

The example of testnet conf [here](resources/conf/testnet-config.conf).

#### API List

Test Endpoint:

| API | URL | Protocol |
|-------|:------------:|:------------:|
| RESTful | https://testnet.nebulas.io/ | HTTP |

* [GetNebState](https://github.com/nebulasio/wiki/blob/master/rpc.md#getnebstate) : returns nebulas client info.
* [Accounts](https://github.com/nebulasio/wiki/blob/master/rpc.md#accounts): list accounts on the neb node.
* [GetAccountState](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate): returns the account balance and nonce.
* [SendTransaction](https://github.com/nebulasio/wiki/blob/master/rpc.md#sendtransaction): submit unsigned transaction. The transaction's from address must be unlocked before send.
* [Call](https://github.com/nebulasio/wiki/blob/master/rpc.md#call): submit smart contract call transaction. The transaction's from address must be unlocked before send.
* [SendRawTransaction](https://github.com/nebulasio/wiki/blob/master/rpc.md#sendrawtransaction): submit the signed transaction.
* [GetTransactionReceipt](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt): get transaction receipt info by tansaction hash.

More Nebulas APIs at [RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md).

#### Token Claim

Every email can claim some tokens every day at [here](https://testnet.nebulas.io/claim).

## Tutorials

#### English

1. [Installation](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2001%20Installation.md) (thanks [Victor](https://github.com/victorychain))
2. [Sending Transaction](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2002%20Transaction.md) (thanks [Victor](https://github.com/victorychain))
3. [Writing Smart Contract in JavaScript](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md) (thanks [otto](https://github.com/ottokafka))
4. [Introducing Smart Contract Storage](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2004%20Smart%20Contract%20Storage.md) (thanks [Victor](https://github.com/victorychain))
5. [Interacting with Nebulas by RPC API](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2005%20Interacting%20with%20Nebulas%20by%20RPC%20API.md) (thanks [Victor](https://github.com/victorychain))

#### 中文

1. [编译安装及运行neb](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2001%20编译安装.md)
2. [在星云链上发送交易](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2002%20发送交易.md)
3. [使用JavaScript编写智能合约](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2003%20编写智能合约.md)
4. [智能合约存储区介绍](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2004%20智能合约存储区.md)
5. [通过RPC接口与星云链交互](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2005%20通过RPC接口与星云链交互.md)

## Contribution

Feel free to join Nebulas Testnet. If you did find something wrong, please [submit a issue](https://github.com/nebulasio/go-nebulas/issues/new) or [submit a pull request](https://github.com/nebulasio/go-nebulas/pulls) to let us know, we will add your name and url to this page soon.

## Updates

#### 0.5.0

- **network** p2p network, full-sync, downloader
- **basic** crypto, blockchain
- **consensus** using dpos
- **smart contract** support javascript & typescript
- **api** rpc & http interfaces
