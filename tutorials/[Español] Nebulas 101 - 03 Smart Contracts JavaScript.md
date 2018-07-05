la dire# Nebulas 101 - 03 Escribe y ejecuta un smart contrato

Con este tutorial nos podems aprender como escribir, desplegar, y ejecutar smart contratos en Nebulas.

## Preparación

Antes de entrar el smart contrato, primero repasa la lección anterior. [02 Transacciones](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEspañol%5D%20Nebulas%20101%20-%2002%20Transacciones.md).

1. Instalar, compilar, y lanzar aplicación neb
2. Crea un cartera dirección, configura coinbase, y comienca minar
3. Consulta de información de neb nodo, cartera y saldo
4. Envia una transacción y verificar la transacción estaba con exito

Si tienes dudas sobre el contenido encima, debes ir atras a lección anterior.
Vamos hacerlo. Nos vamos aprender y usar smart contratos atraves los siguientes pasos:

1. Escribe un smart contrato
2. Desplaga el smart contrato usando command-line o web-wallet
3. Llama el smart contrato, y verifica los resultados de la ejecución del contrato.

## Escribe un smart contrato

Como Ethereum, Nebulas implementa NVM virtual maquina para ejecutar smart contratos, y la implementación de NVM utiliza la máquina de Javascript V8, entonces para el desarrollo actual, podemos escribir los smart contratos usando Javascript y TypeScript.

Escribe breves especificaciones de un smart contrato:

1. El codigo de smart contrato debe ser un objeto prototipe.
2. El codigo de smart contrato debe tener un método init(), este método sea solamente ejecutado un vez durante desplegamento.
3. Los métodos pirvados en en smart contrato debe tener el prefijo _ , y el método privado no puede ser llamado directamente afuera el contrato mismo.

Abajo nos usamos Javascript para escribir el primero smart contrato: banco seguro
Este smart contrato necesita cumplir con los siguientes funciones:

1. El usuario puede salvar diñero de este banco seguro.
2. Los usuarios pueden retirar diñero de este banco seguro.
3. Los usuaros pueden consultar el saldo de este banco seguro.

Ejemplo smart contrato:

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

// salva el valor a contrato, solamente despues la altura de bloque, usuarios pueden retirar
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

Como puedes ver de este ejemplo de smart contrato encima, `BankVaultContract` es un objeto prototipo que
tiene un método init(). Eso satisiface la especificacion más basico para escribir smart contratos que nos describimos antes.

BankVaultContract implementa dos otros métodos:

- save(): El usuario puede salvar diñero a el banco llamando el método save();
- takeout(): Los usuarios pueden retirar diñero de banco por llamar el método takeout();
- balanceOf(): El usuario pueden consultar el saldo de banco por llamar el método balanceOf();

El contrato encima usa el objeto `Blockchain` integrado y el método `BigNumber()` integrado.
Analicemos el análisis sintáctico de codigo de contrato línea por línea.

**save():**

```js

// Deposite la cantidad en la caja fuerta.

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

## Desplegar smart contrato usando web wallet

Arriba describe cómo escribir smart contrato en Nebulas, y ahora nos tenemos que desplegar el smart contrato a la cadena.

Alternativamente a enviar su smart codigo de contrato usando comando curl, tu puedes copiar y pegar el código en la interfaz de web localmente. Primero tu necesitas clonar el repositorio `https://github.com/nebulasio/web-wallet` fuera `go-nebulas`. Despues necesitas ejecutar `node server.js` en terminal. Este permite abrir su browser como Chrome a ver la interfaz de web-wallet funcionando en  `127.0.0.1:8080/contract.html`. Los pasos exactos:

- `git clone https://github.com/nebulasio/web-wallet`
- `cd web-wallet && node server.js`
- abra `127.0.0.1:8080/contract.html`

En la captura de pantalla abajo puedes ver los pasos para desplegar su código de smart contrato:
- Seleccione el red para el despliegue (local, testnet, mainnet)
- Seleccione sección `Deploy`
- Pegue su código en input caja
- Seleccione su archivo de cartera, ubicación predeterminada es `keydir`
- Desbloquee su cartera con el contraseña que has creado
- Para obtener gas para usar su cartera en testnet visita [https://testnet.nebulas.io/claim/](https://testnet.nebulas.io/claim/)
- Clic el `Test` o `Submit` botón

![Despliegue contrato usando Web wallet](resources/101-03-deploy_contract_webwallet.png)


## Desplegar smart contrato usando línea de comando (cli)

Anteriormente, habíamos introducido como hacer una transacción en Nebulas, y nos usamos interfaz sendTransaction() a iniciar una tranferencia.
Despliegue de un smart contrato en Nebulas se en realidad logra por enviar una transacción por llamar la interfaz sendTransaction(), pero com parámetros diferentes.

```js
// transaction - from, to, value, nonce, gasPrice, gasLimit, contract
sendTransactionWithPassphrase(transaction, passphrase)
```

Nos tenemos una convención si `from` y `to` son la misma dirección, `contract` no es null y `binary` es null, nos suponemos que estamos implementando un contrato.

- `from`: la dirección del creador
- `to`: la dirección del creador
- `value`: debe ser `"0"` durante el despliegue;
- `nonce`: debe ser 1 mas que el actual nonce en el estado de la cuenta del creador, que puede ser obtenido por [`GetAccountState`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate).
- `gasPrice`: El gasPrice usa para desplegar el smart contrato, que pudes obtener por  [`GetGasPrice`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getgasprice), o usando valores predeterminadas: `"1000000"`;
- `gasLimit`: El gasLimit para desplegar un contrato. Puedes estimar el consumo de gas por  [`EstimateGas`](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimateGas). Tambien puedes asignar un valor mas grande. El consumo de gas actual es decidado por el ejecución del desplegar.
- `contract`: el información del contrato, los parametros pasados en el contrato.
  - `source`: código de contract
  - `sourceType`: código de tipo, `js` and `ts` (javaScript y typeScript)
  - `args`: parametros para el método inicialzación. Usa cadenas vacías si no hay parametros, usa JSON array si hay paremetros.

Documentación Interfaz detallada  [API](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#sendtransactionwithpassphrase).

Ejemplo de despelegar un smart contrato usando curl:

```bash

> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction": {"from":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","to":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so", "value":"0","nonce":1,"gasPrice":"1000000","gasLimit":"2000000","contract":{"source":"\"use strict\";var DepositeContent=function(text){if(text){var o=JSON.parse(text);this.balance=new BigNumber(o.balance);this.expiryHeight=new BigNumber(o.expiryHeight);}else{this.balance=new BigNumber(0);this.expiryHeight=new BigNumber(0);}};DepositeContent.prototype={toString:function(){return JSON.stringify(this);}};var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\",{parse:function(text){return new DepositeContent(text);},stringify:function(o){return o.toString();}});};BankVaultContract.prototype={init:function(){},save:function(height){var from=Blockchain.transaction.from;var value=Blockchain.transaction.value;var bk_height=new BigNumber(Blockchain.block.height);var orig_deposit=this.bankVault.get(from);if(orig_deposit){value=value.plus(orig_deposit.balance);} var deposit=new DepositeContent();deposit.balance=value;deposit.expiryHeight=bk_height.plus(height);this.bankVault.put(from,deposit);},takeout:function(value){var from=Blockchain.transaction.from;var bk_height=new BigNumber(Blockchain.block.height);var amount=new BigNumber(value);var deposit=this.bankVault.get(from);if(!deposit){throw new Error(\"No deposit before.\");} if(bk_height.lt(deposit.expiryHeight)){throw new Error(\"Can not takeout before expiryHeight.\");} if(amount.gt(deposit.balance)){throw new Error(\"Insufficient balance.\");} var result=Blockchain.transfer(from,amount);if(!result){throw new Error(\"transfer failed.\");} Event.Trigger(\"BankVault\",{Transfer:{from:Blockchain.transaction.to,to:from,value:amount.toString()}});deposit.balance=deposit.balance.sub(amount);this.bankVault.put(from,deposit);},balanceOf:function(){var from=Blockchain.transaction.from;return this.bankVault.get(from);},verifyAddress:function(address){var result=Blockchain.verifyAddress(address);return{valid:result==0?false:true};}};module.exports=BankVaultContract;","sourceType":"js", "args":""}}, "passphrase": "passphrase"}'

{"result":{"txhash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889","contract_address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM"}}
```
El valor de retorno para desplegar un smart contrato es la hash dirección de la transacción `txhash` y la dirección de desplegar del contrato `contract_address`.
El valor del retorno no garantiza el éxito del desplegar del contrato, porque el `sendTransaction()` es un proceso asincrónico, que necesita ser empacado por el minero.
Como la transferida transacción anterior, la transferida no llega en tiempo real, lo depende on el velocidad de empacdo del minero.
Por lo tanto necesitamos esperar para un poco (1 minuto), después tu puedes verificar si el contrato es desplegado exitoso por consultar la dirección de contrato o llamar este smart contrato.


> **Verificar el desplegado del contrato con éxito**
>
> Consulte el recibo del desplegado transacción por
 [`GetTransactionReceipt`](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt) to verify whether the contract has been deployed successfully.
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889"}'
>
> {"result":{"hash":"aaebb86d15ca30b86834efb600f82cbcaf2d7aaffbe4f2c8e70de53cbed17889","chainId":100,"from":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","to":"n1H4MYms9F55ehcvygwWE71J8tJC4CRr2so","value":"0","nonce":"1","timestamp":"1524711841","type":"deploy","data":"eyJTb3VyY2VUeXBlIjoianMiLCJTb3VyY2UiOiJcInVzZSBzdHJpY3RcIjt2YXIgRGVwb3NpdGVDb250ZW50PWZ1bmN0aW9uKHRleHQpe2lmKHRleHQpe3ZhciBvPUpTT04ucGFyc2UodGV4dCk7dGhpcy5iYWxhbmNlPW5ldyBCaWdOdW1iZXIoby5iYWxhbmNlKTt0aGlzLmV4cGlyeUhlaWdodD1uZXcgQmlnTnVtYmVyKG8uZXhwaXJ5SGVpZ2h0KTt9ZWxzZXt0aGlzLmJhbGFuY2U9bmV3IEJpZ051bWJlcigwKTt0aGlzLmV4cGlyeUhlaWdodD1uZXcgQmlnTnVtYmVyKDApO319O0RlcG9zaXRlQ29udGVudC5wcm90b3R5cGU9e3RvU3RyaW5nOmZ1bmN0aW9uKCl7cmV0dXJuIEpTT04uc3RyaW5naWZ5KHRoaXMpO319O3ZhciBCYW5rVmF1bHRDb250cmFjdD1mdW5jdGlvbigpe0xvY2FsQ29udHJhY3RTdG9yYWdlLmRlZmluZU1hcFByb3BlcnR5KHRoaXMsXCJiYW5rVmF1bHRcIix7cGFyc2U6ZnVuY3Rpb24odGV4dCl7cmV0dXJuIG5ldyBEZXBvc2l0ZUNvbnRlbnQodGV4dCk7fSxzdHJpbmdpZnk6ZnVuY3Rpb24obyl7cmV0dXJuIG8udG9TdHJpbmcoKTt9fSk7fTtCYW5rVmF1bHRDb250cmFjdC5wcm90b3R5cGU9e2luaXQ6ZnVuY3Rpb24oKXt9LHNhdmU6ZnVuY3Rpb24oaGVpZ2h0KXt2YXIgZnJvbT1CbG9ja2NoYWluLnRyYW5zYWN0aW9uLmZyb207dmFyIHZhbHVlPUJsb2NrY2hhaW4udHJhbnNhY3Rpb24udmFsdWU7dmFyIGJrX2hlaWdodD1uZXcgQmlnTnVtYmVyKEJsb2NrY2hhaW4uYmxvY2suaGVpZ2h0KTt2YXIgb3JpZ19kZXBvc2l0PXRoaXMuYmFua1ZhdWx0LmdldChmcm9tKTtpZihvcmlnX2RlcG9zaXQpe3ZhbHVlPXZhbHVlLnBsdXMob3JpZ19kZXBvc2l0LmJhbGFuY2UpO30gdmFyIGRlcG9zaXQ9bmV3IERlcG9zaXRlQ29udGVudCgpO2RlcG9zaXQuYmFsYW5jZT12YWx1ZTtkZXBvc2l0LmV4cGlyeUhlaWdodD1ia19oZWlnaHQucGx1cyhoZWlnaHQpO3RoaXMuYmFua1ZhdWx0LnB1dChmcm9tLGRlcG9zaXQpO30sdGFrZW91dDpmdW5jdGlvbih2YWx1ZSl7dmFyIGZyb209QmxvY2tjaGFpbi50cmFuc2FjdGlvbi5mcm9tO3ZhciBia19oZWlnaHQ9bmV3IEJpZ051bWJlcihCbG9ja2NoYWluLmJsb2NrLmhlaWdodCk7dmFyIGFtb3VudD1uZXcgQmlnTnVtYmVyKHZhbHVlKTt2YXIgZGVwb3NpdD10aGlzLmJhbmtWYXVsdC5nZXQoZnJvbSk7aWYoIWRlcG9zaXQpe3Rocm93IG5ldyBFcnJvcihcIk5vIGRlcG9zaXQgYmVmb3JlLlwiKTt9IGlmKGJrX2hlaWdodC5sdChkZXBvc2l0LmV4cGlyeUhlaWdodCkpe3Rocm93IG5ldyBFcnJvcihcIkNhbiBub3QgdGFrZW91dCBiZWZvcmUgZXhwaXJ5SGVpZ2h0LlwiKTt9IGlmKGFtb3VudC5ndChkZXBvc2l0LmJhbGFuY2UpKXt0aHJvdyBuZXcgRXJyb3IoXCJJbnN1ZmZpY2llbnQgYmFsYW5jZS5cIik7fSB2YXIgcmVzdWx0PUJsb2NrY2hhaW4udHJhbnNmZXIoZnJvbSxhbW91bnQpO2lmKCFyZXN1bHQpe3Rocm93IG5ldyBFcnJvcihcInRyYW5zZmVyIGZhaWxlZC5cIik7fSBFdmVudC5UcmlnZ2VyKFwiQmFua1ZhdWx0XCIse1RyYW5zZmVyOntmcm9tOkJsb2NrY2hhaW4udHJhbnNhY3Rpb24udG8sdG86ZnJvbSx2YWx1ZTphbW91bnQudG9TdHJpbmcoKX19KTtkZXBvc2l0LmJhbGFuY2U9ZGVwb3NpdC5iYWxhbmNlLnN1YihhbW91bnQpO3RoaXMuYmFua1ZhdWx0LnB1dChmcm9tLGRlcG9zaXQpO30sYmFsYW5jZU9mOmZ1bmN0aW9uKCl7dmFyIGZyb209QmxvY2tjaGFpbi50cmFuc2FjdGlvbi5mcm9tO3JldHVybiB0aGlzLmJhbmtWYXVsdC5nZXQoZnJvbSk7fSx2ZXJpZnlBZGRyZXNzOmZ1bmN0aW9uKGFkZHJlc3Mpe3ZhciByZXN1bHQ9QmxvY2tjaGFpbi52ZXJpZnlBZGRyZXNzKGFkZHJlc3MpO3JldHVybnt2YWxpZDpyZXN1bHQ9PTA/ZmFsc2U6dHJ1ZX07fX07bW9kdWxlLmV4cG9ydHM9QmFua1ZhdWx0Q29udHJhY3Q7IiwiQXJncyI6IiJ9","gas_price":"1000000","gas_limit":"2000000","contract_address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","status":1,"gas_used":"22016"}}
> ```
> Como aparece encima, el estado de la transacción de despliegue seria 1. Eso significa el contracto se ha implementado con éxito.


## Executa Método Smart Contrato

La manera para ejecutar un método de smart contrato en Nebulas es fácil, usando el método `sendTransactionWithPassphrase` a invocarlo directament.

```js
// transaction - from, to, value, nonce, gasPrice, gasLimit, contract
sendTransactionWithPassphrase(transaction, passphrase)
```

- `from`: la cuenta del usuario
- `to`: la dirección del smart contrato
- `value`: La cantidad de diñero usado para transferencia por el smart contrato.
- `nonce`: debe ser 1 o mas que el actual nonce en el estado de la cuenta del creador, que puede ser obtenido por  [`GetAccountState`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getaccountstate).
- `gasPrice`: El gasPrice usado para desplegar el smart contrato, puede ser obtenido por [`GetGasPrice`](https://github.com/nebulasio/wiki/blob/master/rpc.md#getgasprice), o usando valores defectos  `"1000000"`;
- `gasLimit`: El gasLimit para desplegar el contrato. Tu puedes estimar el consumo de gas para el despliegue por  [`EstimateGas`](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimateGas), y no puede usar el valor defecto. También puedes asignar un valor mas grande. El actual consumo de gas es decide por el despliegue de ejecución.
- `contract`: la información de contrato, los parametros passados dentro cuando el contrato es desplegado
  - `function`: el método de contrato a para llamar
  - `args`: parametros para el método inicialzación de contrato. Usa cuerda vacía cuando no hay parametros, y usa array JSON se hay un parametro.

Por ejemplo, ejecuta método save() del smart contrato:

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction":{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM", "value":"100","nonce":1,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"save","args":"[0]"}}, "passphrase": "passphrase"}'

{"result":{"txhash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84","contract_address":""}}
```

> **Verificar la ejecución del método de contrato `save` es con éxito**
> Haciendo la ejecución del método también es en realidad enviando una transacción en cadena. Nos podemos verificar los resultados atraves de consultar los recibos de la transacción con [`GetTransactionReceipt`](https://github.com/nebulasio/wiki/blob/master/rpc.md#gettransactionreceipt).
> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84"}'
>
> {"result":{"hash":"5337f1051198b8ac57033fec98c7a55e8a001dbd293021ae92564d7528de3f84","chainId":100,"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","value":"100","nonce":"1","timestamp":"1524712532","type":"call","data":"eyJGdW5jdGlvbiI6InNhdmUiLCJBcmdzIjoiWzBdIn0=","gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20361"}}
> ```
> Como se muestra arriba, el estado de la transacción se convierte 1. Significa que el método de contrato ha sido ejecutado con éxito.

Ejecuta el método `takeout` de smart contrato:

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -H 'Content-Type: application/json' -d '{"transaction":{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM", "value":"0","nonce":2,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"takeout","args":"[50]"}}, "passphrase": "passphrase"}'

{"result":{"txhash":"46a307e9beb21f52992a7512f3705fe58ee6c1887122a1b52f5ce5fd5f536a91","contract_address":""}}
```

> **Verifica el ejecución del método de contrato `takeout` es exitoso.
> En el ejecución del método de contrato `save` arriba, nos salvamos 100 NAS en el smart contrato  `n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM`. Usando el método de contrato `takout`, nos retiramos 50 NAS de el 100 NAS.
El saldo del smart contrato debe ser 50 NAS ahora.

> ```bash
> > curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM"}'
>
> {"result":{"balance":"50","nonce":"0","type":88}}
> ```
> Los resultos como esperados.

## Consultar Dato de Smart Contrato

En un smart contrato, la ejecución de algunos métodos no se cambiará en la cadena.  Estes métodos son diseñados para ayudarnos a consultar datos en modo ready-only de blockchains. En Nebulas, nos proporcionamos un API `call` para usuarios a ejecutar estes métodos read-only.

```js
// transaction - from, to, value, nonce, gasPrice, gasLimit, contract
call(from, to, value, nonce, gasPrice, gasLimit, contract)
```

Los parametros de `call` son los mismos como los parametros de executar un método contrato.


LLama el smart contrato método `balanceOf`:

```bash
> curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/call -H 'Content-Type: application/json' -d '{"from":"n1LkDi2gGMqPrjYcczUiweyP4RxTB6Go1qS","to":"n1rVLTRxQEXscTgThmbTnn2NqdWFEKwpYUM","value":"0","nonce":3,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"balanceOf","args":""}}'

{"result":{"result":"{\"balance\":\"50\",\"expiryHeight\":\"84\"}","execute_err":"","estimate_gas":"20209"}}
```

### Solución de problem Paso 03
El codigo de Web-wallet está configurado a escuchar puero 8080. Si tienes conflicto con este puerto, tienes que cambiar el puerto por modificar `server.listen(8080)` en el archivo `server.js`.

### Próximo Paso: Tutorial 4

 [Smart Contract Storage](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEspa%C3%B1ol%5D%20Nebulas%20101%20-%2004%20Smart%20Contract%20Storage.md)
