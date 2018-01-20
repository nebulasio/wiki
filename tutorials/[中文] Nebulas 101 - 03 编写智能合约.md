# Nebulas 101 - 03 编写并运行智能合约

今天我们会学习怎样在Nebulas中编写、部署并执行智能合约。

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
		let o = JSON.parse(text);
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
			throw new Error("Can't takeout before expiryHeight.");
		}

		if (amount.gt(deposit.balance)) {
			throw new Error("Insufficient balance.");
		}

		var result = Blockchain.transfer(from, amount);
		if (result != 0) {
			throw new Error("transfer failed.");
		}
		Event.Trigger("BankVault", {
			Transfer: {
				from: Blockchain.transaction.to,
				to: from,
				value: amount.toString(),
			}
		});

		deposit.balance = deposit.balance.sub(amount);
		this.bankVault.put(from, deposit);
	},

	balanceOf: function () {
		var from = Blockchain.transaction.from;
		return this.bankVault.get(from);
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
save():

```js

	// 将金额存入保险柜
	save: function (height) {
	   // 获取当前调用合约的地址
		var from = Blockchain.transaction.from;
		// 获取当前交易值(value为Bignumber对象)，合约调用者将此金额转入合约地址，存入保险柜
		var value = Blockchain.transaction.value;
		// 当前区块高度
		var bk_height = new BigNumber(Blockchain.block.height);
		
		// 获取存款信息
		var orig_deposit = this.bankVault.get(from);
		if (orig_deposit) {
			value = value.plus(balance);
		}
		
		// 更新存款信息
		var deposit = new DepositeContent();
		deposit.balance = value;
		deposit.expiryHeight = bk_height.plus(height);

		this.bankVault.put(from, deposit);
	},
```

takeout():

```js
	// 从保险柜取出存款
	takeout: function (value) {
		// 存款人地址
		var from = Blockchain.transaction.from;
		// 当前区块高度
		var bk_height = new BigNumber(Blockchain.block.height);
		//取款金额
		var amount = new BigNumber(value);
		
		// 存款信息
		var deposit = this.bankVault.get(from);
		if (!deposit) {
			throw new Error("No deposit before.");
		}

		if (bk_height.lt(deposit.expiryHeight)) {
			throw new Error("Can't takeout before expiryHeight.");
		}

		if (amount.gt(deposit.balance)) {
			throw new Error("Insufficient balance.");
		}

		// 调用blockchain的transfer接口，将待取金额赚到用户的钱包地址
		var result = Blockchain.transfer(from, amount);
		if (result != 0) {
			throw new Error("transfer failed.");
		}
		
		// 添加转移事件监听
        Event.Trigger("BankVault", {
            Transfer: {
                from: Blockchain.transaction.to,
                to: from,
                value: amount.toString(),
            }
        });
        
        // 更新存款信息
		deposit.balance = deposit.balance.sub(amount);
		this.bankVault.put(from, deposit);
	}
```

## 部署智能合约
上面介绍了在Nebulas中怎么去编写一个智能合约，现在我们需要把智能合约部署到链上。
前面有介绍过用户如何在Nebulas中进行转账交易，我们使用sendTransation()接口来发起一笔转账交易。在Nebulas中部署一个智能合约其实也是发送一个transation来实现，即也是通过调用sendTransation()接口，只是参数不一样。

```js
sendTransation(from, to, value, nonce, gasPrice, gasLimit, contract)
```
我们约定：如果from和to是同一个地址，就认为是部署一个智能合约。

- value：部署合约时为`"0"`；
- gasPrice：部署智能合约用到的gasPrice，可以通过`GetGasPrice`获取，或者使用空字符串，使用默认值；
- gasLimit: 部署合约的gasLimit，通过[`EstimateGas`](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimateGas)可以获取部署合约的gas消耗，不能使用默认值，也可以设置一个较大值，执行时以实际使用计算。
- contract: 合约信息，部署合约时传入的参数
	- `source`: 合约代码
	- `sourceType`: 合约代码类型，支持`js`和`ts`(对应javaScript和typeScript代码)
	- `args`: 合约初始化方法参数，无参数为空字符串，有参数时为JSON数组

详细的接口文档[API](https://github.com/nebulasio/wiki/blob/master/rpc.md#sendtransaction).

使用curl方式部署智能合约示例：

```js
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/transaction -H 'Content-Type: application/json' -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c", "value":"0","nonce":1,"gasPrice":"1000000","gasLimit":"2000000","contract":{"source":"\"use strict\";var DepositeContent=function(text){if(text){var o=JSON.parse(text);this.balance=new BigNumber(o.balance);this.expiryHeight=new BigNumber(o.expiryHeight)}else{this.balance=new BigNumber(0);this.expiryHeight=new BigNumber(0)}};DepositeContent.prototype={toString:function(){return JSON.stringify(this)}};var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\",{parse:function(text){return new DepositeContent(text)},stringify:function(o){return o.toString()}})};BankVaultContract.prototype={init:function(){},save:function(height){var from=Blockchain.transaction.from;var value=Blockchain.transaction.value;var bk_height=new BigNumber(Blockchain.block.height);var orig_deposit=this.bankVault.get(from);if(orig_deposit){value=value.plus(orig_deposit.balance)}var deposit=new DepositeContent();deposit.balance=value;deposit.expiryHeight=bk_height.plus(height);this.bankVault.put(from,deposit)},takeout:function(value){var from=Blockchain.transaction.from;var bk_height=new BigNumber(Blockchain.block.height);var amount=new BigNumber(value);var deposit=this.bankVault.get(from);if(!deposit){throw new Error(\"No deposit before.\");}if(bk_height.lt(deposit.expiryHeight)){throw new Error(\"Can not takeout before expiryHeight.\");}if(amount.gt(deposit.balance)){throw new Error(\"Insufficient balance.\");}var result=Blockchain.transfer(from,amount);if(result!=0){throw new Error(\"transfer failed.\");}Event.Trigger(\"BankVault\",{Transfer:{from:Blockchain.transaction.to,to:from,value:amount.toString()}});deposit.balance=deposit.balance.sub(amount);this.bankVault.put(from,deposit)},balanceOf:function(){var from=Blockchain.transaction.from;return this.bankVault.get(from)}};module.exports=BankVaultContract;","sourceType":"js", "args":""}}'

// Result
{
    "txhash":"aa833b2771e708ec6970888d2a9a5ab4c79aeed04408157e6b9397a0b47c5f13",
    "contract_address":"4702b597eebb7a368ac4adbb388e5084b508af582dadde47"
}
```
部署智能合约的返回值是transaction的hash地址`txhash`和合约的部署地址`contract_address`。
得到返回值并不能保证合约已经部署成功，因为sendTransaction()是一个异步的过程，需要经过矿工打包，正如之前的转账交易一样，转账并不能实时到账，依赖矿工打包的速度，所以需要等待一段时间（约1分钟），然后可以通过查询合约地址的信息或者调用智能合约来验证合约是否部署成功。

## 验证合约是否部署成功
在部署智能合约的时候得到了合约的地址`contract_address`，我们可以很方便的使用console控制台查询合约的地址信息来验证合约是否部署成功。
![key](resources/101-03-state.png)
如上图所示，如果我们通过合约的地址可以查询到合约的信息，就表示合约部署成功了。

## 执行智能合约方法
在Nebulas中调用智能合约的方式也很简单，直接使用sendTransation方法来调用智能合约。

```js
sendTransation(from, to, value, nonce, gasPrice, gasLimit, contract)
```
- from: 用户钱包地址
- to: 智能合约地址
- value: 调用智能合约用于转账的金额
- nonce: from用户transaction标识，顺序增长
- gasPrice：部署智能合约用到的gasPrice，可以通过`GetGasPrice`获取，或者使用空字符串，使用默认值；
- gasLimit: 部署合约的gasLimit，通过[`EstimateGas`](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimateGas)可以获取部署合约的gas消耗，不能使用默认值，也可以设置一个较大值，执行时以实际使用计算。
- contract: 合约信息，部署合约时传入的参数
	- `function`: 调用合约方法
	- `args`: 合约方法参数，无参数为空字符串，有参数时为JSON数组

调用智能合约的save()方法：

```js
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/transaction -H 'Content-Type: application/json' -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c", "value":"0","nonce":2,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"save","args":"[0]"}}'

// Result
{
   "txhash": "9008b0958fc26034118bb0f65474c29041116bad8ef909a771af9ed2b2f5d261"
}
```
智能合约的调用本质也是提交一个transation，所以也依赖矿工打包，矿工将交易打包成功以后调用才算成功，所以智能合约的调用也不是立即生效。我们需要等待一段时间（约一分钟），然后可以验证我们的调用是否成功。
上面我们调用save()方法向银行保险柜存储了金额100的资金，需要先从用户的余额扣除100，所以有个转账的过程，转账的金额需要通过value字段来传递。合约调用之后，只需要验证智能合约的地址余额是否是100。
通过console控制台可以很方便的查询到当前智能合约地址的金额：
![key](resources/101-03-save-state.png)

调用智能合约的takeout()方法：

```js
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/transaction -H 'Content-Type: application/json' -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"333cb3ed8c417971845382ede3cf67a0a96270c05fe2f700","value":"0","nonce":4,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"takeout","args":"[50]"}}'

// Result
{
   "txhash": "cab27f9653cd8f3232d68fc8123d85ea508181a545b22d6eefd1f394dee7d053"
}
```
上面takeout()方法与save()方法有所不同，只是从保险柜取出50的金额，将取出的金额转给用户是智能合约内部的操作，所以value参数不需要有值，取出的金额是操作的智能合约相关的参数，所以通过args参数来传递。
然后我们需要验证当前智能合约地址的余额是不是50：
![key](resources/101-03-takeout-state.png)

上图可以看到智能合约调用结果无误，智能合约的部署到调用都是成功的。

## 查询智能合约数据
在Nebulas中已经提交的智能合约和执行方法会提交到链上，查询智能合约已经生成数据的方式也很简单，可以通过rpc接口call()方法来调用智能合约。通过`call()`方式调用合约方法不会发布到链上。

```js
call(from, to, value, nonce, gasPrice, gasLimit, contract)
```
- from: 用户钱包地址
- to: 用户钱包地址/智能合约地址
- value: 调用智能合约用于转账的金额
- nonce: 用户transaction标识，顺序增长
- gasPrice：部署智能合约用到的gasPrice，可以通过`GetGasPrice`获取，或者使用空字符串，使用默认值；
- gasLimit: 部署合约的gasLimit，通过[`EstimateGas`](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimateGas)可以获取部署合约的gas消耗，不能使用默认值，也可以设置一个较大值，执行时以实际使用计算。
- contract: 合约信息，部署合约时传入的参数
	- `source`: 合约代码
	- `sourceType`: 合约代码类型，支持`js`和`ts`(对应javaScript和typeScript代码)
	- `function`: 调用合约方法
	- `args`: 合约初始化方法参数，无参数为空字符串，有参数时为JSON数组

**Notice: 若为提交智能合约，传入`source`,`sourceType`和`args`参数，若为调用智能合约方法，传入`function`和`args`。**

调用智能合约的balanceOf()方法：

```js
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/call -H 'Content-Type: application/json' -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"333cb3ed8c417971845382ede3cf67a0a96270c05fe2f700","value":"0","nonce":3,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"balanceOf","args":""}}'

// Result
{
   "result": ""
}
```
智能合约的查询本质也是提交一个transation，transaction提交后只在本地执行，所以智能合约的查询立即生效。在查询方法返回结果中可以看到执行结果。
