# Nebulas 101 - 03 编写并运行智能合约

在这篇教程中我们会学习怎样在Nebulas中编写、部署并执行智能合约。

## 准备工作

在进入智能合约之前，先温习下先前学过的内容：

1. 安装、编译并启动neb应用
2. 创建钱包地址，设置coinbase，并开始挖矿
3. 查询neb节点信息、钱包地址余额等
4. 发送转账交易，并验证交易是否成功

如果对上述的内容有疑惑的同学可以重新去学习前面的章节，接下来我们会通过下面的步骤来学习和使用智能合约：

1. 编写智能合约
2. 部署智能合约
3. 调用智能合约，验证合约执行结果

## 编写智能合约

跟以太坊类似，Nebulas实现了NVM虚拟机来运行智能合约，NVM的实现使用了JavaScript V8引擎，所以当前的开发版，我们可以使用JavaScript、TypeScript来编写智能合约。

编写智能合约的简要规范：

1. 智能合约代码必须是一个Prototype的对象；
2. 智能合约代码必须有一个init()的方法，这个方法只会在部署的时候被执行一次；
3. 智能合约里面的私有方法是以_开头的方法，私有方法不能被外部直接调用；

下面我们使用JavaScript来编写第一个智能合约：银行保险柜。
这个智能合约需要实现以下功能：

1. 用户可以向这个银行保险柜存钱。
2. 用户可以从这个银行保险柜取钱。
3. 用户可以查询银行保险柜中的余额。

智能合约示例：

```js
'use strict';

var DepositeContent = function (text) {
  if (text) {
    var o = JSON.parse(text);
    this.balance = new BigNumber(o.balance);
    this.expiryHeight = new BigNumber(o.expiryHeight);
  } else {
    this.balance = new BigNumber(0);
    this.expiryHeight = new BigNumber(0);
  }
};

DepositeContent.prototype = {
  toString: function () {
    return JSON.stringify(this);
  }
};

var BankVaultContract = function () {
  LocalContractStorage.defineMapProperty(this, "bankVault", {
    parse: function (text) {
      return new DepositeContent(text);
    },
    stringify: function (o) {
      return o.toString();
    }
  });
};

// save value to contract, only after height of block, users can takeout
BankVaultContract.prototype = {
  init: function () {
    //TODO:
  },

  save: function (height) {
    var from = Blockchain.transaction.from;
    var value = Blockchain.transaction.value;
    var bk_height = new BigNumber(Blockchain.block.height);

    var orig_deposit = this.bankVault.get(from);
    if (orig_deposit) {
      value = value.plus(orig_deposit.balance);
    }

    var deposit = new DepositeContent();
    deposit.balance = value;
    deposit.expiryHeight = bk_height.plus(height);

    this.bankVault.put(from, deposit);
  },

  takeout: function (value) {
    var from = Blockchain.transaction.from;
    var bk_height = new BigNumber(Blockchain.block.height);
    var amount = new BigNumber(value);

    var deposit = this.bankVault.get(from);
    if (!deposit) {
      throw new Error("No deposit before.");
    }

    if (bk_height.lt(deposit.expiryHeight)) {
      throw new Error("Can not takeout before expiryHeight.");
    }

    if (amount.gt(deposit.balance)) {
      throw new Error("Insufficient balance.");
    }

    var result = Blockchain.transfer(from, amount);
    if (!result) {
      throw new Error("transfer failed.");
    }
    Event.Trigger("BankVault", {
      Transfer: {
        from: Blockchain.transaction.to,
        to: from,
        value: amount.toString()
      }
    });

    deposit.balance = deposit.balance.sub(amount);
    this.bankVault.put(from, deposit);
  },
  balanceOf: function () {
    var from = Blockchain.transaction.from;
    return this.bankVault.get(from);
  },
  verifyAddress: function (address) {
    // 1-valid, 0-invalid
    var result = Blockchain.verifyAddress(address);
    return {
      valid: result == 0 ? false : true
    };
  }
};
module.exports = BankVaultContract;
```

上面智能合约的示例可以看到，`BankVaultContract`是一个prototype对象，这个对象有一个init()方法，满足了我们说的编写智能合约最基础的规范。

`BankVaultContract`实现了另外两个方法：

- save(): 用户可以通过调用save()方法向银行保险柜存钱；
- takeout(): 用户可以通过调用takeout()方法向银行保险柜取钱；
- balanceOf(): 用户可以通过调用balanceOf()方法向银行保险柜查询余额；

上面的合约代码用到了内置的`Blockchain`对象和内置的`BigNumber()`方法，下面我们来逐行拆解分析合约代码：

**save():**

```js
// Deposit the amount into the safe

save: function (height) {
  var from = Blockchain.transaction.from;
  var value = Blockchain.transaction.value;
  var bk_height = new BigNumber(Blockchain.block.height);

  var orig_deposit = this.bankVault.get(from);
  if (orig_deposit) {
    value = value.plus(orig_deposit.balance);
  }
  var deposit = new DepositeContent();
  deposit.balance = value;
  deposit.expiryHeight = bk_height.plus(height);

  this.bankVault.put(from, deposit);
},
```

**takeout():**

```js
takeout: function (value) {
  var from = Blockchain.transaction.from;
  var bk_height = new BigNumber(Blockchain.block.height);
  var amount = new BigNumber(value);

  var deposit = this.bankVault.get(from);
  if (!deposit) {
    throw new Error("No deposit before.");
  }

  if (bk_height.lt(deposit.expiryHeight)) {
    throw new Error("Can not takeout before expiryHeight.");
  }

  if (amount.gt(deposit.balance)) {
    throw new Error("Insufficient balance.");
  }

  var result = Blockchain.transfer(from, amount);
  if (!result) {
    throw new Error("transfer failed.");
  }
  Event.Trigger("BankVault", {
    Transfer: {
      from: Blockchain.transaction.to,
      to: from,
      value: amount.toString()
    }
  });

  deposit.balance = deposit.balance.sub(amount);
  this.bankVault.put(from, deposit);
},
```


## 部署智能合约

上面介绍了在Nebulas中怎么去编写一个智能合约，现在我们需要把智能合约部署到链上。在Nebulas中部署一个智能合约其实也是发送一个transaction来实现，前面有介绍过用户如何在Nebulas中进行转账交易，由于是本地测试，我们直接使用`解锁 & 发送`的方式来发送交易。

首先，我们在`conf/default/genesis.conf`中预分配过代币的账户里，选择账户`n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so`作为本章的发送者账号，并检查该账户的状态。

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so"}'

{"result":{"balance":"5000000000000000000000000","nonce":"0","type":87}}
```

该账户有足够的钱来支付手续费，接下来，我们解锁发送者账户`n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so`，解锁12小时。

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","passphrase":"passphrase","duration":"43200000000000"}'

{"result":{"result":true}}
```

然后，我们发送部署BankVault合约的交易。

```js
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction": {"from":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","to":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so", "value":"0","nonce":1,"gasPrice":"1000000","gasLimit":"2000000","contract":{"source":"\"use strict\";var DepositeContent=function(text){if(text){var o=JSON.parse(text);this.balance=new BigNumber(o.balance);this.expiryHeight=new BigNumber(o.expiryHeight);}else{this.balance=new BigNumber(0);this.expiryHeight=new BigNumber(0);}};DepositeContent.prototype={toString:function(){return JSON.stringify(this);}};var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\",{parse:function(text){return new DepositeContent(text);},stringify:function(o){return o.toString();}});};BankVaultContract.prototype={init:function(){},save:function(height){var from=Blockchain.transaction.from;var value=Blockchain.transaction.value;var bk_height=new BigNumber(Blockchain.block.height);var orig_deposit=this.bankVault.get(from);if(orig_deposit){value=value.plus(orig_deposit.balance);} var deposit=new DepositeContent();deposit.balance=value;deposit.expiryHeight=bk_height.plus(height);this.bankVault.put(from,deposit);},takeout:function(value){var from=Blockchain.transaction.from;var bk_height=new BigNumber(Blockchain.block.height);var amount=new BigNumber(value);var deposit=this.bankVault.get(from);if(!deposit){throw new Error(\"No deposit before.\");} if(bk_height.lt(deposit.expiryHeight)){throw new Error(\"Can not takeout before expiryHeight.\");} if(amount.gt(deposit.balance)){throw new Error(\"Insufficient balance.\");} var result=Blockchain.transfer(from,amount);if(!result){throw new Error(\"transfer failed.\");} Event.Trigger(\"BankVault\",{Transfer:{from:Blockchain.transaction.to,to:from,value:amount.toString()}});deposit.balance=deposit.balance.sub(amount);this.bankVault.put(from,deposit);},balanceOf:function(){var from=Blockchain.transaction.from;return this.bankVault.get(from);},verifyAddress:function(address){var result=Blockchain.verifyAddress(address);return{valid:result==0?false:true};}};module.exports=BankVaultContract;","sourceType":"js", "args":""}}, "passphrase": "passphrase"}'

{"result":{"txhash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889","contract_address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM"}}
```

- from: 合约创建者账户地址
- to: 和from一致，同为合约创建者地址
- value：部署合约时为`"0"`
- nonce: 比创建者当前的`nonce`多1，可以通过[`GetAccountState`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate)获取创建前当前`nonce`
- gasPrice：部署智能合约用到的gasPrice，可以通过[`GetGasPrice`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getgasprice)获取，或者使用默认值:`"1000000"`；
- gasLimit: 部署合约的gasLimit，通过[`EstimateGas`](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimateGas)可以获取部署合约的gas消耗，不能使用默认值，也可以设置一个较大值，执行时以实际使用计算。
- contract: 合约信息，部署合约时传入的参数
  - `source`: 合约代码
  - `sourceType`: 合约代码类型，支持`js`和`ts`(对应javaScript和typeScript代码)
  - `args`: 合约初始化方法参数，无参数为空字符串，有参数时为JSON数组

部署智能合约的返回值是transaction的hash地址`txhash`和合约的部署地址`contract_address`。
得到返回值并不能保证合约已经部署成功，因为发送交易是一个异步的过程，需要经过矿工打包，正如之前的转账交易一样，转账并不能实时到账，依赖矿工打包的速度，所以需要等待一段时间，然后可以通过查询合约地址的信息或者调用智能合约来验证合约是否部署成功。

> **验证合约是否部署成功**
> 
> 在部署智能合约的时候得到了transaction的hash地址`txhash`，我们可以很方便的使用console控制台查询transaction的hash信息来验证合约是否部署成功。
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889"}'
> 
> {"result":{"hash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889","chainId":100,"from":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","to":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","value":"0","nonce":"1","timestamp":"1524711841","type":"deploy","data":"eyJTb3VyY2VUeXBlIjoianMiLCJTb3VyY2UiOiJcInVzZSBzdHJpY3RcIjt2YXIgRGVwb3NpdGVDb250ZW50PWZ1bmN0aW9uKHRleHQpe2lmKHRleHQpe3ZhciBvPUpTT04ucGFyc2UodGV4dCk7dGhpcy5iYWxhbmNlPW5ldyBCaWdOdW1iZXIoby5iYWxhbmNlKTt0aGlzLmV4cGlyeUhlaWdodD1uZXcgQmlnTnVtYmVyKG8uZXhwaXJ5SGVpZ2h0KTt9ZWxzZXt0aGlzLmJhbGFuY2U9bmV3IEJpZ051bWJlcigwKTt0aGlzLmV4cGlyeUhlaWdodD1uZXcgQmlnTnVtYmVyKDApO319O0RlcG9zaXRlQ29udGVudC5wcm90b3R5cGU9e3RvU3RyaW5nOmZ1bmN0aW9uKCl7cmV0dXJuIEpTT04uc3RyaW5naWZ5KHRoaXMpO319O3ZhciBCYW5rVmF1bHRDb250cmFjdD1mdW5jdGlvbigpe0xvY2FsQ29udHJhY3RTdG9yYWdlLmRlZmluZU1hcFByb3BlcnR5KHRoaXMsXCJiYW5rVmF1bHRcIix7cGFyc2U6ZnVuY3Rpb24odGV4dCl7cmV0dXJuIG5ldyBEZXBvc2l0ZUNvbnRlbnQodGV4dCk7fSxzdHJpbmdpZnk6ZnVuY3Rpb24obyl7cmV0dXJuIG8udG9TdHJpbmcoKTt9fSk7fTtCYW5rVmF1bHRDb250cmFjdC5wcm90b3R5cGU9e2luaXQ6ZnVuY3Rpb24oKXt9LHNhdmU6ZnVuY3Rpb24oaGVpZ2h0KXt2YXIgZnJvbT1CbG9ja2NoYWluLnRyYW5zYWN0aW9uLmZyb207dmFyIHZhbHVlPUJsb2NrY2hhaW4udHJhbnNhY3Rpb24udmFsdWU7dmFyIGJrX2hlaWdodD1uZXcgQmlnTnVtYmVyKEJsb2NrY2hhaW4uYmxvY2suaGVpZ2h0KTt2YXIgb3JpZ19kZXBvc2l0PXRoaXMuYmFua1ZhdWx0LmdldChmcm9tKTtpZihvcmlnX2RlcG9zaXQpe3ZhbHVlPXZhbHVlLnBsdXMob3JpZ19kZXBvc2l0LmJhbGFuY2UpO30gdmFyIGRlcG9zaXQ9bmV3IERlcG9zaXRlQ29udGVudCgpO2RlcG9zaXQuYmFsYW5jZT12YWx1ZTtkZXBvc2l0LmV4cGlyeUhlaWdodD1ia19oZWlnaHQucGx1cyhoZWlnaHQpO3RoaXMuYmFua1ZhdWx0LnB1dChmcm9tLGRlcG9zaXQpO30sdGFrZW91dDpmdW5jdGlvbih2YWx1ZSl7dmFyIGZyb209QmxvY2tjaGFpbi50cmFuc2FjdGlvbi5mcm9tO3ZhciBia19oZWlnaHQ9bmV3IEJpZ051bWJlcihCbG9ja2NoYWluLmJsb2NrLmhlaWdodCk7dmFyIGFtb3VudD1uZXcgQmlnTnVtYmVyKHZhbHVlKTt2YXIgZGVwb3NpdD10aGlzLmJhbmtWYXVsdC5nZXQoZnJvbSk7aWYoIWRlcG9zaXQpe3Rocm93IG5ldyBFcnJvcihcIk5vIGRlcG9zaXQgYmVmb3JlLlwiKTt9IGlmKGJrX2hlaWdodC5sdChkZXBvc2l0LmV4cGlyeUhlaWdodCkpe3Rocm93IG5ldyBFcnJvcihcIkNhbiBub3QgdGFrZW91dCBiZWZvcmUgZXhwaXJ5SGVpZ2h0LlwiKTt9IGlmKGFtb3VudC5ndChkZXBvc2l0LmJhbGFuY2UpKXt0aHJvdyBuZXcgRXJyb3IoXCJJbnN1ZmZpY2llbnQgYmFsYW5jZS5cIik7fSB2YXIgcmVzdWx0PUJsb2NrY2hhaW4udHJhbnNmZXIoZnJvbSxhbW91bnQpO2lmKCFyZXN1bHQpe3Rocm93IG5ldyBFcnJvcihcInRyYW5zZmVyIGZhaWxlZC5cIik7fSBFdmVudC5UcmlnZ2VyKFwiQmFua1ZhdWx0XCIse1RyYW5zZmVyOntmcm9tOkJsb2NrY2hhaW4udHJhbnNhY3Rpb24udG8sdG86ZnJvbSx2YWx1ZTphbW91bnQudG9TdHJpbmcoKX19KTtkZXBvc2l0LmJhbGFuY2U9ZGVwb3NpdC5iYWxhbmNlLnN1YihhbW91bnQpO3RoaXMuYmFua1ZhdWx0LnB1dChmcm9tLGRlcG9zaXQpO30sYmFsYW5jZU9mOmZ1bmN0aW9uKCl7dmFyIGZyb209QmxvY2tjaGFpbi50cmFuc2FjdGlvbi5mcm9tO3JldHVybiB0aGlzLmJhbmtWYXVsdC5nZXQoZnJvbSk7fSx2ZXJpZnlBZGRyZXNzOmZ1bmN0aW9uKGFkZHJlc3Mpe3ZhciByZXN1bHQ9QmxvY2tjaGFpbi52ZXJpZnlBZGRyZXNzKGFkZHJlc3MpO3JldHVybnt2YWxpZDpyZXN1bHQ9PTA/ZmFsc2U6dHJ1ZX07fX07bW9kdWxlLmV4cG9ydHM9QmFua1ZhdWx0Q29udHJhY3Q7IiwiQXJncyI6IiJ9","gas_price":"1000000","gas_limit":"2000000","contract_address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","status":1,"gas_used":"22016"}}
>```
> 如上所示，部署合约的交易的状态变成了1，表示合约部署成功了。

## 执行智能合约方法

在Nebulas中调用智能合约的方式也很简单，同样是通过发送交易来调用智能合约。

### 调用智能合约的`save`方法

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction":{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM", "value":"100","nonce":1,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"save","args":"[0]"}}, "passphrase": "passphrase"}'

{"result":{"txhash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84","contract_address":""}}
```

- from: 用户的账户地址
- to: 智能合约地址
- value: 调用save()方法时想要存入智能合约代币数量
- nonce: 比创建者当前的`nonce`多1，可以通过[`GetAccountState`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate)获取创建前当前`nonce`
- gasPrice：部署智能合约用到的gasPrice，可以通过[`GetGasPrice`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getgasprice)获取，或者使用默认值:`"1000000"`；
- gasLimit: 部署合约的gasLimit，通过[`EstimateGas`](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimateGas)可以获取部署合约的gas消耗，不能使用默认值，也可以设置一个较大值，执行时以实际使用计算。
- contract: 合约信息，部署合约时传入的参数
  - `function`: 调用合约方法
  - `args`: 合约方法参数，无参数为空字符串，有参数时为JSON数组

> **验证智能合约的方法`save`是否执行成功**
>
> 由于执行合约方法本质是提交一个交易，所以我们可以通过验证交易是否提交成功来判断合约方法是否执行成功。
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84"}'
> 
> {"result":{"hash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84","chainId":100,"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","value":"100","nonce":"1","timestamp":"1524712532","type":"call","data":"eyJGdW5jdGlvbiI6InNhdmUiLCJBcmdzIjoiWzBdIn0=","gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20361"}}
> ```
> 如上所示，交易状态变为了1，表示执行`save`方法成功了。

### 调用智能合约的`takeout`方法

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction":{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM", "value":"0","nonce":2,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"takeout","args":"[50]"}}, "passphrase": "passphrase"}'

{"result":{"txhash":"46a307e9beb21f52992a7512f3705fe58ee6c1887122a1b52f5ce5fd5f536a91","contract_address":""}}
```

> **验证智能合约的方法`takeout`是否执行成功**
>
> 在上面save方法的执行中，我们在合约`n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM`中存了100NAS。此时，我们执行`takeout`函数，从中取出50NAS。合约里应该还有50NAS。我们检测下合约账户的余额来验证`takeout`方法执行是否成功。
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM"}'
>
> {"result":{"balance":"50","nonce":"0","type":88}}
> ```
> 结果和预期相符。

## 查询智能合约数据

在智能合约中，我们有一些方法并不更改合约的状态，这些方法被设计来帮助我们获取合约数据，它们是只读的。在星云链上，我们在API Module中为用户提供了[`Call`](https://github.com/nebulasio/wiki/blob/master/rpc.md#call)接口来帮助用户来执行这些只读的方法，使用`Call`接口不会发送交易，也就无需支付上链手续费。

我们调用合约`n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM`中的`balanceOf`方法来查询该合约的余额。

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/call -H 'Content-Type: application/json' -d '{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","value":"0","nonce":3,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"balanceOf","args":""}}'

{"result":{"result":"{\"balance\":\"50\",\"expiryHeight\":\"84\"}","execute_err":"","estimate_gas":"20209"}}
```

### 下一章

[智能合约存储区](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B中文%5D%20Nebulas%20101%20-%2004%20智能合约存储区.md)
