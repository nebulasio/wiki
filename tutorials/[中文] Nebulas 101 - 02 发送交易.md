# Nebulas 101 - 02 在星云链上发送交易

Nebulas提供了三种方式去发送我们的交易：

1. 通过http接口
2. 通过console控制台
3. 通过nebulas测试框架

下面我们分别介绍如何通过以上三种方式在nebulas中发送一笔交易，并验证交易是否成功。

### 准备工作
在启动neb应用之前，需要先做一些准备工作：

1. 准备两个钱包地址，一个钱包地址用于挖矿钱包地址coinbase，接收挖矿奖励，也是后文转账交易的发送方，可在节点config文件中配置；另一个钱包地址是转账的接收地址，即后文转帐交易的接收方，可以用 neb 指令创建。
2. 修改节点的配置文件，配置 coinbase 地址。

#### 准备工作细节
##### 1. 配置挖矿钱包地址coinbase 

coinbase 对应着矿工挖矿的奖励地址，矿工挖矿得到的奖励都会进到这个地址。所以在启动节点之前，需要先配置coinbase地址。这里我们使用默认配置文件 config_local.conf 中的 coinbase 地址 `n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5`。

##### 2. 创建转账的接收地址

现在我们通过 `account new` 指令创建一个转帐交易接收地址。
```sh
$ ./neb -c conf/default/config_local.conf account new
Your new account is locked with a passphrase. Please give a passphrase. Do not forget this passphrase.
Passphrase:
Repeat passphrase:
Address: n1SQe5d1NKHYFMKtJ5sNHPsSPVavGzW71Wy
```
执行完这个命令以后，根据提示输入密码，该密码用于加密私钥信息。之后neb程序会在项目根目录的`keydir`子目录下新生成该地址对应的Key文件，如图所示：.
![key](resources/101-02-new-key.png)

**注意:** 每当执行该命令创建账户时，都会得到一个不同的账户地址。所以这里你得到的地址并不是`n1SQe5d1NKHYFMKtJ5sNHPsSPVavGzW71Wy`，在下面的例子中要使用你自己得到的地址替换"your_address"。

### 启动neb应用
完成所有的准备工作后，就可以启动neb应用。启动neb应用的方式非常简单：
```
$ ./neb -c conf/default/config_local.conf
```
neb应用会加载我们先前设置的`conf/default/config_local.conf`配置文件。neb应用启动之后会默认进入挖矿状态，一段时间以后（1~2分钟），挖矿产生的奖励会在我们刚刚设置的coinbase账户地址上面。当前开发代码的挖矿奖励为16 NAS（后续会根据白皮书的要求进行调整修正），平均出块时间大约10秒钟。

### 查询账户状态(余额)

Nebulas提供了RPC接口，让开发者通过HTTP或gPRC协议与星云链进行交互，完成更丰富复杂的操作。。在这里，我们介绍如何通过HTTP协议的接口，查询各个帐户的余额。Nebulas的HTTP接口地址和端口是通过配置文件中的`api_http_port`属性来配置，默认端口是`8685`。

接下来，我们使用curl工具来展示RPC接口的调用。

我们可以通过查询该coinbase地址账户余额的接口去查看这个用户挖矿的奖励。当coinbase账户地址有余额以后，就可以进行转账交易了。
当系统启动以后，我们可以通过curl发送http请求的方式查询账户的余额信息，下面的返回值表示这个地址的余额是67066180000000000000 Wei(1 NAS = 1*10^18 Wei)：

```sh
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5"}'

// Result
{
	"result": {
		"balance": "67066180000000000000",
		"nonce": "0",
		"type": 87
	}
}
```
[`GetAccountState`接口](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt)用于查询账户状态，可以得到账户余额，交易序号信息。

### 发送并验证转账交易
发送转账交易，可以按照如下步骤来进行：

#### 1. 获取账户信息；

```diff
// Request
curl -i -H Accept:application/json -X GET http://localhost:8685/v1/admin/accounts

// Result

"result": {
    "addresses": [
        "n1FkntVUMPAsESuCAAPK711omQk19JotBjM", 
        "n1JNHZJEUvfBYfjDRD14Q73FX62nJAzXkMR", 
        "n1Kjom3J4KPsHKKzZ2xtt8Lc9W5pRDjeLcW", 
        "n1NHcbEus81PJxybnyg4aJgHAaSLDx9Vtf8", 
        "n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", 
        "your_address", 
        "n1TV3sU6jyzR4rJ1D7jCAmtVGSntJagXZHC", 
        "n1WwqBXVMuYC3mFCEEuFFtAXad6yxqj4as4", 
        "n1Z6SbjLuAEXfhX1UJvXT6BB5osWYxVg3F3", 
        "n1Zn6iyyQRhqthmCfqGBzWfip1Wx8wEvtrJ"]
}

```

[`Accounts`接口](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#accounts)返回了当前启动的节点里面所有的账户信息，我们可以从中找到我们之前创建的coinbase账户地址，以及我们接受转账的账户地址。


#### 2. 找一个账户余额大于0的账户用于转账，并解锁该账户；


这里我们使用`config`文件中的 coinbase 账户`n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5`
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "passphrase":"passphrase"}'

// Result
{
	"result": {
		"result": true
	}
}
```
在转账交易之前需要先对发送方地址进行解锁，[`UnLockAccount`接口](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#unlockaccount)就是用于解锁账户，解锁账户需要使用创建地址时设置的密码。配置文件中用到的账户的默认密码都是“passphrase”。

#### 3. 使用已经解锁的账户向另一个账户发起一笔转账交易；

发送一笔交易时需要先对交易进行签名，然后再发送交易。

##### 3.1 对交易进行签名
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"transaction":{"from":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","to":"your_address", "value":"10","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}, "passphrase":"passphrase"}'

// Result
{
	"result": {
		"data": "CiCLGwkou3td6j97Hoig0Ilrj6MDVTT/ZIhdhVHDfL0pTRIaGVduKnw+6lM3HBXhJEzzuvv3yNdYANelaeAaGhlXgnTQAVavvSNEb+nUQTztv0NL502gnnJqIhAAAAAAAAAAAAAAAAAAAAAKKAEww5v01QU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBh8vZ1c9u9Je+MeySP4KwHjUGNqryPC47VQKGIlM0fLo0IhEHLECIgnBBxq/NRSlNM0XwVSsC2TTiXpGFsUXphgE="
	}
}
```
[`SignTransactionWithPassphrase`接口](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#signtransactionwithpassphrase)对一笔交易信息进行签名。上面的交易内容为账户`n1QZMX`向账户`n1SQe5` 转账金额10 Wei。转账时必须配置`gasPrice`和`gasLimit`，这里的nonce必须是该用户上一个nonce+1，该用户上一个nonce值可以通过查询账户状态信息获取。该接口返回值是该交易的签名数据。

**注意:** 交易中的`value`, `gasPrice` 和 `gasLimit` 应该是字符串类型，需要用双引号或单引号括起来，因为他们的值有可能很大，会超出整型数值的表示范围。如果没有双引号会得到如下错误：
```
{"error":"json: cannot unmarshal number into Go value of type string"} 
// this means you forgot to add “10” quotes around the numbers
```


##### 3.2 发送交易签名
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/rawtransaction -d '{"data":"CiCLGwkou3td6j97Hoig0Ilrj6MDVTT/ZIhdhVHDfL0pTRIaGVduKnw+6lM3HBXhJEzzuvv3yNdYANelaeAaGhlXgnTQAVavvSNEb+nUQTztv0NL502gnnJqIhAAAAAAAAAAAAAAAAAAAAAKKAEww5v01QU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBh8vZ1c9u9Je+MeySP4KwHjUGNqryPC47VQKGIlM0fLo0IhEHLECIgnBBxq/NRSlNM0XwVSsC2TTiXpGFsUXphgE="}'

// Result
{
    "result":{
        "txhash": "8b1b0928bb7b5dea3f7b1e88a0d0896b8fa3035534ff64885d8551c37cbd294d"
    }
}
```
 [`SendRawTransaction`接口](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#sendrawtransaction)用于发送交易，发送信息为交易的签名数据，返回值为该交易的 hash 值，通过该 hash 值可以用来对这笔交易进行查询。
 
**注意：** 如果你尝试重发一次该指令，会得到一个错误。因为每次交易的nonce（交易编号）不能重复，这样是为了避免重复发送和执行交易导致账户损失。所以每个账号发送的不同交易的编号都不能相同，每发送一笔交易，nonce应该自增。
```
{"error":"transaction's nonce is invalid, should bigger than the from's nonce"}
```

另外，以上两步交易也可以通过[`SendTransaction`接口](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#sendtransaction)一次完成：
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transaction -d '{"from":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","to":"your_address", "value":"10","nonce":0,"gasPrice":"1000000","gasLimit":"2000000"}'

// Result
{
    "result":{
      "txhash":"fb5204e106168549465ea38c040df0eacaa7cbd461454621867eb5abba92b4a5",
      "contract_address":""
    }
}
```

#### 4. 等待大约30s，然后查询该转账交易信息（因为转账交易需要矿工打包才能成功，所以会有一定的延时，并不是实时立马成功）；

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"8b1b0928bb7b5dea3f7b1e88a0d0896b8fa3035534ff64885d8551c37cbd294d"}'

// Result
{
	"result": {
		"hash": "8b1b0928bb7b5dea3f7b1e88a0d0896b8fa3035534ff64885d8551c37cbd294d",
		"chainId": 100,
		"from": "n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5",
		"to": "your_address",
		"value": "10",
		"nonce": "1",
		"timestamp": "1522339267",
		"type": "binary",
		"data": null,
		"gas_price": "1000000",
		"gas_limit": "2000000",
		"contract_address": "",
		"status": 1,
		"gas_used": "20000"
	}
}
```
[`GetTransactionReceipt`接口](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt)可以对之前的转账交易进行查询，请求参数是之前转账交易的hash值。如果查询到交易信息，说明该交易执行成功。

#### 5. 查询转账接收账户余额，验证转账交易是否成功；

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"your_address"}'

// Result
{
	"result": {
		"balance": "10",
		"nonce": "0",
		"type": 87
	}
}
```
查询可得转账接收放账户余额为10 Wei, 正好等于交易额度。

### 通过console控制台
Nebulas提供了javascript的交互控制台。控制台实现了[API](https://github.com/nebulasio/wiki/blob/master/rpc.md)和[Admin](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)接口。控制台提供了账号查看，创建账号地址，解锁账号，交易签名，发送交易等功能。控制台需要在本地先启动节点, 或者在启动控制台后通过`admin.setHost()`连接远程节点。
通过console控制台发送交易的步骤和通过调用http请求基本类似，调用方式更加简单。
##### 启动console控制台
```
$ ./neb -c conf/default/config_local.conf console
```

上面这种方式默认会连接本地启动的neb节点。console控制台实现了方法自动补全功能，使用`TAB`查看已有的方法：

```js
> api.
api.call                    api.getBlockByHeight        api.latestIrreversibleBlock 
api.estimateGas             api.getDynasty              api.sendRawTransaction      
api.gasPrice                api.getEventsByHash         api.subscribe               
api.getAccountState         api.getNebState             
api.getBlockByHash          api.getTransactionReceipt
```
```js
> admin.
admin.accounts                      admin.nodeInfo                      admin.signHash                      
admin.getConfig                     admin.sendTransaction               admin.signTransactionWithPassphrase 
admin.lockAccount                   admin.sendTransactionWithPassphrase admin.startPprof                    
admin.newAccount                    admin.setHost                       admin.unlockAccount  
```

##### 查看账号地址

```js
> admin.accounts()
{
    "result": {
        "addresses": [
            "n1FkntVUMPAsESuCAAPK711omQk19JotBjM",
            "n1JNHZJEUvfBYfjDRD14Q73FX62nJAzXkMR",
            "n1Kjom3J4KPsHKKzZ2xtt8Lc9W5pRDjeLcW",
            "n1NHcbEus81PJxybnyg4aJgHAaSLDx9Vtf8",
            "n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5",
            "your_address",
            "n1TV3sU6jyzR4rJ1D7jCAmtVGSntJagXZHC",
            "n1WwqBXVMuYC3mFCEEuFFtAXad6yxqj4as4",
            "n1Z6SbjLuAEXfhX1UJvXT6BB5osWYxVg3F3",
            "n1Zn6iyyQRhqthmCfqGBzWfip1Wx8wEvtrJ"
        ]
    }
}

```

##### 解锁账号

当前官方代码中默认keydir中的地址的密码是`passphrase`

```js
> admin.unlockAccount("n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5")
Unlock account n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5
Passphrase:
{
    "result": {
        "result": true
    }
}
```


##### 发送交易

在admin.sendTransaction（）中输入要发送出的账号地址、要发送到的账号地址、转账金额、交易编号、GasPrice、GasLimit。注意账号地址前后要加上双引号。


```js
> admin.sendTransaction("n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "your_address","10",2, "1000000", "200000")
{
    "result": {
        "contract_address": "",
        "txhash": "84d1ed79830566013df68809eb65e3948551ba0c2758e048ff2101aa5665703d"
    }
}
```

##### 查询交易


```js
> api.getTransactionReceipt("84d1ed79830566013df68809eb65e3948551ba0c2758e048ff2101aa5665703d")
{
    "result": {
        "chainId": 100,
        "contract_address": "",
        "data": null,
        "from": "n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5",
        "gas_limit": "200000",
        "gas_price": "1000000",
        "gas_used": "20000",
        "hash": "84d1ed79830566013df68809eb65e3948551ba0c2758e048ff2101aa5665703d",
        "nonce": "2",
        "status": 1,
        "timestamp": "1522341302",
        "to": "your_address",
        "type": "binary",
        "value": "10"
    }
}
```

##### 查询账号余额情况

```js
> api.getAccountState("your_address")
{
    "result": {
        "balance": "20",
        "nonce": "0",
        "type": 87
    }
}
```

### 通过nebtestkit 测试框架
[nebtestkit](https://github.com/nebulasio/go-nebulas/tree/develop/nebtestkit) 是一个基于[mocha](https://github.com/mochajs/mocha)的集成测试框架。通过`nebtestkit`可以启动一个或者多个nebulas节点，组件一个完整的私有链或者加入一个已经存在的网络，然后进行转账交易、部署和调用智能合约等。
关于`nebtestkit`的使用说明可以参考[nebtestkit使用说明](https://github.com/nebulasio/go-nebulas/blob/develop/nebtestkit/README.md), 这里不再赘述。



