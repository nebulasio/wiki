# Nebulas 101 - 03 スマートコントラクトを作って走る

[YouTube チュートリアル](https://www.youtube.com/watch?v=98iW0WvajVU&index=2&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R)

このチュートリアルでどうスマートコントラクトを書くか、配備するか、または実行するかをNebulasで学ぶ。

## 前提準備

スマートコントラクトを準備する前に、ひとまず前に学ぶ内容を復習する:

1. nebアプリをインストール、コンパイルとスタートする
2. ワーレットアドレスを作る、コインベースをセットアップする、発掘を開始する。
3. nebノードの情報、ワーレットアドレスとバランスをクエリする
4. 交易を発信して、成功するかどうかを検証する

上記の内容に疑問がある場合は、前の関連のある章を再度読んでください。スマートコントラクトをどう使用するかを以下の手順で学びましょう:

1. スマートコントラクトを書く
2. スマートコントラクトを配備する
3. スマートコントラクトを呼び出して、コントラクト実行の結果を検証する

## スマートコントラクトを書く

イーサリアムのように、 NebulasはNebulas仮想マシン(NVM)を導入してスマートコントラクトを走る。そしてNVMの導入はJavaScript V8エンジンを使用する。だから、現在の開発ではJavaScriptとTypeScriptを使用してスマートコントラクトを書くことができます。

簡単なスマートコントラクトの仕様を書く:

1. スマートコントラクトコードはプロトタイプオブジェクトになければならない;
2. スマートコントラクトコードはinit()方法を持っていなければならない。この方法は配備中でただ一回で実行される;
3. スマートコントラクトでプライベートの方法は_でプレフィックスしなければならない。そしてこのプライベートの方法はこのコントラクト以外で直接に呼び出しできない;

以下はJavaScriptを使用して初めてのスマートコントラクトをかく:バンクセーフ。
このスマートコントラクトは以下の機能を満足するべき:

1. ユーザーがこのバンクセーフに貯金することができる。
2. ユーザーがこのバンクセーフから金を引き出すことができる。
3. ユーザーがこのバンクセーフのバランスをチェックすることができる。

スマートコントラクトの例:

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

// save value to contract, only after height of block, users can withdraw
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

以上のスマートコントラクトの例のように、 `BankVaultContract`はinit()方法を持つ一つのプロトタイプオブジェクトである。これ以前に説明したスマートコントラクトを書くための最も基本的な仕様を満たしている。BankVaultContractは他の二つの方法を導入する:

- save(): ユーザーがsave()方法を呼び出すことでバンクセーフに貯金することができる;
- takeout(): ユーザーがtakeout()方法を呼び出すことでバンクセーフから金を引き出すことができる;
- balanceOf(): ユーザーがbalanceOf()方法を呼び出すことでバンクセーフのバランスをチェックすることができる;

以上のコントラクトコードはビルドイン `Blockchain` オブジェクトとビルドイン `BigNumber()` 方法を使用する。
コントラクトコードの解析を1行ずつ細かみましょう。

**save():**

```js

// Deposit an amount to the safe

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

**takeout ():**

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

## スマートコントラクトを配備する

以上はNebulasでどうスマートコントラクトを書くかを説明する。そして今はチェンにスマートコントラクトを配備する必要がある。
より前に、Nebulasで交易を行う方法を紹介する。そしてsendTransaction()インターフェースを使用して転送を始める。Nebulasでスマートコントラクトを配備することには、異なるパラメーターでsendTransaction()インターフェースを呼び出して交易を送信することによって実際に達成される。

```js
// transaction - from, to, value, nonce, gasPrice, gasLimit, contract
sendTransactionWithPassphrase(transaction, passphrase)
```

`from`と`to`が同じアドレスであればコンベンションがある。`contract`はヌルでなく`binary`はヌルとする。スマートコントラクトを配備することを仮定する。

- `from`: クリエーターのアドレス
- `to`: クリエーターのアドレス
- `value`: コントラクトを配備するには`"0"`でなければならない;
- `nonce`: クリエーターのアカウント状態に今のノンスよりも1だけ大きくする必要がある。これは[`GetAccountState`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate)を通じて得ることができる。
- `gasPrice`: gasPriceはスマートコントラクトを配備することに使用する。これは[`GetGasPrice`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getgasprice)を通じて得ることができる。もしくはデフォルト値: `"1000000"`を使用する;
- `gasLimit`: コントラクト配備ためのgasLimit。[`EstimateGas`](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimateGas)を通じて配備の時の推定ガス消費量がえることができる。そしてデフォルト値の使用はできない。もっと大きい値を設置しなければならない。配備の実行で実際のガス消耗を決定する。
- `contract`: コントラクトの情報。
コントラクトが配備されたことでパラメーターは合格する。
  - `source`: コントラクトコード
  - `sourceType`: コントラクトコードタイプ。`js`と`ts`(それぞれのjavaScriptとtypeScriptのコードに対応する)
  - `args`: コントラクト初期化方法ためのパラメーター。パラメーターがなければ空白のままにして、あるとしたらJSONの配列を使用する。

インターフェース詳細ドキュメント [API](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#sendtransactionwithpassphrase).

curlを使用してスマートコントラクトを配備するの例:

```bash

> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction": {"from":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","to":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so", "value":"0","nonce":1,"gasPrice":"1000000","gasLimit":"2000000","contract":{"source":"\"use strict\";var DepositeContent=function(text){if(text){var o=JSON.parse(text);this.balance=new BigNumber(o.balance);this.expiryHeight=new BigNumber(o.expiryHeight);}else{this.balance=new BigNumber(0);this.expiryHeight=new BigNumber(0);}};DepositeContent.prototype={toString:function(){return JSON.stringify(this);}};var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\",{parse:function(text){return new DepositeContent(text);},stringify:function(o){return o.toString();}});};BankVaultContract.prototype={init:function(){},save:function(height){var from=Blockchain.transaction.from;var value=Blockchain.transaction.value;var bk_height=new BigNumber(Blockchain.block.height);var orig_deposit=this.bankVault.get(from);if(orig_deposit){value=value.plus(orig_deposit.balance);} var deposit=new DepositeContent();deposit.balance=value;deposit.expiryHeight=bk_height.plus(height);this.bankVault.put(from,deposit);},takeout:function(value){var from=Blockchain.transaction.from;var bk_height=new BigNumber(Blockchain.block.height);var
 amount=new BigNumber(value);var deposit=this.bankVault.get(from);if(!deposit){throw new Error(\"No deposit before.\");} if(bk_height.lt(deposit.expiryHeight)){throw new Error(\"Can not takeout before expiryHeight.\");} if(amount.gt(deposit.balance)){throw new Error(\"Insufficient balance.\");} var result=Blockchain.transfer(from,amount);if(!result){throw new Error(\"transfer failed.\");} Event.Trigger(\"BankVault\",{Transfer:{from:Blockchain.transaction.to,to:from,value:amount.toString()}});deposit.balance=deposit.balance.sub(amount);this.bankVault.put(from,deposit);},balanceOf:function(){var from=Blockchain.transaction.from;return this.bankVault.get(from);},verifyAddress:function(address){var result=Blockchain.verifyAddress(address);return{valid:result==0?false:true};}};module.exports=BankVaultContract;","sourceType":"js", "args":""}}, "passphrase": "passphrase"}'

{"result":{"txhash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889","contract_address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM"}}
```

スマートコントラクトを配備ためのリターン値は交易のハッシュアドレス `txhash`。そしてコントラクトの配備アドレスは`contract_address`である。リターン値を取得するとはコントラクトの成功配備を保証しない。なぜなら、sendTransaction () は非同期のプロセスであり、マイナーによってパッケージする必要がある。前のような転送交易のように、転送はリアルタイムには辿り着かない、マイナーのパッキングスピードに依頼する。よって一時的(1分ぐらい)を待っていなければならない。そしてコントラクトアドレスをクエリすることやスマートコントラクトを呼び出すことでコントラクトの配備が成功するかどうかを検証する。

> **コントラクトの配備が成功するかの検証**
>
> [`GetTransactionReceipt`](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt)を通じて交易配備のレシートをチェックして、コントラクトの配備は成功かどうかを検証する。
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889"}'
> 
> {"result":{"hash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889","chainId":100,"from":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","to":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","value":"0","nonce":"1","timestamp":"1524711841","type":"deploy","data":"eyJTb3VyY2VUeXBlIjoianMiLCJTb3VyY2UiOiJcInVzZSBzdHJpY3RcIjt2YXIgRGVwb3NpdGVDb250ZW50PWZ1bmN0aW9uKHRleHQpe2lmKHRleHQpe3ZhciBvPUpTT04ucGFyc2UodGV4dCk7dGhpcy5iYWxhbmNlPW5ldyBCaWdOdW1iZXIoby5iYWxhbmNlKTt0aGlzLmV4cGlyeUhlaWdodD1uZXcgQmlnTnVtYmVyKG8uZXhwaXJ5SGVpZ2h0KTt9ZWxzZXt0aGlzLmJhbGFuY2U9bmV3IEJpZ051bWJlcigwKTt0aGlzLmV4cGlyeUhlaWdodD1uZXcgQmlnTnVtYmVyKDApO319O0RlcG9zaXRlQ29udGVudC5wcm90b3R5cGU9e3RvU3RyaW5nOmZ1bmN0aW9uKCl7cmV0dXJuIEpTT04uc3RyaW5naWZ5KHRoaXMpO319O3ZhciBCYW5rVmF1bHRDb250cmFjdD1mdW5jdGlvbigpe0xvY2FsQ29udHJhY3RTdG9yYWdlLmRlZmluZU1hcFByb3BlcnR5KHRoaXMsXCJiYW5rVmF1bHRcIix7cGFyc2U6ZnVuY3Rpb24odGV4dCl7cmV0dXJuIG5ldyBEZXBvc2l0ZUNvbnRlbnQodGV4dCk7fSxzdHJpbmdpZnk6ZnVuY3Rpb24obyl7cmV0dXJuIG8udG9TdHJpbmcoKTt9fSk7fTtCYW5rVmF1bHRDb250cmFjdC5wcm90b3R5cGU9e2luaXQ6ZnVuY3Rpb24oKXt9LHNhdmU6ZnVuY3Rpb24oaGVpZ2h0KXt2YXIgZnJvbT1CbG9ja2NoYWluLnRyYW5zYWN0aW9uLmZyb207dmFyIHZhbHVlPUJsb2NrY2hhaW4udHJhbnNhY3Rpb24udmFsdWU7dmFyIGJrX2hlaWdodD1uZXcgQmlnTnVtYmVyKEJsb2NrY2hhaW4uYmxvY2suaGVpZ2h0KTt2YXIgb3JpZ19kZXBvc2l0PXRoaXMuYmFua1ZhdWx0LmdldChmcm9tKTtpZihvcmlnX2RlcG9zaXQpe3ZhbHVlPXZhbHVlLnBsdXMob3JpZ19kZXBvc2l0LmJhbGFuY2UpO30gdmFyIGRlcG9zaXQ9bmV3IERlcG9zaXRlQ29udGVudCgpO2RlcG9zaXQuYmFsYW5jZT12YWx1ZTtkZXBvc2l0LmV4cGlyeUhlaWdodD1ia19oZWlnaHQucGx1cyhoZWlnaHQpO3RoaXMuYmFua1ZhdWx0LnB1dChmcm9tLGRlcG9zaXQpO30sdGFrZW91dDpmdW5jdGlvbih2YWx1ZSl7dmFyIGZyb209QmxvY2tjaGFpbi50cmFuc2FjdGlvbi5mcm9tO3ZhciBia19oZWlnaHQ9bmV3IEJpZ051bWJlcihCbG9ja2NoYWluLmJsb2NrLmhlaWdodCk7dmFyIGFtb3VudD1uZXcgQmlnTnVtYmVyKHZhbHVlKTt2YXIgZGVwb3NpdD10aGlzLmJhbmtWYXVsdC5nZXQoZnJvbSk7aWYoIWRlcG9zaXQpe3Rocm93IG5ldyBFcnJvcihcIk5vIGRlcG9zaXQgYmVmb3JlLlwiKTt9IGlmKGJrX2hlaWdodC5sdChkZXBvc2l0LmV4cGlyeUhlaWdodCkpe3Rocm93IG5ldyBFcnJvcihcIkNhbiBub3QgdGFrZW91dCBiZWZvcmUgZXhwaXJ5SGVpZ2h0LlwiKTt9IGlmKGFtb3VudC5ndChkZXBvc2l0LmJhbGFuY2UpKXt0aHJvdyBuZXcgRXJyb3IoXCJJbnN1ZmZpY2llbnQgYmFsYW5jZS5cIik7fSB2YXIgcmVzdWx0PUJsb2NrY2hhaW4udHJhbnNmZXIoZnJvbSxhbW91bnQpO2lmKCFyZXN1bHQpe3Rocm93IG5ldyBFcnJvcihcInRyYW5zZmVyIGZhaWxlZC5cIik7fSBFdmVudC5UcmlnZ2VyKFwiQmFua1ZhdWx0XCIse1RyYW5zZmVyOntmcm9tOkJsb2NrY2hhaW4udHJhbnNhY3Rpb24udG8sdG86ZnJvbSx2YWx1ZTphbW91bnQudG9TdHJpbmcoKX19KTtkZXBvc2l0LmJhbGFuY2U9ZGVwb3NpdC5iYWxhbmNlLnN1YihhbW91bnQpO3RoaXMuYmFua1ZhdWx0LnB1dChmcm9tLGRlcG9zaXQpO30sYmFsYW5jZU9mOmZ1bmN0aW9uKCl7dmFyIGZyb209QmxvY2tjaGFpbi50cmFuc2FjdGlvbi5mcm9tO3JldHVybiB0aGlzLmJhbmtWYXVsdC5nZXQoZnJvbSk7fSx2ZXJpZnlBZGRyZXNzOmZ1bmN0aW9uKGFkZHJlc3Mpe3ZhciByZXN1bHQ9QmxvY2tjaGFpbi52ZXJpZnlBZGRyZXNzKGFkZHJlc3MpO3JldHVybnt2YWxpZDpyZXN1bHQ9PTA/ZmFsc2U6dHJ1ZX07fX07bW9kdWxlLmV4cG9ydHM9QmFua1ZhdWx0Q29udHJhY3Q7IiwiQXJncyI6IiJ9","gas_price":"1000000","gas_limit":"2000000","contract_address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","status":1,"gas_used":"22016"}}
> ```
> 上記のように、交易配備の状態は1。コントラクトが成功に配備したと意味する。

## スマートコントラクトの方法を実行する

Nebulasでスマートコントラクトの方法を実行する道も簡単である。sendTransactionWithPassphrase() 方法を使用して直接にスマートコントラクトの方法を呼び出す。

```js
// transaction - from, to, value, nonce, gasPrice, gasLimit, contract
sendTransactionWithPassphrase(transaction, passphrase)
```

- `from`: ユーザーのアカウントアドレス
- `to`: スマートコントラクトのアドレス
- `value`: スマートコントラクトによって転送される金額。
- `nonce`: クリエーターのアカウント状態に今のノンスよりも1だけ大きくする必要がある。これは[`GetAccountState`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate)を通じて得ることができる。
- `gasPrice`: gasPriceはスマートコントラクトを配備することに使用する。これは[`GetGasPrice`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getgasprice)を通じて得ることができる。もしくはデフォルト値: `"1000000"`を使用する;
- `gasLimit`: コントラクト配備ためのgasLimit。[`EstimateGas`](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimateGas)を通じて配備の時の推定ガス消費量がえることができる。そしてデフォルト値の使用はできない。もっと大きい値を設置しなければならない。配備の実行で実際のガス消耗を決定する。
- `contract`: コントラクトの情報。コントラクトが配備されたことでパラメーターは合格する。
  - `function`: コントラクトを呼び出すための方法
  - `args`: コントラクト初期化方法ためのパラメーター。パラメーターがなければ空白のままにして、あるとしたらJSONの配列を使用する。

例として、スマートコントラクトのsave()方法を実行する:

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction":{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM", "value":"100","nonce":1,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"save","args":"[0]"}}, "passphrase": "passphrase"}'

{"result":{"txhash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84","contract_address":""}}
```

> **コントラクト方法`save`が成功するかを検証する**
> コントラクトを実行することは実にチェンに交易を差し出すことに同じである。[`GetTransactionReceipt`](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt)を通じて交易のレシートをチェックして結果を検証することができる。
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84"}'
> 
> {"result":{"hash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84","chainId":100,"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","value":"100","nonce":"1","timestamp":"1524712532","type":"call","data":"eyJGdW5jdGlvbiI6InNhdmUiLCJBcmdzIjoiWzBdIn0=","gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20361"}}
> ```
> 上記のように、交易の状態は1。コントラクト方法が成功に実行したと意味する。

スマートコントラクトのtakeout()方法を実行する:

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction":{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM", "value":"0","nonce":2,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"takeout","args":"[50]"}}, "passphrase": "passphrase"}'

{"result":{"txhash":"46a307e9beb21f52992a7512f3705fe58ee6c1887122a1b52f5ce5fd5f536a91","contract_address":""}}
```

> **コントラクト方法`takeout`が成功するかを検証する**
> 前のコントラクト方法`save`の実行で、スマートコントラクト`n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM`に100 wei (10^-18 NAS)を貯金する。コントラクト方法`takeout`を使用すると、100 weiから50 weiを引き出す。今のスマートコントラクトバランスが50 weiになっているはず。
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM"}'
>
> {"result":{"balance":"50","nonce":"0","type":88}}
> ```
> 結果は予想通りである。

## スマートコントラクトデータをクエリする

スマートコントラクトでの一部の方法の実行がチェン上のものを何も変更しない。これらの方法はブロックチェーンから読み取り専用モードでデータをクエリするに役立つで設計されている。NebulasでAPI `call`をユーザーに提供してこれらの読み取り専用の方法を実行する。

```js
// transaction - from, to, value,
 nonce, gasPrice, gasLimit, contract
call(from, to, value, nonce, gasPrice, gasLimit, contract)
```

`call`のパラメーターはコントラクト方法の実行ときのパラメーターに同じである。

スマートコントラクト`balanceOf`方法を呼び出す:

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/call -H 'Content-Type: application/json' -d '{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","value":"0","nonce":3,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"balanceOf","args":""}}'

{"result":{"result":"{\"balance\":\"50\",\"expiryHeight\":\"84\"}","execute_err":"","estimate_gas":"20209"}}
```

### 次の章: チュートリアル 4

 [スマート・コントラクトメモリ](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2004%20Smart%20Contract%20Storage.md)
