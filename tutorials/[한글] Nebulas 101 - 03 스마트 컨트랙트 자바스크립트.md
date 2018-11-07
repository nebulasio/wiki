# Nebulas 101 - 03 스마트 컨트랙트 작성 및 실행하기

[유튜브 튜토리얼](https://www.youtube.com/watch?v=98iW0WvajVU&index=2&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R)

이 튜토리얼에서 네뷸러스 위에서 동작하는 스마트 컨트랙트를 작성하고, 배포하고, 실행하는 방법을 배울 것입니다.

## 준비

스마트 컨트랙트를 준비하기 전에, 먼저 배운 내용들을 복습하겠습니다:

1. neb 응용프로그램을 설치하고, 컴파일하고 실행합니다.
2. 지갑 주소를 생성하고, 코인베이스를 설정하고, 채굴을 시작합니다.
3. 네뷸러스의 노드 정보와 지갑 주소와 잔액을 쿼리합니다.
4. 트랜잭션을 전송하고 트랜잭션 전송이 성공했는지 확인합니다.

위 내용에 의구심이 든다면 이전의 관련된 챕터를 다시 읽어야 합니다.
다음 단계를 따라 스마트 컨트랙트를 사용하는 법을 배울 것입니다.

1. 스마트 컨트랙트를 작성합니다.
2. 커맨드라인 혹은 웹 지갑을 사용하여 스마트 컨트랙트를 배포합니다.
3. 스마트 컨트랙트를 호출하고, 컨트랙트 실행의 결과를 확인합니다.

## 스마트 컨트랙트 작성

이더리움과 같이, 네뷸러스는 스마트 컨트랙트를 실행하기 위한 네뷸러스 가상 머신 (NVM)을 실행합니다. NVM 실행은 자바스크립트 V8 엔진을 사용하고, 현재 개발을 위해서 자바스크립트와 타입스크립트를 사용하여 스마트 컨트랙트를 작성할 수 있습니다.

스마트 컨트랙트의 간단한 사양입니다:

1. 스마트 컨트랙트 코드는 프로토타입 객체가 되어야 합니다;
2. 스마트 컨트랙트 코드는 `init()` 메소드를 가져야 하고, 이 메소드는 배포하는 동안 오직 한 번 실행됩니다.
3. 스마트 컨트랙트에서 `private` 메소드는 _로 시작해야 하고, `private` 메소드는 컨트랙트 밖에서 직접 호출될 수 없습니다.

아래에 자바스크립트를 사용하여 첫 번째 스마트 컨트랙트를 작성하겠습니다: 은행 금고.
이 스마트 컨트랙트는 다음의 함수가 필요합니다:

1. 유저는 이 은행 금고에 입금할 수 있습니다.
2. 유저는 이 은행 금고에서 돈을 인출할 수 있습니다.
3. 유저는 은행 금고의 잔액을 확인할 수 있습니다.

스마트 컨트랙트 예제:

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

// 컨트랙트에 돈을 저장합니다. 유저는 블록 높이 이후에 인출할 수 있습니다.
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

위 스마트 컨트랙트 예제에서 보았듯이, `BankVaultContract`는 `init()` 메소드를 가지고 있는 프로토타입 객체입니다. 스마트 컨트랙트를 작성할 때 지켜야하는 기초적인 사양을 충족합니다. `BankVaultContract`는 두 개의 다른 메소드를 구현합니다.

- `save()`: 유저는 `save()` 메소드를 호출하여 은행 금고로 돈을 입금할 수 있습니다;
- `takeout()`: 유저는 `takeout()` 메소드를 호출하여 은행 금고에서 돈을 인출할 수 있습니다;
- `balanceof()`: 유저는 `balanceOf()` 메소드를 호출하여 은행 금고의 잔액을 확인할 수 있습니다;

위 컨트랙트 코드는 내장된 `Blockchain` 객체와 `BigNumber()` 메소드를 사용합니다. 분석한 컨트랙트 코드를 한 줄씩 알아봅시다.

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

## 커맨드라인을 사용하여 스마트 컨트랙트 배포하기

위에서 네뷸러스 안에서 스마트 컨트랙트를 어떻게 작성하는지 설명했습니다. 그리고나서 체인에 스마트 컨트랙트를 배포해야 합니다. 먼저, 네뷸러스에서 트랜잭션을 어떻게 만드는지 알아봤고, 트랜잭션을 전송하기 위해 `sendTransaction()` 인터페이스를 사용했습니다. 네뷸러스 안에서 스마트 컨트랙트를 배포하는 것은 단지 다른 매개변수로 `sendTransaction()` 인터페이스를 호출하는 것으로 트랜잭션을 전송할 수 있습니다.

```js
// transaction - from, to, value, nonce, gasPrice, gasLimit, contract
sendTransactionWithPassphrase(transaction, passphrase)
```

`from`과 `to`가 같은 주소를 써야하는 관습을 가지고 있습니다. `contract`는 `null`이 아니고 `binary`는 `null`입니다. 스마트 컨트랙트를 배포하기 위해서는 이와 같이 가정합니다.

- `from`: 생성자의 주소
- `to`: 생성자의 주소
- `value`: 컨트랙트를 배포할 때, `"0"`이 되어야 합니다.
- `nonce`: 생성자의 주소 상태의 현재 논스에서 1이 더 큰 값입니다. 논스는 [`GetAccountState`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate)에서 얻을 수 있습니다.
- `gasPrice`: gasPrice는 스마트 컨트랙트를 배포할 때 사용합니다. gasPrice는 [`GetGasPrice`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getgasprice)에서 얻을 수 있습니다. 혹은 기본값으로 `"1000000"`을 사용합니다.
- `gasLimit`: 스마트 컨트랙트 배포를 위한 gasLimit. [`EstimateGas`](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimateGas)에서 추정되는 가스 소비를 얻을 수 있고, 기본값을 사용할 수 없습니다. 또한 큰 값으로 설정할 수 있습니다. 실제 가스 소비는 배포가 실행되면서 결정됩니다.
- `contract`: 컨트랙트 정보. 매개변수는 컨트랙트가 배포될 때 전달됩니다.
  - `source`: 컨트랙트 코드
  - `sourceType`: 컨트랙트 코드 타입, `js`와 `ts` (자바스크립트와 타입스크립트 코드가 각각 해당됩니다)
  - `args`: 컨트랙트 생성 메소드를 위한 매개변수. 매개변수가 없다면 빈 문자열을 사용하고 매개변수가 있다면 하나의 JSON 배열을 사용합니다.

상세한 인터페이스 문서 [API](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#sendtransactionwithpassphrase).


`curl`을 사용하여 스마트 컨트랙트를 배포하는 예제:

```bash

> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction": {"from":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","to":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so", "value":"0","nonce":1,"gasPrice":"1000000","gasLimit":"2000000","contract":{"source":"\"use strict\";var DepositeContent=function(text){if(text){var o=JSON.parse(text);this.balance=new BigNumber(o.balance);this.expiryHeight=new BigNumber(o.expiryHeight);}else{this.balance=new BigNumber(0);this.expiryHeight=new BigNumber(0);}};DepositeContent.prototype={toString:function(){return JSON.stringify(this);}};var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\",{parse:function(text){return new DepositeContent(text);},stringify:function(o){return o.toString();}});};BankVaultContract.prototype={init:function(){},save:function(height){var from=Blockchain.transaction.from;var value=Blockchain.transaction.value;var bk_height=new BigNumber(Blockchain.block.height);var orig_deposit=this.bankVault.get(from);if(orig_deposit){value=value.plus(orig_deposit.balance);} var deposit=new DepositeContent();deposit.balance=value;deposit.expiryHeight=bk_height.plus(height);this.bankVault.put(from,deposit);},takeout:function(value){var from=Blockchain.transaction.from;var bk_height=new BigNumber(Blockchain.block.height);var
 amount=new BigNumber(value);var deposit=this.bankVault.get(from);if(!deposit){throw new Error(\"No deposit before.\");} if(bk_height.lt(deposit.expiryHeight)){throw new Error(\"Can not takeout before expiryHeight.\");} if(amount.gt(deposit.balance)){throw new Error(\"Insufficient balance.\");} var result=Blockchain.transfer(from,amount);if(!result){throw new Error(\"transfer failed.\");} Event.Trigger(\"BankVault\",{Transfer:{from:Blockchain.transaction.to,to:from,value:amount.toString()}});deposit.balance=deposit.balance.sub(amount);this.bankVault.put(from,deposit);},balanceOf:function(){var from=Blockchain.transaction.from;return this.bankVault.get(from);},verifyAddress:function(address){var result=Blockchain.verifyAddress(address);return{valid:result==0?false:true};}};module.exports=BankVaultContract;","sourceType":"js", "args":""}}, "passphrase": "passphrase"}'

{"result":{"txhash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889","contract_address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM"}}
```

스마트 컨트랙트를 배포하는 반환값은 트랜잭션의 해쉬 주소인 `txhash`이고, 컨트랙트 배포 주소는 `contract_address`입니다.
반환값을 얻는 것은 컨트랙트가 성공적으로 배포되었다는 것을 보장하지 않습니다. 왜냐하면 `sendTransaction()`가 비동기 과정이기 때문에 채굴자에게 패키징되어야합니다. 이전의 전송 트랜잭션으로써, 그 전송은 실시간으로 도착하지 않습니다. 이것은 채굴자의 패키징 속도에 의존합니다. 따라서 잠시 기다려야 합니다 (약 1분), 그리고나서 컨트랙트 주소를 쿼리하고 스마트 컨트랙트를 호출하는 것으로 컨트랙트가 성공적으로 배포되었는지 확인할 수 있습니다.

> **컨트랙트 배포가 성공했는지 확인하기**
>
> 컨트랙트 배포가 성공했는지 확인하기 위해 [`GetTransactionReceipt`](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt)를 통해 배포 트랜잭션의 영수증을 확인합니다.
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889"}'
>
> {"result":{"hash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889","chainId":100,"from":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","to":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","value":"0","nonce":"1","timestamp":"1524711841","type":"deploy","data":"eyJTb3VyY2VUeXBlIjoianMiLCJTb3VyY2UiOiJcInVzZSBzdHJpY3RcIjt2YXIgRGVwb3NpdGVDb250ZW50PWZ1bmN0aW9uKHRleHQpe2lmKHRleHQpe3ZhciBvPUpTT04ucGFyc2UodGV4dCk7dGhpcy5iYWxhbmNlPW5ldyBCaWdOdW1iZXIoby5iYWxhbmNlKTt0aGlzLmV4cGlyeUhlaWdodD1uZXcgQmlnTnVtYmVyKG8uZXhwaXJ5SGVpZ2h0KTt9ZWxzZXt0aGlzLmJhbGFuY2U9bmV3IEJpZ051bWJlcigwKTt0aGlzLmV4cGlyeUhlaWdodD1uZXcgQmlnTnVtYmVyKDApO319O0RlcG9zaXRlQ29udGVudC5wcm90b3R5cGU9e3RvU3RyaW5nOmZ1bmN0aW9uKCl7cmV0dXJuIEpTT04uc3RyaW5naWZ5KHRoaXMpO319O3ZhciBCYW5rVmF1bHRDb250cmFjdD1mdW5jdGlvbigpe0xvY2FsQ29udHJhY3RTdG9yYWdlLmRlZmluZU1hcFByb3BlcnR5KHRoaXMsXCJiYW5rVmF1bHRcIix7cGFyc2U6ZnVuY3Rpb24odGV4dCl7cmV0dXJuIG5ldyBEZXBvc2l0ZUNvbnRlbnQodGV4dCk7fSxzdHJpbmdpZnk6ZnVuY3Rpb24obyl7cmV0dXJuIG8udG9TdHJpbmcoKTt9fSk7fTtCYW5rVmF1bHRDb250cmFjdC5wcm90b3R5cGU9e2luaXQ6ZnVuY3Rpb24oKXt9LHNhdmU6ZnVuY3Rpb24oaGVpZ2h0KXt2YXIgZnJvbT1CbG9ja2NoYWluLnRyYW5zYWN0aW9uLmZyb207dmFyIHZhbHVlPUJsb2NrY2hhaW4udHJhbnNhY3Rpb24udmFsdWU7dmFyIGJrX2hlaWdodD1uZXcgQmlnTnVtYmVyKEJsb2NrY2hhaW4uYmxvY2suaGVpZ2h0KTt2YXIgb3JpZ19kZXBvc2l0PXRoaXMuYmFua1ZhdWx0LmdldChmcm9tKTtpZihvcmlnX2RlcG9zaXQpe3ZhbHVlPXZhbHVlLnBsdXMob3JpZ19kZXBvc2l0LmJhbGFuY2UpO30gdmFyIGRlcG9zaXQ9bmV3IERlcG9zaXRlQ29udGVudCgpO2RlcG9zaXQuYmFsYW5jZT12YWx1ZTtkZXBvc2l0LmV4cGlyeUhlaWdodD1ia19oZWlnaHQucGx1cyhoZWlnaHQpO3RoaXMuYmFua1ZhdWx0LnB1dChmcm9tLGRlcG9zaXQpO30sdGFrZW91dDpmdW5jdGlvbih2YWx1ZSl7dmFyIGZyb209QmxvY2tjaGFpbi50cmFuc2FjdGlvbi5mcm9tO3ZhciBia19oZWlnaHQ9bmV3IEJpZ051bWJlcihCbG9ja2NoYWluLmJsb2NrLmhlaWdodCk7dmFyIGFtb3VudD1uZXcgQmlnTnVtYmVyKHZhbHVlKTt2YXIgZGVwb3NpdD10aGlzLmJhbmtWYXVsdC5nZXQoZnJvbSk7aWYoIWRlcG9zaXQpe3Rocm93IG5ldyBFcnJvcihcIk5vIGRlcG9zaXQgYmVmb3JlLlwiKTt9IGlmKGJrX2hlaWdodC5sdChkZXBvc2l0LmV4cGlyeUhlaWdodCkpe3Rocm93IG5ldyBFcnJvcihcIkNhbiBub3QgdGFrZW91dCBiZWZvcmUgZXhwaXJ5SGVpZ2h0LlwiKTt9IGlmKGFtb3VudC5ndChkZXBvc2l0LmJhbGFuY2UpKXt0aHJvdyBuZXcgRXJyb3IoXCJJbnN1ZmZpY2llbnQgYmFsYW5jZS5cIik7fSB2YXIgcmVzdWx0PUJsb2NrY2hhaW4udHJhbnNmZXIoZnJvbSxhbW91bnQpO2lmKCFyZXN1bHQpe3Rocm93IG5ldyBFcnJvcihcInRyYW5zZmVyIGZhaWxlZC5cIik7fSBFdmVudC5UcmlnZ2VyKFwiQmFua1ZhdWx0XCIse1RyYW5zZmVyOntmcm9tOkJsb2NrY2hhaW4udHJhbnNhY3Rpb24udG8sdG86ZnJvbSx2YWx1ZTphbW91bnQudG9TdHJpbmcoKX19KTtkZXBvc2l0LmJhbGFuY2U9ZGVwb3NpdC5iYWxhbmNlLnN1YihhbW91bnQpO3RoaXMuYmFua1ZhdWx0LnB1dChmcm9tLGRlcG9zaXQpO30sYmFsYW5jZU9mOmZ1bmN0aW9uKCl7dmFyIGZyb209QmxvY2tjaGFpbi50cmFuc2FjdGlvbi5mcm9tO3JldHVybiB0aGlzLmJhbmtWYXVsdC5nZXQoZnJvbSk7fSx2ZXJpZnlBZGRyZXNzOmZ1bmN0aW9uKGFkZHJlc3Mpe3ZhciByZXN1bHQ9QmxvY2tjaGFpbi52ZXJpZnlBZGRyZXNzKGFkZHJlc3MpO3JldHVybnt2YWxpZDpyZXN1bHQ9PTA/ZmFsc2U6dHJ1ZX07fX07bW9kdWxlLmV4cG9ydHM9QmFua1ZhdWx0Q29udHJhY3Q7IiwiQXJncyI6IiJ9","gas_price":"1000000","gas_limit":"2000000","contract_address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","status":1,"gas_used":"22016"}}
> ```
> 위에서 보았듯이, 배포 트랜잭션의 status는 1이 됩니다. 컨트랙트 배포가 성공했다는 것을 의미합니다.

## 웹 지갑을 사용해서 스마트 컨트랙트 배포하기

`curl` 커맨드를 사용해서 스마트 컨트랙트 코드를 전송하는 것에 대체하여, 웹 인터페이스에서 로컬로 코드를 복사, 붙혀넣기 할 수 있습니다. 먼저 웹 지갑 인터페이스가 필요합니다. `go-nebulas` 저장소 바깥에 `https://github.com/nebulasio/web-wallet`를 클론해주세요. 그리고 터미널창에서 `node server.js`를 실행하면 `127.0.0.1:8080/contract.html`에서 작동하는 웹 지갑 인터페이스를 브라우저에서 볼 수 있습니다. 여기 정확한 단계가 있습니다:

- `git clone https://github.com/nebulasio/web-wallet`
- `cd web-wallet && node server.js`
- open `127.0.0.1:8080/contract.html`


아래의 스크린샷에서, 스마트 컨트랙트 코드를 배포하는 단계를 볼 수 있습니다.
- 배포를 위해 네트워크를 선택합니다 (로컬, 테스트넷, 메인넷)
- `Contract`를 클릭하고 메뉴에서 `Deploy`를 클릭합니다.
- 입력칸에 컨트랙트 코드를 붙혀넣습니다.
- `keydir` 디렉토리 안에 위치한 지갑 파일을 선택합니다.
- 생성한 비밀번호로 지갑을 언락합니다.
- 웹 지갑에서 가스를 얻고 싶으면 [https://testnet.nebulas.io/claim/](https://testnet.nebulas.io/claim/)를 방문하세요.
- `Test` 혹은 `Submit` 버튼을 클릭하세요.

![웹 지갑을 사용하여 컨트랙트 배포하기](resources/101-03-deploy_contract_webwallet.png)


## 스마트 컨트랙트 메소드 호출하기

네뷸러스에서 스마트 컨트랙트 메소드를 실행하는 방법은 직접적으로 스마트 컨트랙트 메소드를 호출하는 메소드인 `sendTransactionWithPassphrase()`를 사용하여 간단합니다.

```js
// transaction - from, to, value, nonce, gasPrice, gasLimit, contract
sendTransactionWithPassphrase(transaction, passphrase)
```

- `from`: 유저의 지갑 주소
- `to`: 스마트 컨트랙트 주소
- `value`: 스마트 컨트랙트에 전송할 때 사용되는 돈의 양
- `nonce`: 생성자의 주소 상태의 현재 논스에서 1이 더 큰 값입니다. 논스는 [`GetAccountState`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate)에서 얻을 수 있습니다.
- `gasPrice`: gasPrice는 스마트 컨트랙트를 배포할 때 사용합니다. gasPrice는 [`GetGasPrice`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getgasprice)에서 얻을 수 있습니다. 혹은 기본값으로 `"1000000"`을 사용합니다.
- `gasLimit`: 스마트 컨트랙트 배포를 위한 gasLimit. [`EstimateGas`](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimateGas)에서 추정되는 가스 소비를 얻을 수 있고, 기본값을 사용할 수 없습니다. 또한 큰 값으로 설정할 수 있습니다. 실제 가스 소비는 배포가 실행되면서 결정됩니다.
- `contract`: 컨트랙트 정보. 매개변수는 컨트랙트가 배포될 때 전달됩니다.
  - `function`: 호출되는 컨트랙트 메소드
  - `args`: 컨트랙트 생성 메소드를 위한 매개변수. 매개변수가 없다면 빈 문자열을 사용하고 매개변수가 있다면 하나의 JSON 배열을 사용합니다.

예를 들어서, 스마트 컨트랙트의 `save()` 메소드를 실행해보겠습니다:

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction":{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM", "value":"100","nonce":1,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"save","args":"[0]"}}, "passphrase": "passphrase"}'

{"result":{"txhash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84","contract_address":""}}
```

> **컨트랙트 메소드 `save`의 실행이 성공했는지 확인하기**
> 컨트랙트 메소드를 실행하면 체인에 실제로 트랜잭션을 제출하는 것입니다. [`GetTransactionReceipt`](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt)를 통해 트랜잭션의 영수증을 확인하는 것을 통하여 결과를 확인할 수 있습니다.
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84"}'
>
> {"result":{"hash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84","chainId":100,"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","value":"100","nonce":"1","timestamp":"1524712532","type":"call","data":"eyJGdW5jdGlvbiI6InNhdmUiLCJBcmdzIjoiWzBdIn0=","gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20361"}}
> ```
> 위에서 보았듯이, 트랜잭션의 status가 1이 되었습니다. 이것은 컨트랙트 메소드가 성공적으로 실행되었다는 것을 의미합니다.

스마트 컨트랙트의 `takeout()` 메소드를 실행합니다:

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction":{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM", "value":"0","nonce":2,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"takeout","args":"[50]"}}, "passphrase": "passphrase"}'

{"result":{"txhash":"46a307e9beb21f52992a7512f3705fe58ee6c1887122a1b52f5ce5fd5f536a91","contract_address":""}}
```

> **컨트랙트 메소드 `takeout`의 실행이 성공했다는 것 확인하기**
> 위 컨트랙트 `save` 메소드에서, 스마트 컨트랙트 `n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM`에 100 wei (10^-18 NAS)를 입금했습니다. 컨트랙트 `takeout` 메소드를 사용해서, 100 wei에서 50 wei를 인출할 것입니다. 스마트 컨트랙트의 잔액은 50 wei이 되어야 합니다.
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM"}'
>
> {"result":{"balance":"50","nonce":"0","type":88}}
> ```
> 결과는 예상했던 것과 같습니다.

## 스마트 컨트랙트의 데이터 쿼리하기

스마트 컨트랙트 안에서, 몇몇 메소드의 실행은 체인에서 절대 변경될 수 없습니다. 이 메소드들은 블록체인에서 읽기 전용으로 데이터를 쿼리하도록 설계되었습니다. 네뷸러스 안에서 유저가 읽기 전용의 메소드를 실행하도록 `call` API를 제공합니다.

```js
// transaction - from, to, value,
 nonce, gasPrice, gasLimit, contract
call(from, to, value, nonce, gasPrice, gasLimit, contract)
```

`call`의 파라미터는 컨트랙트 메소드를 실행할 때의 파라미터와 같습니다.

스마트 컨트랙트 메소드인 `balanceOf`를 호출합니다:

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/call -H 'Content-Type: application/json' -d '{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","value":"0","nonce":3,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"balanceOf","args":""}}'

{"result":{"result":"{\"balance\":\"50\",\"expiryHeight\":\"84\"}","execute_err":"","estimate_gas":"20209"}}
```

### 다음 단계: 튜토리얼 4

 [스마트 컨트랙트 저장소](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B한글%5D%20Nebulas%20101%20-%2004%20스마트%20컨트랙트%20저장소.md)
