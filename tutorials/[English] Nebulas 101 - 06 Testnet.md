# Nebulas 101 - 06 Connecting to the test network

The test network of the Nebulas chain has a specific chainID. When connecting to the test network, the genesis.conf of the Genesis block needs to be updated to the Genesis block configuration of the test network, and update the node configuration information at the same time. Please refer to official introduction of [Testnet](https://github.com/nebulasio/wiki/blob/master/testnet.md)

## Configuration

The configaration files of testnet are in folder [`testnet/conf`](https://github.com/nebulasio/go-nebulas/tree/testnet/testnet/conf) under `testnet` branch:

 - config.conf
 - genesis.conf

Generate your own private key for testnet:

```
./neb network ssh-keygen testnet/conf/network.key
```

Config the `private_key` with the generated key in config file.

```
private_key: "testnet/conf/network.key"
```

*Notice: If you do not provide a node connection service, you may not configure the network key in the config file. In this way, each time you start the node, the node randomly generates a new key for the p2p network.*

Start Node with the testnet config file:

```
./neb -c testnet/conf/config.conf
```

## Testnet NAS Tokens

NAS for testing network can be obtained from the official website, which is used for testing. Distribution Address: [https://testnet.nebulas.io/claim/](https://testnet.nebulas.io/claim/).

## explorer

Nebulas provides a block explorer to view block/transaction information.[explorer](https://explorer.nebulas.io/#/)

## wallet

Nebulas provides a web wallet to send transaction and deploy/call contract.[wallet](https://github.com/nebulasio/web-wallet)
