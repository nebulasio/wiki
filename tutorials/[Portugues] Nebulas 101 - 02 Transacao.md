# Nebulas 101 - 02 Sending Transactions on Nebulas

[Tutorial Youtube](https://www.youtube.com/watch?v=-44tVVR6ETo&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R&index=1)

> Nesta porção do tutorial vamos continuar o que estávamos a fazer no [Tutorial de Instalação 1](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BPortugues%5D%20Nebulas%20101%20-%2001%20Instalacao.md).

Nebulas dispõe de três métodos para enviar transações:

1. Assina & Envia
2. Envia com senha
3. Desbloqueia & Envia

Aqui está uma introdução ao envio de uma transação de Nebulas através dos três métodos acima descritos, e a verificação do sucesso da transação.

## Preparação de Contas

Com Nebulas, cada endereço representa uma conta única.

Prepara duas contas: um endereço para enviar tokens (o endereço de envio/remetente, chamado “from”), e o endereço para receber os tokens (o endereço de recepção/destinatário, chamado “to”).

### O Remetente

Aqui vamos utilizar uma conta Coinbase no `conf/example/miner.conf`, que usa `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE` como remetente. Sendo a conta Coinbase do mineiro, irá receber uns tokens como recompensa de mineração. Mais tarde podemos enviar esses tokens para outra conta.

### O Destinatário

Crie uma nova wallet/carteira para receber os tokens.

```bash
$ ./neb account new
A sua conta está bloqueada com uma senha, Por favor defina uma senha. Não se esqueça desta senha.
Passphrase:
Repeat passphrase:
Address: n1SQe5d1NKHYFMKtJ5sNHPsSPVavGzW71Wy
```

> Quando executar este comando a carteira/wallet terá um endereço diferente, com `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`. Por favor use o endereço gerado como o destinatário.

O ficheiro keystore da nova carteira ficará em `$GOPATH/src/github.com/nebulasio/go-nebulas/keydir/`

## Activação dos Nós

### Activação do Nó Semente

Primeiro, active o nó semente como o primeiro nó da blockchain local privada.

```bash
./neb -c conf/default/config.conf
```

### Active o Nó Mineiro

Em segundo lugar, inicíe o nó mineiro ligado ao nó semente. Este nó irá gerar novos blocos na chain local privada.

```bash
./neb -c conf/example/miner.conf
``` 

> **Quanto tempo até um novo bloco ser cunhado?**
> 
> Em Nebulas, DpoS foi escolhido como o algoritmo para o mecanismo temporário de consenso antes do Proof-of-Devotion (PoD, descrita no [White Paper Técnico](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)) estar pronto. Neste algoritmo de consenso, cada mineiro vai cunhar um novo bloco a cada 15 segundos.
> 
> Em contexto corrente, temos de esperar 315(=15*21) segundos até cunhar um novo bloco pois apenas há um mineiro a trabalhar agora, entre os 21 mineiros definidos em `conf/default/genesis.conf`.

Logo que um novo bloco tenha sido cunhado pelo mineiro, a recompensa da mineração será adicionada ao endereço da carteira Coinbase delineado em `conf/example/miner.conf`, que é `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`.

## Interação com os Nós

Nebulas fornece os criadores com API HTTP, API gRPC, e CLI para interagir com os nós em funcionamento. Aqui, vamos partilhar três métodos para enviar uma transação, com o API HTTP ([API Module](https://github.com/nebulasio/wiki/blob/master/rpc.md) | [Admin Module](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)). 


> O ouvinte de HTTP da Nebulas encontra-se definido na configuração do nó. A porta padrão do nosso nó semente é `8685`.

Para começar, verifique o balanço do remetente antes de enviar a transação.

### Verificação do Estado da Conta

Adquira o estado da conta do remetente `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE` com `/v1/user/accountstate` no API Module, usando `curl`.

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

> **Nota**
> O tipo é usado para verificar se esta conta é uma conta de smart contract. `88` representa uma conta do tipo smart contract, e `87` uma conta non-contract.

Como pode ver, o destinatário foi recompensado com alguns tokens por ter minerado novos blocos.

Então vamos verificar o estado da conta do destinatário/recipiente.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"o_seu_endereço"}'


{
    "result": {
        "balance": "0",
        "nonce": "0",
        "type": 87
    }
}
```

A nova conta não tem tokens, como esperado.

### Envio de Transação

Agora vamos enviar uma transação usando três métodos para transferir alguns tokens do remetente para o destinatário!

#### Assina & Envia

Desta forma, pode assinar uma transação num ambiente sem rede (offline), e depois submetê-la a um nó que esteja online. Este é o método mais seguro para submeter uma transação sem expor a sua chava privada à Internet.

Primeiro, assine a transação para obter os dados brutos.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}, "passphrase":"passphrase"}'

{"result":{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}}
```

> **Nota**
> Nonce é um atributo deveras importante numa transação. Foi engendrado para prevenir [ataques de replay](https://en.wikipedia.org/wiki/Replay_attack). Para uma conta especifica, apenas após a transação com nonce N for aceite é que a transação com nonce N+1 será processada. Logo, temos de verificar o ultimo nonce da conta na chain antes de preparar uma nova transação.

Depois, envia os dados brutos para um nó Nebulas online.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/rawtransaction -d '{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}'

{"result":{"txhash":"1b8cc3f977256c4d620b7d72f531bc19f10eb13a05ff24a8a792cd5da53a1277","contract_address":""}}â?Ž
```

#### Envio com Senha

Se confia no nó Nebulas tanto que lhe pode delegar os seus ficheiros keystore, o segundo método é uma boa opção para si.

Primeiro, carregue os seus ficheiros keystore para os directórios da chave no seu nó Nebulas de confiança.

Depois, envie a transação com a sua senha.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":2,"gasPrice":"1000000","gasLimit":"2000000"},"passphrase":"passphrase"}'

{"result":{"txhash":"3cdd38a66c8f399e2f28134e0eb556b292e19d48439f6afde384ca9b60c27010","contract_address":""}}
```

> **Nota**
> Por ter enviado uma transação com nonce 1 da conta `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`, a nova transação com o mesmo `from` (remetente) deve ser incrementada por 1, para 2.

#### Desbloquear & Enviar

Este é o método mais perigoso. Provávelmente não o deve utilizar a não ser que tenha confiança absoluta no nó Nebulas recipiente.

Primeiro, carregue os seus ficheiros keystore para os directórios da chave no seu nó Nebulas de confiança.

Depois desbloqueie as suas contas com a sua senha para uma duração específica no nó. A unidade de duração é nano segundos (300000000000=300s).

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","passphrase":"passphrase","duration":"300000000000"}'

{"result":{"result":true}}
```

Depois de desbloquear a conta, qualquer um será capaz de enviar qualquer transação directamente dentro da duração definida nesse nó sem a sua autorização.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transaction -d '{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":3,"gasPrice":"1000000","gasLimit":"2000000"}'

{"result":{"txhash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","contract_address":""}}â?Ž
```

## Recibo da Transação

Vai obter uma `txhash` nos três métodos após efectuar a transação com sucesso. O valor do `txhash` pode ser usado para verificar o estado da transação.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf"}'

{"result":{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","chainId":100,"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","value":"1000000000000000000","nonce":"3","timestamp":"1524667888","type":"binary","data":null,"gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20000"}}â?Ž
```

O `status` pode tomar os valores de 0, 1, ou 2.

- **0: Falha.** Significa que a transação foi submetida à chain mas a sua execução falhou.
- **1: Sucesso.** Significa que a transação foi submetida à chain e a sua execução foi um sucesso.
- **2: Pendente.** Significa que a transação ainda não foi empacotada num bloco.

### Dupla Verificação

Vamos verificar novamente o balanço do destinatário/recipiente. 

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5"}'

{"result":{"balance":"3000000000000000000","nonce":"0","type":87}}
```

Aqui deve ver o balanço, que é o total de todas as transferências efectuadas com sucesso que executou.

### Próximo passo: Tutorial 3

 [Programa e executa um smart contract com JavaScript](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BPortugues%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md)
