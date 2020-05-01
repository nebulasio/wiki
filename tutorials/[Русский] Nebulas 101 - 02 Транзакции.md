# Nebulas 101 - 02 Отправление транзакций в сети Nebulas

[Руководство на Youtube](https://www.youtube.com/watch?v=-44tVVR6ETo&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R&index=1)

> Для полного понимания данного руководства советуем ознакомиться с [Руководством по установке](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2001%20Installation.md).

В сети Nebulas существует три метода отправления транзакций:

1. Подписать и отправить
2. Отправить с помощью парольной фразой
3. Разблокировать кошелек и отправить

Ниже будут подробно описаны все методы отправления транзакций, а также способы проверки их успешности.

## Подготовка аккаунтов

В сети Nebulas каждый адрес представляет собой уникальный аккаунт.

Подготовьте два аккаунта: аккаунт для отправления токенов (отправляющий адрес, называемый «from») и аккаунт для получения токенов (принимающий адрес, называемый «to»).

### Отправитель

В данном примере мы будем использовать аккаунт coinbase, расположенный в `conf/example/miner.conf`, которому соответствует адрес `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`. Так этот аккаунт является еще и майнером, он будет получать токены за подтверждение транзакций в сети. Позже мы отправим эти токены на новый аккаунт.

### Получатель

Создайте новый кошелек, чтобы получить токены.

```bash
$ ./neb account new
Your new account is locked with a passphrase. Please give a passphrase. Do not forget this passphrase.
Passphrase:
Repeat passphrase:
Address: n1SQe5d1NKHYFMKtJ5sNHPsSPVavGzW71Wy
```

> Необходимо помнить, что после запуска данной команды, адрес вашего нового кошелька не будет таким, как старый `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`. Пожалуйста, используйте ваш сгенерированный новый адрес в качестве получателя.

Файл с ключами от нового кошелька расположен здесь `$GOPATH/src/github.com/nebulasio/go-nebulas/keydir/`

## Запуск ноды

### Запуск первичной ноды

Во-первых, запустите первичную ноду в вашей локальной сети.

```bash
./neb -c conf/default/config.conf
```

### Запуск майнинг-ноды

Во-вторых, запустите майнинг-ноду, подключенную к первичной ноде. Эта нода будет генерировать новые блоки в вашей локальной сети.

```bash
./neb -c conf/example/miner.conf
``` 

> **С какой скоростью будут появляться новые блоки?**
> 
> В сети Nebulas, DPoS был выбран в качестве временного алгоритма консенсуса перед тем, как Proof-of-Devotion(PoD, описанным в [Технической документации](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)) будет готов. В этом алгоритме, все майнеры будут генерировать блок в течение 15 секунд.
> 
> На скгодняшний день нужно ждать 315(=15*21) секунд чтобы появился новый блок, т.к. работает только 1 майнер из заявленных 21 `conf/default/genesis.conf`.

Как только майнер сгенерирует блок, вознаграждение будет отправлено на адрес `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`, указанном в `conf/example/miner.conf`.

## Взаимодействие с нодами

Nebulas позволяет разработчикам использовать HTTP API, gRPC API и CLI для взаимодействия с работающими нодами. Здесь мы расскажем как отправлять транзакции тремя способами, используя HTTP API ([API Модуль](https://github.com/nebulasio/wiki/blob/master/rpc.md) | [Admin Модуль](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)). 

> Nebulas HTTP Listener определен в конфигурации ноды. Стандартный порт первичной ноды: `8685`.

Вначале проверьте баланс отправителя перед отправкой транзакции.

### Проверка баланса аккаунта

Получить состояние аккаунта отправителя `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE` с помощью `/v1/user/accountstate` в API модуле, используя `curl`.

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

> **Примечание**
> Свойство "Type" используется для проверки того, является ли адрес смартконтрактом. `88` означает адрес смартконтракта, а `87` обычный пользовательский адрес.

Как мы видим, получатель был награжден несколькими токенами за генерацию новых блоков.

Затем давайте проверим состояние учетной записи получателя.

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

Как и ожидалось, на балансе нового аккаунта токенов нет.

### Отправление транзакций

Теперь давайте отправим транзакцию тремя способами для перевода нескольких токенов от отправителя получателю!

#### Подписать и отправить

С помощью этого метода мы можем подписать транзакцию в оффлайн режиме, а затем отправить ее на другую онлайн ноду. Это самый безопасный способ для отправления транзакций, т.к. ваш приватный ключ не падает в интернет.

Сначала подпишите транзакцию, чтобы получить необработанные данные.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}, "passphrase":"passphrase"}'

{"result":{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}}
```

> **Примечание**
> Nonce - очень важный атрибут транзакции. Он создан чтобы предотвратить [Атаку повторного воспроизведения](https://ru.wikipedia.org/wiki/%D0%90%D1%82%D0%B0%D0%BA%D0%B0_%D0%BF%D0%BE%D0%B2%D1%82%D0%BE%D1%80%D0%BD%D0%BE%D0%B3%D0%BE_%D0%B2%D0%BE%D1%81%D0%BF%D1%80%D0%BE%D0%B8%D0%B7%D0%B2%D0%B5%D0%B4%D0%B5%D0%BD%D0%B8%D1%8F). Только после того, как как была принята транзакция с nonce N, будет обработана транзакция с nonce N + 1. Таким образом, перед отправлением новой транзакции, проверяется актуальный баланс аккаунта в сети.

Затем отправьте необработанные данные онлайн-ноде Nebulas.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/rawtransaction -d '{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}'

{"result":{"txhash":"1b8cc3f977256c4d620b7d72f531bc19f10eb13a05ff24a8a792cd5da53a1277","contract_address":""}}
```

#### Отправление с помощью парольной фразой

Если вы доверяете ноде Nebulas настолько, что можете передать ей файлы хранилища ключей, этот метод вам подойдет.

Сначала загрузите файлы хранилища ключей в папки keydir в ноду Nebulas.

Затем отправьте транзакцию с помощью вашей кодовой фразы.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":2,"gasPrice":"1000000","gasLimit":"2000000"},"passphrase":"passphrase"}'

{"result":{"txhash":"3cdd38a66c8f399e2f28134e0eb556b292e19d48439f6afde384ca9b60c27010","contract_address":""}}
```

> **Примечание**
> Т.к. мы отправили транзакцию с nonce равным 1 с аккаунта `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`, nonce новой транзакции от того же `from` должен увеличиться на 1, а значит стать 2.

#### Разблокировать кошелек и отправить

Это самый опасный метод. Вероятно, вы не должны использовать его, если не имеете полной уверенности в безопастности получения данных нодой Nebulas.

Сначала загрузите файлы хранилища ключей в папки keydir в ноду Nebulas.

Затем разблокируйте свой аккаунт с помощью вашей кодовой фразы с заданной продолжительности хранения в ноде.
Единицей продолжительности является наносекунда (300000000000=300сек).

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","passphrase":"passphrase","duration":"300000000000"}'

{"result":{"result":true}}
```

После разблокировки вашего аккаунта, любой, кто имеет к нему доступ, сможет отправить транзакцию в течение указанного времени даже без вашего разрешения.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transaction -d '{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":3,"gasPrice":"1000000","gasLimit":"2000000"}'

{"result":{"txhash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","contract_address":""}}
```

## Получение транзакции

После успешного отправления транзакции любым из трех вышеперечисленных методов мы получим `txhash`. Значение `txhash` может использоваться для запроса статуса транзакции.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf"}'

{"result":{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","chainId":100,"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","value":"1000000000000000000","nonce":"3","timestamp":"1524667888","type":"binary","data":null,"gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20000"}}
```

Значение поля `status` может быть равным 0, 1 или 2.

- **0: Неудачная.** Это означает, что транзакция была отправлена по сети, но ее выполнение не удалось.
- **1: Успешная.** Это означает, что транзакция была отправлена по сети и ее исполнение выполнено успешно.
- **2: В ожидании.** Это означает, что транзакция пока еще не была добавлена в блок.

### Перепроверка

Давайте перепроверим баланс получателя.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5"}'

{"result":{"balance":"3000000000000000000","nonce":"0","type":87}}
```

Здесь вы должны увидеть баланс, который является результатом всех успешных транзакций, которые вы отправляли.

### Следующий шаг: Руководство 3

 [Написание и запуск смартконтрактов на JavaScript](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md)
