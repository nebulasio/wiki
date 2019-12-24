# How to Join Nebulas Testnet

## Introduction

We are glad to release Nebulas Testnet here. It simulate the Nebulas network and NVM, and allow developers to interact with Nebulas without paying the cost of gas.

```
https://github.com/nebulasio/go-nebulas/tree/testnet
```

### Configuration

The testnet configuration files are in folder [`testnet/conf`](https://github.com/nebulasio/go-nebulas/tree/testnet/testnet/conf) under `testnet` branch, including

#### genesis.conf

All configurable information about genesis block is defined in genesis.conf, including

- **meta.chain_id:** chain identity
- **consensus.dpos.dynasty:** the initial dynasty of validators
- **token_distribution:** the initial allocation of tokens

> *Attention*: DO NOT change the genesis.conf.

#### config.conf

All configurable information about runtime is defined in config.conf.

Please check the [`template.conf`](resources/conf/template.conf) to find more details about the runtime configuration.

> *Tips*: the official seed node info is as below,

``` json
seed:["/ip4/47.92.203.173/tcp/8680/ipfs/QmfSJ7JUnCEDP6LFyKkBUbpuDMETPbqMVZvPQy4keeyBDP","/ip4/47.89.180.5/tcp/8680/ipfs/QmTmnd5KXm4UFUquAJEGdrwj1cbJCHsTfPWAp5aKrKoRJK"]
```

#### Miner config 
Nodes can participate in mining and share rewards after signing up for mining. The miner node needs to turn on the mine switch and configure both the miner address and reward address(coinbase).

miner config example:

```
chain {
  # testnet chainID
  chain_id: 1001
  # testnet datadir, should be different with private chain
  datadir: "testnet/data.db"
  keydir: "keydir"
  # testnet genesis.conf
  genesis: "testnet/conf/genesis.conf"
  # testnet dynasty.conf
  dynasty: "testnet/conf/dynasty.conf"
  
  # start mine
  start_mine: true
  # receive the mining award, must change to your address
  coinbase: "n1XkoVVjswb5Gek3rRufqjKNpwrDdsnQ7Hq"
  # block signature address, needs to be placed in the node's configuration `keydir`. Also make sure that the address is the node address at the time of registration
  miner: "n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE"
  # 
  passphrase: "passphrase"

  signature_ciphers: ["ECC_SECP256K1"]
}
```

#### Synchronization

Since Nebulas testnet is running there for certain period of time, it will take quite some time to sync all the testnet data from scratch. 

For developers' convenience, we provided a offline data package, which already includes the data of more than 1.2 million blocks, you can download the package directly by following either link below (choose whichever is faster for you):
- [`download from AWS s3`](https://s3-us-west-1.amazonaws.com/develop-center/testnet/data.db.tar.gz)
- [`download from Aliyun oss`](http://develop-center.oss-cn-zhangjiakou.aliyuncs.com/data/testnet/data1540000.db.tar.gz)

> Please note that, the data package should be put under the same path of "datadir" as specified in your `config.conf` file.


#### API List

Test Endpoint:

| API | URL | Protocol |
|-------|:------------:|:------------:|
| RESTful | https://testnet.nebulas.io/ | HTTP |

- [GetNebState](https://github.com/nebulasio/wiki/blob/master/rpc.md#getnebstate) : returns nebulas client info.
- [GetAccountState](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate): returns the account balance and nonce.
- [LatestIrreversibleBlock](https://github.com/nebulasio/wiki/blob/master/rpc.md#latestirreversibleblock): returns the lastest irreversible block.
- [Call](https://github.com/nebulasio/wiki/blob/master/rpc.md#call): execute smart contract locally. the tx won't be submitted on chain.
- [SendRawTransaction](https://github.com/nebulasio/wiki/blob/master/rpc.md#sendrawtransaction): submit signed transaction. The transaction must be signed before send.
- [GetTransactionReceipt](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt): get transaction receipt info by tansaction hash.

More Nebulas APIs at [RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md).

#### Token Claim

Every email can claim some tokens every day at [here](https://testnet.nebulas.io/claim).

## Tutorials

#### English

1. [Installation](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2001%20Installation.md) (thanks [Victor](https://github.com/victorychain))
2. [Sending a Transaction](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2002%20Transaction.md) (thanks [Victor](https://github.com/victorychain))
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
