# Nebulas 101 - 06 连接测试网络
星云链的测试网络有特定的chainID,在连接测试网络时需要将创世区块配置`genesis.conf`更新为测试网络的创世区块配置，同时更新配置信息。官方的介绍[Testnet](https://github.com/nebulasio/wiki/blob/master/testnet.md)。

## 配置

配置文件在官方代码仓库中[`testnet/conf`](https://github.com/nebulasio/go-nebulas/tree/develop/testnet/conf):

- config.conf
- genesis.conf

为测试网络生成连接私钥

```
./neb network ssh-keygen testnet/conf/network.key
```

将生成的key文件配置到你的配置文件中

```
private_key: "testnet/conf/network.key"
```

*注意：如果您不提供节点连接服务，你可以不在配置文件中配置网络密钥。这样，每当你启动节点时，节点就会随机地为p2p网络生成一个新密钥。*

使用测试网络的配置文件启动节点

```
./neb -c testnet/conf/config.conf
```

## 测试网络NAS获取

测试网络的NAS可以从官方的网站分发获取，用于测试。分发地址：[https://testnet.nebulas.io/claim/](https://testnet.nebulas.io/claim/).

## 浏览器
星云提供了一个区块浏览器来查看星云网络的区块和交易信息.[explorer](https://explorer.nebulas.io/#/)

## 钱包
星云提供了一个官方的网页钱包，支持测试网络的转账.[wallet](https://github.com/nebulasio/web-wallet)