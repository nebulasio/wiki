# Nebulas 101 - 02 在星云链上发送交易

Nebulas提供了三种方式去发送我们的交易：

1. 签名 & 发送
2. 密码 & 发送
3. 解锁 & 发送

下面我们分别介绍如何通过以上三种方式在nebulas中发送一笔交易，并验证交易是否成功。

## 准备账户

在星云链上，每个地址表示一个唯一的账户，一一对应。

在发送交易前，我们需要准备两个账户：一个账户用来发送代币 (称为"from") 和另一个账户来接受代币 (称为"to").

### 发送者账户

在这里，我们将会使用配置文件`conf/default/genesis.conf`中预分配过代币的账户中选择一个作为发送者账户，默认选择`n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`。

### 接受者账户

我们使用如下指令创建一个全新的账户来做接受者，请记住输入的密码。

```bash
$ ./neb account new
Your new account is locked with a passphrase. Please give a passphrase. Do not forget this passphrase.
Passphrase:
Repeat passphrase:
Address: n1SQe5d1NKHYFMKtJ5sNHPsSPVavGzW71Wy
```

> 提示：你创建的新账户和上面可能不一样，请以你创建的账户做为接受者继续接下来的实验。

新账户的keystore文件将会被放置在`$GOPATH/src/github.com/nebulasio/go-nebulas/keydir/`内。

## 启动私有链

我们将在本地搭建一个私有链来作为本教程的测试环境。

### 启动种子节点

首先，我们启动本地私有链的第一个节点，它可以作为其他节点的种子节点。

```bash
./neb -c conf/default/config.conf
```

### 启动矿工节点

接着，我们启动一个矿工节点接入本地私有链，这个节点之后将会产生新的区块。

```bash
./neb -c conf/example/miner.conf
```

> **多久会生成一个新的区块?**
>
> 在星云链上, 在贡献度证明（Proof-of-Devotion， [技术白皮书](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)中有详细描述）被充分验证前，DPoS被选择作为一个过渡方案。在我们采用DPoS共识算法中，总共有21个矿工，每个矿工会轮流每15s出一个新区块。
> 
> 在我们目前的测试环境中，由于我们只启动了21个矿工中的一个，所以需要等待15\*21s才会出一个新区块。

一旦一个新区块被挖出，挖块奖励将会被自动发送到当前矿工配置的Coinbase账户中，在`conf/example/miner.conf`里，该账户就是`n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`.

## 星云链交互

星云链提供给开发者HTTP API, RPC API和CLI来和运行中的星云节点交互。在教程中，我们将会基于HTTP API（[API Module](https://github.com/nebulasio/wiki/blob/master/rpc.md) | [Admin Module](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)）来介绍三种发送交易的方法。

> 提示：星云链的HTTP服务默认端口号为8685。

首先，在发送新交易前，我们检查下发送者账户的状态。

### 检查账户状态

每个交易如果需要上链，都需要给矿工缴纳一部分手续费，所以发送者账户中需要有一部分钱才能成功发送交易。一般一个普通转账交易，手续费在0.000000002NAS左右，非常少。

我们可以通过[API Module](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate)中的`/v1/user/accountstate`接口来获取发送者账户`n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`的账户信息，检查下是否有足够的钱支付上链手续费。

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE"}'

{"result":{"balance":"5000000000000000000000000","nonce":"0","type":87}}
```

> 提示：`Type`用于标记账户类型。88表示该账户为智能合约账户，部署一个合约之后，就可以得到一个合约账户。87表示非合约账户，我们通过`./neb account new`创建的账户就是非合约账户，用户存储链上资产。

> 提示：`Nonce`用于标记账户发起的所有交易的顺序。同一个账户，每发起一个新的交易，`Nonce`就加一，初始为0，第一个交易的`Nonce`为1。

如我们所见，发送者账户在预分配后拥有5000000000000000000000000(5 * 10\^24)个代币，1个NAS是1000000000000000000（10\^18）个代币，用于支付交易上链的手续费绰绰有余。

然后我们检查接受者账户的状态。

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1SQe5d1NKHYFMKtJ5sNHPsSPVavGzW71Wy"}'

{"result":{"balance":"0","nonce":"0","type":87}}
```

如我们期望的那样，新账户没有任何代币。

### 发送交易

接下来，我们将介绍星云链上三种发送交易的方式。

#### 签名 & 发送

使用这种方式，我们可以在离线环境下先使用私钥签名好交易，然后把签好名的交易在联网的机器上发出。这是最安全的发送交易的方式，私钥可以完全离线保存，不触网。[Web-Wallet](https://github.com/nebulasio/web-wallet)正是基于[Neb.js](https://github.com/nebulasio/neb.js)采用这种方法发送的交易。

首先，我们使用[Admin Module](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#signtransactionwithpassphrase)中的`v1/admin/sign`接口给准备发的交易签名，得到交易的二进制数据。

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}, "passphrase":"passphrase"}'

{"result":{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}}
```

> 提示：在发送交易时，对于同一个账户，只有当他`Nonce`为N的交易上链后，`Nonce`为N+1的交易才能上链，有严格的顺序，`Nonce`必须严格加1。可以通过[GetAccountState](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate)接口查看最新的Nonce。

然后，我们将签好名的交易原始数据提交到本地私有链里的星云节点。

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/rawtransaction -d '{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}'

{"result":{"txhash":"1b8cc3f977256c4d620b7d72f531bc19f10eb13a05ff24a8a792cd5da53a1277","contract_address":""}}⏎
```

#### 密码 & 发送

如果你信任一个星云节点帮你保存keystore文件，你可以使用第二种方法发送交易。

首先，上传你的keystore文件到你信任的星云节点的keydir文件夹下。如果在节点在本地，可以使用如下指令。

```bash
cp /path/to/keystore.json /path/to/keydir/
```

然后，我们发送交易的同时，带上我们keystore的密码，在被信任的节点使用[SendTransactionWithPassphrase](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#sendtransactionwithpassphrase)接口上一次性完成签名和发送过程。

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":2,"gasPrice":"1000000","gasLimit":"2000000"},"passphrase":"passphrase"}'

{"result":{"txhash":"3cdd38a66c8f399e2f28134e0eb556b292e19d48439f6afde384ca9b60c27010","contract_address":""}}
```

> 提示：因为我们在之前使用`n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`发送了一个`Nonce`为1的交易，所以这里新的交易的`Nonce`应该增加1，变成2再提交。

#### 解锁 & 发送

这是最危险的发送交易的方法。除非你完全信任一个星云节点，否则不要使用这种方法来发送交易。

首先，上传你的keystore文件到你信任的星云节点的keydir文件夹下。如果在节点在本地，可以使用如下指令。

```bash
cp /path/to/keystore.json /path/to/keydir/
```

然后，使用你的keystore文件的密码，在指定的时间范围来在被信任的节点上使用[Unlock](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#unlockaccount)接口解锁账户。时间单位为纳秒，300000000000为300s。

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","passphrase":"passphrase","duration":"300000000000"}'

{"result":{"result":true}}
```

一旦一个账户在节点上被解锁，任何可以访问该机器[SendTransaction](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#sendtransaction)接口的人，都可以直接使用该账户的身份发送交易。

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transaction -d '{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":3,"gasPrice":"1000000","gasLimit":"2000000"}'

{"result":{"txhash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","contract_address":""}}⏎
```

## 交易收据

不论使用的哪一种方法发送交易，我们都会得到两个返回值，`txhash`和`contract_address`。其中`txhash`为交易hash，是一个交易的唯一标识。如果当前交易是一个部署合约的交易，`contract_address`将会是合约地址，调用合约时都会使用这个地址，是合约的唯一标识。我们将在[编写并运行智能合约](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B%E4%B8%AD%E6%96%87%5D%20Nebulas%20101%20-%2003%20%E7%BC%96%E5%86%99%E6%99%BA%E8%83%BD%E5%90%88%E7%BA%A6.md)中介绍如何发送部署合约的交易。

使用`txhash`我们可以查看交易收据，知道当前交易的状态。

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf"}'

{"result":{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","chainId":100,"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","value":"1000000000000000000","nonce":"3","timestamp":"1524667888","type":"binary","data":null,"gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20000"}}⏎
```

这里的`status`可能有三种状态值，0，1和2。

- **0: 交易失败.** 表示当前交易已经上链，但是执行失败了。可能是因为部署合约或者调用合约参数错误。
- **1: 交易成功.** 表示当前交易已经上链，而且执行成功了。
- **2: 交易待定.** 表示当前交易还没有上链。可能是因为当前交易还没有被打包；如果长时间处于当前状态，可能是因为当前交易的发送者账户的余额不够支付上链手续费。

### 复查接受者账户余额

我们复查一下接受者账户上的钱是否已经到账了。

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5"}'

{"result":{"balance":"3000000000000000000","nonce":"0","type":87}}
```

我们用三种方式分别发送了一笔转账，每笔转一个NAS，所以这里看到接受者账户中已经有了3个NAS，即3000000000000000000个代币。

### 下一章

 [编写并运行智能合约](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md)