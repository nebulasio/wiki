# Nebulas 101 - 03 Escrever e executar um smart contract

[Tutorial YouTube](https://www.youtube.com/watch?v=98iW0WvajVU&index=2&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R)

Através deste tutorial irá aprender como escrever, implementar, e executar smart contracts em Nebulas.

## Preparação

Antes de preencher o smart contract, primeiro reveja os conteúdos préviamente lecionados:

1. Instalação, compilação e inicialização de uma aplicação neb
2. Criação de um endereço da carteira, configuração da coinbase, a inicialização de mineração
3. Inquirir informação ao nó de neb, endereço da carteira, e balanço
4. Envio de transação e verificação do sucesso da mesma

Se tiver dúvidas dos conteúdos acima deve voltar atrás para rever os capítulos relevantes.
Vamos fazer isto. Vamos aprender a usar smart contracts através das seguintes etapas:

1. Escrever o smart contract
2. Implementar o smart contract
3. Chamar o smart contract, e verificar os resultados da sua execução

## Escrever um Smart Contract

Tal como o Ethereum, Nebulas implementa uma máquina virtual (NVM) para executar smart contracts. A implementação do NVM usa o interpretador JavaScript V8, portanto de momento podemos escrever smart contracts usando JavaScript e TypeScript.

Escreve uma especificação breve de um smart contract:

1. O código do smart contract tem de ser um objecto Prototype;
2. O código do smart contract tem de ter um método init(), que apenas será executado uma vez durante a implementação;
3. Os métodos privados do smart contract tem de ser prefixados com _ , e não podem ser chamados directamente for a do contracto;

Abaixo usamos JavaScript para escrever o primeiro smart contract: cofre de banco.
Este smart contract tem de cumprir as seguintes funções:

1. Utilizadores pode depositar dinheiro neste cofre bancário.
2. Utilizadores podem levantar dinheiro deste cofre bancário.
3. Utilizadores podem verificar o balanço deste cofre bancário.

Exemplo de smart contract:

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

// guarda valor do contracto, apenas após `height` do bloco, utilizadores podem levantar
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
Como pode ver no exemplo do smart contracto acima, `BankVaultContract` é um objecto protótipo com um método init(). Satisfaz a especificação mais básica para escrever smart contracts como descrito préviamente.
BankVaultContract implementa dois outros métodos:

- save(): Utilizadores podem depositar dinheiro no cofre do banco ao chamar o método save();
- takeout(): Utilizadores podem levantar dinheiro no cofre do banco ao chamar o método takeout();
- balanceOf(): Utilizadores podem verificar o balanço do cofre do banco ao chamar o método balanceOf();

O código do contracto acima usa o objecto embutido `Blockchain` e o método embutido `BigNumber()`. Vamos esmiuçar a análise do código do contracto linha por linha:

**save():**

```js

// Deposita uma quantidade no cofre

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

## Implementação de smart contracts

Aprendeu a escrever um smart contract em Nebulas, e agora precisamos de o implementar na chain.
Antes, foi introduzido como fazer uma transação em Nebulas, e usámos a interface sendTransaction() para iniciar a transferência. Implementar um smart contract em Nebulas é efectuado ao enviar uma transação, chamando a interface sendTransaction(), apenas com parâmetros diferentes. 

```js
// transaction – from, to, value, nonce, gasPrice, gasLimit, contract
sendTransactionWithPassphrase(transaction, passphrase)
```

Obedecemos a uma convenção que se `from` e `to` tiverem o mesmo endereço, `contract` não é nulo e `binary` é nulo, assumimos que estamos a implementar um smart contract.

- `from`: endereço do criador
- `to`: endereço do criador 
- `value`: deve ser `"0"` ao implementar o contracto;
- `nonce`: deve ser um a somar ao nonce actual no estado da conta do criador, que pode ser obtido via [`GetAccountState`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate).
- `gasPrice`: o gasPrice usado para implementar o smart contract, que pode ser obtido via [`GetGasPrice`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getgasprice), or using default values: `"1000000"`;
- `gasLimit`: o gasLimit para implementar o contracto. Pode obter uma estimativa do consumo do gás para implementação via [`EstimateGas`](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimateGas), e não pode usar o valor padrão. Também pode definir um valor maior. O consumo real de gás é decidido pela execução da implementação.
- `contract`: a informação do contracto, os parâmetros passados na implementação do contracto
  - `source`: código fonte do contracto
  - `sourceType`: tipo do código do contracto, `js`, e `ts` (correspondentes ao código de JavaScript e typeScript)
  - `args`: parâmetros para o método de inicialização do contracto. Usa uma string vazia se não existem parâmetros, e usa um arranjo (array) JSON se existem.

Detailed Interface Documentation [API](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#sendtransactionwithpassphrase).

Exemplo de implementação de um smart contract usando curl:

```bash

> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction": {"from":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","to":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so", "value":"0","nonce":1,"gasPrice":"1000000","gasLimit":"2000000","contract":{"source":"\"use strict\";var DepositeContent=function(text){if(text){var o=JSON.parse(text);this.balance=new BigNumber(o.balance);this.expiryHeight=new BigNumber(o.expiryHeight);}else{this.balance=new BigNumber(0);this.expiryHeight=new BigNumber(0);}};DepositeContent.prototype={toString:function(){return JSON.stringify(this);}};var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\",{parse:function(text){return new DepositeContent(text);},stringify:function(o){return o.toString();}});};BankVaultContract.prototype={init:function(){},save:function(height){var from=Blockchain.transaction.from;var value=Blockchain.transaction.value;var bk_height=new BigNumber(Blockchain.block.height);var orig_deposit=this.bankVault.get(from);if(orig_deposit){value=value.plus(orig_deposit.balance);} var deposit=new DepositeContent();deposit.balance=value;deposit.expiryHeight=bk_height.plus(height);this.bankVault.put(from,deposit);},takeout:function(value){var from=Blockchain.transaction.from;var bk_height=new BigNumber(Blockchain.block.height);var
 amount=new BigNumber(value);var deposit=this.bankVault.get(from);if(!deposit){throw new Error(\"No deposit before.\");} if(bk_height.lt(deposit.expiryHeight)){throw new Error(\"Can not takeout before expiryHeight.\");} if(amount.gt(deposit.balance)){throw new Error(\"Insufficient balance.\");} var result=Blockchain.transfer(from,amount);if(!result){throw new Error(\"transfer failed.\");} Event.Trigger(\"BankVault\",{Transfer:{from:Blockchain.transaction.to,to:from,value:amount.toString()}});deposit.balance=deposit.balance.sub(amount);this.bankVault.put(from,deposit);},balanceOf:function(){var from=Blockchain.transaction.from;return this.bankVault.get(from);},verifyAddress:function(address){var result=Blockchain.verifyAddress(address);return{valid:result==0?false:true};}};module.exports=BankVaultContract;","sourceType":"js", "args":""}}, "passphrase": "passphrase"}'

{"result":{"txhash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889","contract_address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM"}}
```

O valor retornado para implementar um smart contract é o endereço hash da transação `txhash` e o endereço da implementação do contracto é `contract_address`.
A obtenção deste valor não garante o sucesso da implementação do contracto, porque sendTransaction() é um processo assíncrono, que precisa de ser empacotado pelo mineiro. Tal como a transação prévia, a transferência não chega em tempo real, e depende da velocidade da empacotação do mineiro. Portanto precisa de esperar cerca de 1 minuto, e depois pode verificar se o contracto foi implementado com sucesso, ao inquirir o endereço do contracto, ou chamando este smart contract.

> **Verifique se a implementação do contracto foi bem sucedida**
>
> Verifique o recibo da transação da implementação via [`GetTransactionReceipt`](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt) para verificar se o contracto foi implementado com sucesso.
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889"}'
> 
> {"result":{"hash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889","chainId":100,"from":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","to":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","value":"0","nonce":"1","timestamp":"1524711841","type":"deploy","data":"eyJTb3VyY2VUeXBlIjoianMiLCJTb3VyY2UiOiJcInVzZSBzdHJpY3RcIjt2YXIgRGVwb3NpdGVDb250ZW50PWZ1bmN0aW9uKHRleHQpe2lmKHRleHQpe3ZhciBvPUpTT04ucGFyc2UodGV4dCk7dGhpcy5iYWxhbmNlPW5ldyBCaWdOdW1iZXIoby5iYWxhbmNlKTt0aGlzLmV4cGlyeUhlaWdodD1uZXcgQmlnTnVtYmVyKG8uZXhwaXJ5SGVpZ2h0KTt9ZWxzZXt0aGlzLmJhbGFuY2U9bmV3IEJpZ051bWJlcigwKTt0aGlzLmV4cGlyeUhlaWdodD1uZXcgQmlnTnVtYmVyKDApO319O0RlcG9zaXRlQ29udGVudC5wcm90b3R5cGU9e3RvU3RyaW5nOmZ1bmN0aW9uKCl7cmV0dXJuIEpTT04uc3RyaW5naWZ5KHRoaXMpO319O3ZhciBCYW5rVmF1bHRDb250cmFjdD1mdW5jdGlvbigpe0xvY2FsQ29udHJhY3RTdG9yYWdlLmRlZmluZU1hcFByb3BlcnR5KHRoaXMsXCJiYW5rVmF1bHRcIix7cGFyc2U6ZnVuY3Rpb24odGV4dCl7cmV0dXJuIG5ldyBEZXBvc2l0ZUNvbnRlbnQodGV4dCk7fSxzdHJpbmdpZnk6ZnVuY3Rpb24obyl7cmV0dXJuIG8udG9TdHJpbmcoKTt9fSk7fTtCYW5rVmF1bHRDb250cmFjdC5wcm90b3R5cGU9e2luaXQ6ZnVuY3Rpb24oKXt9LHNhdmU6ZnVuY3Rpb24oaGVpZ2h0KXt2YXIgZnJvbT1CbG9ja2NoYWluLnRyYW5zYWN0aW9uLmZyb207dmFyIHZhbHVlPUJsb2NrY2hhaW4udHJhbnNhY3Rpb24udmFsdWU7dmFyIGJrX2hlaWdodD1uZXcgQmlnTnVtYmVyKEJsb2NrY2hhaW4uYmxvY2suaGVpZ2h0KTt2YXIgb3JpZ19kZXBvc2l0PXRoaXMuYmFua1ZhdWx0LmdldChmcm9tKTtpZihvcmlnX2RlcG9zaXQpe3ZhbHVlPXZhbHVlLnBsdXMob3JpZ19kZXBvc2l0LmJhbGFuY2UpO30gdmFyIGRlcG9zaXQ9bmV3IERlcG9zaXRlQ29udGVudCgpO2RlcG9zaXQuYmFsYW5jZT12YWx1ZTtkZXBvc2l0LmV4cGlyeUhlaWdodD1ia19oZWlnaHQucGx1cyhoZWlnaHQpO3RoaXMuYmFua1ZhdWx0LnB1dChmcm9tLGRlcG9zaXQpO30sdGFrZW91dDpmdW5jdGlvbih2YWx1ZSl7dmFyIGZyb209QmxvY2tjaGFpbi50cmFuc2FjdGlvbi5mcm9tO3ZhciBia19oZWlnaHQ9bmV3IEJpZ051bWJlcihCbG9ja2NoYWluLmJsb2NrLmhlaWdodCk7dmFyIGFtb3VudD1uZXcgQmlnTnVtYmVyKHZhbHVlKTt2YXIgZGVwb3NpdD10aGlzLmJhbmtWYXVsdC5nZXQoZnJvbSk7aWYoIWRlcG9zaXQpe3Rocm93IG5ldyBFcnJvcihcIk5vIGRlcG9zaXQgYmVmb3JlLlwiKTt9IGlmKGJrX2hlaWdodC5sdChkZXBvc2l0LmV4cGlyeUhlaWdodCkpe3Rocm93IG5ldyBFcnJvcihcIkNhbiBub3QgdGFrZW91dCBiZWZvcmUgZXhwaXJ5SGVpZ2h0LlwiKTt9IGlmKGFtb3VudC5ndChkZXBvc2l0LmJhbGFuY2UpKXt0aHJvdyBuZXcgRXJyb3IoXCJJbnN1ZmZpY2llbnQgYmFsYW5jZS5cIik7fSB2YXIgcmVzdWx0PUJsb2NrY2hhaW4udHJhbnNmZXIoZnJvbSxhbW91bnQpO2lmKCFyZXN1bHQpe3Rocm93IG5ldyBFcnJvcihcInRyYW5zZmVyIGZhaWxlZC5cIik7fSBFdmVudC5UcmlnZ2VyKFwiQmFua1ZhdWx0XCIse1RyYW5zZmVyOntmcm9tOkJsb2NrY2hhaW4udHJhbnNhY3Rpb24udG8sdG86ZnJvbSx2YWx1ZTphbW91bnQudG9TdHJpbmcoKX19KTtkZXBvc2l0LmJhbGFuY2U9ZGVwb3NpdC5iYWxhbmNlLnN1YihhbW91bnQpO3RoaXMuYmFua1ZhdWx0LnB1dChmcm9tLGRlcG9zaXQpO30sYmFsYW5jZU9mOmZ1bmN0aW9uKCl7dmFyIGZyb209QmxvY2tjaGFpbi50cmFuc2FjdGlvbi5mcm9tO3JldHVybiB0aGlzLmJhbmtWYXVsdC5nZXQoZnJvbSk7fSx2ZXJpZnlBZGRyZXNzOmZ1bmN0aW9uKGFkZHJlc3Mpe3ZhciByZXN1bHQ9QmxvY2tjaGFpbi52ZXJpZnlBZGRyZXNzKGFkZHJlc3MpO3JldHVybnt2YWxpZDpyZXN1bHQ9PTA/ZmFsc2U6dHJ1ZX07fX07bW9kdWxlLmV4cG9ydHM9QmFua1ZhdWx0Q29udHJhY3Q7IiwiQXJncyI6IiJ9","gas_price":"1000000","gas_limit":"2000000","contract_address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","status":1,"gas_used":"22016"}}
> ```
> Como mostrado acima, o estado da implementação da transacção torna-se 1. Significa que o contracto foi implementado com sucesso.

## Executar Método de Smart Contract

A maneira de executar um método de smart contract em Nebulas também é directa, usando o método sendTransactionWithPassphrase() para invocar o método do smart contract directamente.

```js
// transaction - from, to, value, nonce, gasPrice, gasLimit, contract
sendTransactionWithPassphrase(transaction, passphrase)
```

- `from`: o endereço do utilizar
- `to`: o endereço do smart contract
- `value`: a quantidade de dinheiro a ser transferida pelo smart contract
- `nonce`: deve ser um a somar ao nonce actual no estado da conta do criador, que pode ser obtido via [`GetAccountState`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate).
- `gasPrice`: o gasPrice usado para implementar o smart contract, que pode ser obtido via [`GetGasPrice`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getgasprice), or using default values `"1000000"`;
- `gasLimit`: o gasLimit para implementar o contracto. Pode obter uma estimativa do consumo do gás para implementação via [`EstimateGas`](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimateGas), e não pode usar o valor padrão. Também pode definir um valor maior. O consumo real de gás é decidido pela execução da implementação.
- `contract`: a informação do contracto, os parâmetros passados na implementação do contracto
 - `function`: o método do contracto a ser chamado
 - `args`: parâmetros para o método de inicialização do contracto. Usa uma string vazia se não existem parâmetros, e usa um arranjo (array) JSON se existem.


Por exemplo, execute o método save() do smart contract:

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction":{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM", "value":"100","nonce":1,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"save","args":"[0]"}}, "passphrase": "passphrase"}'

{"result":{"txhash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84","contract_address":""}}
```

> **Verifique que a execução do método do contracto `save` foi bem sucedida**
> A execução do método do contracto é realmente a submissão de uma transação na chain. Pode verificar o resultado através da análise do recibo da transação via [`GetTransactionReceipt`](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt).
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84"}'
> 
> {"result":{"hash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84","chainId":100,"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","value":"100","nonce":"1","timestamp":"1524712532","type":"call","data":"eyJGdW5jdGlvbiI6InNhdmUiLCJBcmdzIjoiWzBdIn0=","gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20361"}}
> ```
> Como visto acima, o estado da transação da implementação torna-se 1. Significa que o método do contracto foi executado com sucesso.

Execute o método takeout() do smart contract:

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction":{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM", "value":"0","nonce":2,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"takeout","args":"[50]"}}, "passphrase": "passphrase"}'

{"result":{"txhash":"46a307e9beb21f52992a7512f3705fe58ee6c1887122a1b52f5ce5fd5f536a91","contract_address":""}}
```

> **Verifique que a execução do método `takeout` do contracto sucedeu**
> Na execução do método de contracto `save` acima descrito, depositou 100 wei (10^-18 NAS) no smart contract `n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM`. Utilizando o método de contracto `takeout`, vai levantar 50 wei dos 100 wei depositadas. O balanço do smart contract deve ser 50 wei agora.
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM"}'
>
> {"result":{"balance":"50","nonce":"0","type":88}}
> ```
> O resultado foi o esperado.

## Inquirir Dados do Smart Contract

Num smart contract, a execução de alguns métodos não muda nada na chain. Estes métodos foram engendrados para nos ajudar a inquirir dados de blockchains em modo somente de leitura (read-only). Em Nebulas, fornecemos um API `call` (chamada) para os utilizadores executarem esses métodos read-only.

```js
// transaction - from, to, value,
 nonce, gasPrice, gasLimit, contract
call(from, to, value, nonce, gasPrice, gasLimit, contract)
```

Os parâmetros de `call` são os mesmos da execução de um método de contracto.

Chame o método de smart contract `balanceOf`:

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/call -H 'Content-Type: application/json' -d '{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","value":"0","nonce":3,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"balanceOf","args":""}}'

{"result":{"result":"{\"balance\":\"50\",\"expiryHeight\":\"84\"}","execute_err":"","estimate_gas":"20209"}}
```

### Próximo passo: Tutorial 4

 [Armazenamento de Smart Contracts](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BPortugues%5D%20Nebulas%20101%20-%2004%20Armazenamento%20Smart%20Contract.md)
 

