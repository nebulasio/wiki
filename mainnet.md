# How to Join Nebulas Mainnet

## Introduction

We are glad to release Nebulas Mainnet here. Please join and enjoy Nebulas Mainnet.

```
https://github.com/nebulasio/go-nebulas/tree/master
```

### Configuration

The Mainnet configuration files are in folder [`mainnet/conf`](https://github.com/nebulasio/go-nebulas/tree/master/mainnet/conf), including

### genesis.conf

All configurable information about genesis block is defined in genesis.conf, including

- **meta.chain_id:** chain identity
- **consensus.dpos.dynasty:** the initial dynasty of validators
- **token_distribution:** the initial allocation of tokens

> *Attention*: DO NOT change the genesis.conf.

### config.conf

All configurable information about runtime is defined in config.conf.

Please check the [`template.conf`](resources/conf/template.conf) to find more details about the runtime configuration.

> *Tips*: the official seed node info is as below,

```json
seed:["/ip4/52.2.205.12/tcp/8680/ipfs/QmQK7W8wrByJ6So7rf84sZzKBxMYmc1i4a7JZsne93ysz5","/ip4/52.56.55.238/tcp/8680/ipfs/QmVy9AHxBpd1iTvECDR7fvdZnqXeDhnxkZJrKsyuHNYKAh","/ip4/13.251.33.39/tcp/8680/ipfs/QmVm5CECJdPAHmzJWN2X7tP335L5LguGb9QLQ78riA9gw3"]
```

### API List

Main Endpoint:

| API | URL | Protocol |
|-------|:------------:|:------------:|
| RESTful | https://mainnet.nebulas.io/ | HTTP |

- [GetNebState](https://github.com/nebulasio/wiki/blob/master/rpc.md#getnebstate) : returns nebulas client info.
- [GetAccountState](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate): returns the account balance and nonce.
- [Call](https://github.com/nebulasio/wiki/blob/master/rpc.md#call): execute smart contract local, don't submit on chain.
- [SendRawTransaction](https://github.com/nebulasio/wiki/blob/master/rpc.md#sendrawtransaction): submit the signed transaction.
- [GetTransactionReceipt](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt): get transaction receipt info by tansaction hash.

More Nebulas APIs at [RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md).

## Tutorials

### English

1. [Installation](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2001%20Installation.md) (thanks [Victor](https://github.com/victorychain))
2. [Sending a Transaction](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2002%20Transaction.md) (thanks [Victor](https://github.com/victorychain))
3. [Writing Smart Contract in JavaScript](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md) (thanks [otto](https://github.com/ottokafka))
4. [Introducing Smart Contract Storage](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2004%20Smart%20Contract%20Storage.md) (thanks [Victor](https://github.com/victorychain))
5. [Interacting with Nebulas by RPC API](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2005%20Interacting%20with%20Nebulas%20by%20RPC%20API.md) (thanks [Victor](https://github.com/victorychain))
6. [Connecting to the test network](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2006%20Testnet.md)

### 中文

1. [编译安装及运行neb](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2001%20编译安装.md)
2. [在星云链上发送交易](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2002%20发送交易.md)
3. [使用JavaScript编写智能合约](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2003%20编写智能合约.md)
4. [智能合约存储区介绍](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2004%20智能合约存储区.md)
5. [通过RPC接口与星云链交互](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2005%20通过RPC接口与星云链交互.md)
6. [连接测试网络](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B%E4%B8%AD%E6%96%87%5D%20Nebulas%20101%20-%2006%20%E6%B5%8B%E8%AF%95%E7%BD%91%E7%BB%9C.md)

## Contribution

Feel free to join Nebulas Mainnet. If you did find something wrong, please [submit a issue](https://github.com/nebulasio/go-nebulas/issues/new) or [submit a pull request](https://github.com/nebulasio/go-nebulas/pulls) to let us know, we will add your name and url to this page soon.
