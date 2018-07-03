# Nebulas 101 - 02 Envío de Transacciones en Nebulas

> Para este porcion del tutorial nos siguemos donde lo dejamos en el parte [01 Instalacciones ](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEspa%C3%B1ol%5D%20Nebulas%20101%20-%2001%20Instalacion.md)

Nebulas proporciona tres métodos a enviar transacciones:

1. Firmar & Enviar
2. Enviar con contraseña
3. Desbloquear & Enviar  

Aqui es un introducción para enviar una transacción en Nebulas atraves los tres métodos encima y verificando si es exitoso.

## Preparar Cuentas

En Nebulas, cada dirección representa una uniqa cuenta.

Prepara dos cuentas: una dirección a enviar tokens (la dirreción de envio, llamada "from") y una dirreción para reciber los tokens (la dirreción de envio, llamada "to").


### El Remintente

Aqui nos utilizamos la coinbase cuenta ubicado en `conf/example/miner.conf`, cual es `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE` como el remintente. Como la cuenta del minero, recibirá algunas tokens como el premio de la minería.

### El Receptor

Crea un nuevo cartera para reciber los tokens.

```bash
$ ./neb account new
Your new account is locked with a passphrase. Please give a passphrase. Do not forget this passphrase.
Passphrase:
Repeat passphrase:
Address: n1SQe5d1NKHYFMKtJ5sNHPsSPVavGzW71Wy
```

> Mientra ejecuta este comando tu tendrá una nueva dirección de cartera como  `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`. Por favor utiliza su nuevo direccíon como el receptor.

El archivo keystore de nuevo carter estará ubicado en `$GOPATH/src/github.com/nebulasio/go-nebulas/keydir/`

## Lanca Los Nodos

### Lanca Nodo Semilla

Primero, lanzarte un nodo semilla como el primero nodo en el cadena privada.

```bash
./neb -c conf/default/config.conf
```

### Lanca Nodo Minería


Segundo, lanzarte un nodo minería conectado a el nodo semilla. Este nodo generará nuevos bloques en la cadena privada.

```bash
./neb -c conf/example/miner.conf
```

> **¿Cuánto tiempo se acuñará un nuevo bloque?**
>
> En Nebulas, DPoS esta eligado como un consenso temporal.
 algoritmo antes Proof-of-Devotion(PoD, escrito en [Technical White Paper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)) está listo.
 En este consenso algoritmo, cada minero acuñara nuevos bloques uno a uno cada 15 segundos.

>
> En el contexto actual, hay que esperar for 315(=15*21) segundos para obtener un nuevo bloque porque hay solamente uno minero entre 21 mineros definido en `conf/default/genesis.conf` funcionando ahora.

En cuanto uno nuevo bloque acuñado por el minero, el premio minero seria adicionado a el coinbase dirreción de cartera marcado en `conf/example/miner.conf` que es `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`.

## Interacción con Nodos

Nebulas proporciona desarrolladores con HTTP API, gRPC API y CLI para interactuar con los nodos en ejecución. Aquí es como enviar una transacción en tres métodos con HTTP API ([API Module](https://github.com/nebulasio/wiki/blob/master/rpc.md) | [Admin Module](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)).

> El Nebulas HTTP oyente está definiado en el configuaraccion del nodo. El defecto puerto de nos nodo semilla es `8685`.

En el primero, verificar el saldo del remintente antes de enviar la transacción.

### Verificar Estado de Cuenta

Obtener el estado de cuenta del remintente `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE` con `/v1/user/accountstate` en API Module usando `curl`.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE"}'

{
    "result": {
        "balance": "67066180000000000000",
        "nonce": "0",
        "type": 87
    }
}
```

> **Aviso**
> Typo es usado para verificar si esta cuenta es un smart contrato cuenta. `88` significa
cuenta de smart contrato y `87` signfiica una cuenta sin-contrato.

Así es como nos vemos, el receptor ha recompensado algunos tokens para minar nuevos bloques.

Vamos verificar el estado de cuenta del receptor.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"your_address"}'


{
    "result": {
        "balance": "0",
        "nonce": "0",
        "type": 87
    }
}
```

La cuenta neuva no tiene tokens como se esperaba.

### Enviar una Transacción

Vamos enviar un transsacción en tres métodos para transferir algunos tokens del remintente a el receptor.

#### Firmar & Enviar

En este manera, nos podemos firmar la transacción en un ambiente offline y despues enviarlo a un otro nodo. Este manera es mas sugera para todos a enviar una transacción sin explotar su clave privada a el internet.

Primero, firmar la transacción para obtener los datos sin procesar.


```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}, "passphrase":"passphrase"}'

{"result":{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}}
```

> **Aviso**
> Nonce es un importante atributo en la transacción. Es definido para preventir  [replay ataques](https://en.wikipedia.org/wiki/Replay_attack).
Para cualquier cuenta dada, solamente después su transacción con nonce N sea aceptada, se procesará su transacción con nonce N+1. Por lo tanto, tenemos que verificar el último nonce de la cuenta en cadena antes de preperando una nueva transacctión.

Después, enviar los datos a un nodo Nebulas en línea.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/rawtransaction -d '{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}'

{"result":{"txhash":"1b8cc3f977256c4d620b7d72f531bc19f10eb13a05ff24a8a792cd5da53a1277","contract_address":""}}⏎
```

#### Enviar con llave

Si tienes confianza de nodo Nebulas tanto que puedes delegar tu archivos de keystore al nodo,
el segundo método es mejor.

Primero, suba su archivo keystore a el directorio keydir en el nodo Nebulas de confianza.

Después, enviar la transacción con su contraseña.


```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":2,"gasPrice":"1000000","gasLimit":"2000000"},"passphrase":"passphrase"}'

{"result":{"txhash":"3cdd38a66c8f399e2f28134e0eb556b292e19d48439f6afde384ca9b60c27010","contract_address":""}}
```

> **Aviso**
> Por que tenemos ha enviado la transacción con nonce 1 de la cuenta `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`, transacción neuva con el mismo `from` debe aumentarse en 1 o 2.

#### Desbloquer & Enviar

Este manera es método mas peligroso. No debes usarlo a menos que tengas plena confianza en el nodo receptor de Nebulas.

Primero, suba su clave archivos a el directorio keydir en el nodo Nebulas de confianza.

Después desbloquear sus cuentas con su clave para el duración dado en el nodo.

La unidad del duración es sugundos nanos (300000000000=300s).

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","passphrase":"passphrase","duration":"300000000000"}'

{"result":{"result":true}}
```

Despues desbloquear de la cuenta, todo el mundo puede enviar cualquier transacción directamente entre el duración en este nodo sin autorización.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transaction -d '{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":3,"gasPrice":"1000000","gasLimit":"2000000"}'

{"result":{"txhash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","contract_address":""}}⏎
```

## Recibo de la Transacción

Nos obtenemos un `txhash` en tres métodos después de enviar una transacción exitosa.
 El valor de `txhash` puede ser utilizado para consultar el estado la transacción.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf"}'

{"result":{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","chainId":100,"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","value":"1000000000000000000","nonce":"3","timestamp":"1524667888","type":"binary","data":null,"gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20000"}}⏎
```

Los campos `status` pueden ser 0, 1 or 2.

- **0: Falló.** Significa que la transacción ha enviado en cadena pero su ejecución falló.
- **1: Exito.** Significa que la transacción ha enviado en cadena y su ejecución tuvo éxito.
- **2: Pendiente.** Significa que la transacción no ha paquedo en el bloque.

### Verificar de nuevo

Vamos verificar el saldo del receptor:

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5"}'

{"result":{"balance":"3000000000000000000","nonce":"0","type":87}}
```
Aquí deberías ver un saldo que es de todas las transferencias exitosas.

### Solución de problema Paso 02

Si tu maquina no puede resolver `http://localhost` con el comando `curl`, verifica que tu archivo  /etc/hosts tiene:
```bash
::1             localhost
```

### Próximo Paso: Tutorial 3

[Smart Contratos Javascript en Nebulas](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEspa%C3%B1ol%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20Javascript.md)
