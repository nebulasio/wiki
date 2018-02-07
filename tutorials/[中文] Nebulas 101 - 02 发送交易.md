# Nebulas 101 - 02 在星云链上发送交易

Nebulas提供了三种方式去发送我们的交易：

1. 通过http接口
2. 通过console控制台
3. 通过nebulas测试框架

下面我们分别介绍如何通过以上三种方式在nebulas中发送一笔交易，并验证交易是否成功。

### 准备工作
在启动neb应用之前，需要先做一些准备工作：
1. 创建两个钱包地址，一个钱包地址用于挖矿钱包地址coinbase，接收挖矿奖励，也是后文转账交易的发送方，另一个钱包地址是转账的接收地址，即后文转帐交易的接收方。
2. 修改节点的配置文件，配置coinbase地址。

#### 准备工作细节
1. 配置挖矿钱包地址coinbase 
coinbase 对应着矿工挖矿的奖励地址，矿工挖矿得到的奖励都会进到这个地址。所以在启动节点之前，需要先配置coinbase地址。我们可以从genesis.conf里面consensus-> dpos-> dynasty里面前六个地址选取一个地址作为coinbase的地址。当前我们的版本是六个地址作为一个dynasty，轮流出块，所以只有前六个地址可以配置成coinbase地址。
![key](resources/101-02-genesis.png)

把我们选取的coinbase地址`1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c`替换掉配置文件`conf/default/config.conf`里面的`chain`属性里面的coinbase 和 miner（如下图所示）。后面用户启动neb应用后挖矿产生的奖励就会进入这个地址。
![key](resources/101-02-coinbase.png)

2. 创建转账的接收地址
现在我们通过同样的方式创建一个转帐交易接收地址。
```sh
$ ./neb account new
Your new account is locked with a passphrase. Please give a passphrase. Do not forget this passphrase.
Passphrase:
Repeat passphrase:
Address: e6dea0d0769fbf71ab01f8e0d78cd59e78361a450e1f4f88
```
执行完这个命令以后，neb程序会在当前目录的`keydir`子目录下新生成该地址对应的Key文件，如图所示：
![key](resources/101-02-new-key.png)

### 启动neb应用
完成所有的准备工作后，就可以启动neb应用。启动neb应用的方式非常简单：
```
$ ./neb -c conf/default/config.conf
```
neb应用会加载我们先前设置的`conf/default/config.conf`配置文件。neb应用启动之后会默认进入挖矿状态，一段时间以后（1~2分钟），挖矿产生的奖励会在我们刚刚设置的coinbase账户地址上面。当前开发代码的挖矿奖励为16 NAS（后续会根据白皮书的要求进行调整修正），平均出块时间大约10秒钟。

### 查询余额

Nebulas提供了RPC接口，让开发者通过HTTP或gPRC协议与星云链进行交互，完成更丰富复杂的操作。。在这里，我们介绍如何通过HTTP协议的接口，查询各个帐户的余额。Nebulas的HTTP接口地址和端口是通过配置文件中的`api_http_port`属性来配置，默认端口是`8685`。

接下来，我们使用curl工具来展示RPC接口的调用。

我们可以通过查询该coinbase地址账户余额的接口去查看这个用户挖矿的奖励。当coinbase账户地址有余额以后，就可以进行转账交易了。
当系统启动以后，我们可以通过curl发送http请求的方式查询账户的余额信息，下面的返回值表示这个地址的余额是64：

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c"}'

// Result
{
   "Balance":"64"，
   "nonce":"0"
}
```


### 发送并验证转账交易
发送转账交易，可以按照如下步骤来进行：

1. 获取账户信息；

```
// Request
curl -i -H Accept:application/json -X GET http://localhost:8685/v1/user/accounts

// Result
{
   "addresses":[
       "1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c",
       "e6dea0d0769fbf71ab01f8e0d78cd59e78361a450e1f4f88"
   ]
}
```
这个接口返回了当前启动的节点里面所有的账户信息，我们可以从中找到我们之前创建的coinbase账户地址（红色标识）以及我们接受转账的账户地址（蓝色标识）。

2. 找到账户余额大于0的账户，并解锁该账户；

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c", "passphrase":"passphrase"}'

// Result
{
   "result":true
}
```
这个接口就是准备工作提到的转账交易时候需要先对发送方地址进行解锁，解锁账户需要使用创建地址时候的密码。

3. 使用已经解锁的账户向另一个账户发起一笔转账交易；

```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/transaction  -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"e6dea0d0769fbf71ab01f8e0d78cd59e78361a450e1f4f88","nonce":1,"value":"10"}'

// Result
{
  "txhash": "93930906f21282b4cd72de8292d122806f65e6803cddd9e9e203561996237ace"
}
```
转账交易接口：账户`0fba`向账户`6c05` 转账金额10。这里的nonce必须是该用户上一个nonce+1，该用户上一个nonce值可以通过查询账户余额信息获取。该接口返回值是交易的hash值，这个hash值可以用来对这笔交易进行查询。转账时若不配置`gasPrice`和`gasLimit`,会使用默认值来执行交易（只有在普通交易中可以不做配置，后面提到的部署合约需要配置），详细的接口[API](https://github.com/nebulasio/wiki/blob/master/rpc.md#sendtransaction).

4. 等待大约30s，然后查询该转账交易信息（因为转账交易需要矿工打包才能成功，所以会有一定的延时，并不是实时立马成功）；

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"93930906f21282b4cd72de8292d122806f65e6803cddd9e9e203561996237ace"}'

// Result
{
   "hash":"93930906f21282b4cd72de8292d122806f65e6803cddd9e9e203561996237ace",
   "from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c",
   "to":"e6dea0d0769fbf71ab01f8e0d78cd59e78361a450e1f4f88",
   "nonce":"1",
   "timestamp":"1511519091",
   "chainId":1
}
```
这个接口可以对之前的转账交易进行查询，请求参数是之前转账交易的hash值。如果查询到交易信息，说明该交易执行成功。

5. 查询转账接收账户余额，验证转账交易是否成功；

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"e6dea0d0769fbf71ab01f8e0d78cd59e78361a450e1f4f88"}'

// Result
{
   "balance":"10"
}
```

### 通过console控制台
Nebulas提供了javascript的交互控制台。控制台实现了[API](https://github.com/nebulasio/wiki/blob/master/rpc.md)和[Admin](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)接口。控制台提供了账号查看，创建账号地址，解锁账号，交易签名，发送交易等功能。控制台需要在本地先启动节点, 或者在启动控制台后通过`admin.setHost()`连接远程节点。
通过console控制台发送交易的步骤和通过调用http请求基本类似，调用方式更加简单。
##### 启动console控制台
```
./neb console
```

上面这种方式默认会连接本地启动的neb节点。console控制台实现了方法自动补全功能，使用`TAB`查看已有的方法：

```js
> api.
api.accounts              api.gasPrice              api.getNebState           api.sendTransaction
api.blockDump             api.getAccountState       api.getTransactionReceipt api.setRequest
api.call                  api.getBlockByHash        api.nodeInfo              api.subscribe
api.estimateGas           api.getEventsByHash       api.sendRawTransaction
```
```js
> admin.
admin.getDynasty                    admin.sendTransactionWithPassphrase admin.signTransaction
admin.lockAccount                   admin.setHost                       admin.unlockAccount
admin.newAccount                    admin.setRequest
```

##### 查看账号地址

```js
> api.accounts()
{
    "addresses": [
        "1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c",
        "2fe3f9f51f9a05dd5f7c5329127f7c917917149b4e16b0b8",
        "333cb3ed8c417971845382ede3cf67a0a96270c05fe2f700",
        "48f981ed38910f1232c1bab124f650c482a57271632db9e3",
        "59fc526072b09af8a8ca9732dae17132c4e9127e43cf2232",
        "75e4e5a71d647298b88928d8cb5da43d90ab1a6c52d0905f",
        "7da9dabedb4c6e121146fb4250a9883d6180570e63d6b080",
        "98a3eed687640b75ec55bf5c9e284371bdcaeab943524d51",
        "a8f1f53952c535c6600c77cf92b65e0c9b64496a8a328569",
        "b040353ec0f2c113d5639444f7253681aecda1f8b91f179f",
        "b414432e15f21237013017fa6ee90fc99433dec82c1c8370",
        "b49f30d0e5c9c88cade54cd1adecf6bc2c7e0e5af646d903",
        "b7d83b44a3719720ec54cdb9f54c0202de68f1ebcb927b4f",
        "ba56cc452e450551b7b9cffe25084a069e8c1e94412aad22",
        "c5bcfcb3fa8250be4f2bf2b1e70e1da500c668377ba8cd4a",
        "c79d9667c71bb09d6ca7c3ed12bfe5e7be24e2ffe13a833d",
        "d1abde197e97398864ba74511f02832726edad596775420a",
        "d86f99d97a394fa7a623fdf84fdc7446b99c3cb335fca4bf",
        "e0f78b011e639ce6d8b76f97712118f3fe4a12dd954eba49",
        "f38db3b6c801dddd624d6ddc2088aa64b5a24936619e4848",
        "fc751b484bd5296f8d267a8537d33f25a848f7f7af8cfcf6"
    ]
}
```

##### 解锁账号

当前官方代码中默认keydir中的地址的密码是`passphrase`

```js
> admin.unlockAccount("1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c")
Unlock account 1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c
Passphrase:
{
    "result": true
}```


##### 发送交易

****   请在（）中输入要发送出的账号地址及要发送到的账号地址，账号地址前后要加上双引号


```js
> api.sendTransaction("1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c", "b49f30d0e5c9c88cade54cd1adecf6bc2c7e0e5af646d903","1000000000000000000",1)
{
    "txhash": "4cfb6461873a478f10eb35424e03ab5abad3e10bd030d2f31b3c96a02b747d22"
}
```

##### 查询交易

**** 请在（）中输入要查讯的账号地址，账号地址前后要加上双引号

```js
> api.getTransactionReceipt("4cfb6461873a478f10eb35424e03ab5abad3e10bd030d2f31b3c96a02b747d22")
{
    "chainId": 100,
    "from": "1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c",
    "gas_limit": "20000",
    "gas_price": "1000000",
    "hash": "4cfb6461873a478f10eb35424e03ab5abad3e10bd030d2f31b3c96a02b747d22",
    "nonce": "1",
    "timestamp": "1514898795",
    "to": "b49f30d0e5c9c88cade54cd1adecf6bc2c7e0e5af646d903",
    "type": "binary",
    "value": "1000000000000000000"
}
```

##### 查询账号余额情况

**** 请在（）中输入要查讯的账号地址，账号地址前后要加上双引号

> api.getAccountState("1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c")
{
    "balance": "9998999999980000000000",
    "nonce": "1"
    

### 通过nebtestkit
[nebtestkit](https://github.com/nebulasio/go-nebulas/tree/develop/nebtestkit) 是一个基于[mocha](https://github.com/mochajs/mocha)的集成测试框架。通过`nebtestkit`可以启动一个或者多个nebulas节点，组件一个完整的私有链或者加入一个已经存在的网络，然后进行转账交易、部署和调用智能合约等。
关于`nebtestkit`的使用说明可以参考[nebtestkit使用说明](https://github.com/nebulasio/go-nebulas/blob/develop/nebtestkit/README.md), 这里不再赘述。



