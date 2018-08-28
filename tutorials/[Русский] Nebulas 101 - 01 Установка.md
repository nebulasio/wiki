

# Nebulas 101 - 01 Сборка и установка Nebulas

[Руководство на Youtube](https://www.youtube.com/watch?v=qtjss2LzSI4&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R)

Код проекта [Nebulas](https://nebulas.io/) был выпущен в нескольких версиях и протестирован для локального запуска. Вы можете скачать исходный код Nebulas для поднятия локальной сети.

Чтобы узнать о Nebulas, пожалуйста, прочитайте [Нетехническую документацию](https://nebulas.io/docs/NebulasWhitepaper.pdf) Nebulas.

Чтобы узнать о технологиях, пожалуйста, прочитайте [Техническую документацию](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf) Nebulas, а также просмотрите [исходный код на гите](https://github.com/nebulasio/go-nebulas).

> На данный момент сеть Nebulas можно запустить только на Mac и Linux. Версия для Windows появится в ближайшее время.

## Реализация на Golang

На сегодняшний день, Nebulas реализуется на языке Golang.

| Компоненты | Версия | Описание |
|----------|-------------|-------------|
|[Golang](https://golang.org) | >= 1.9.2| Язык программирования Go |

### Mac OSX

[Homebrew](https://brew.sh/) рекомендуется для установки golang на Mac.

```bash
# Установка
brew install go

# Переменные рабочей директории
export GOPATH=/path/to/workspace
```

> Примечание. GOPATH — это локальная рабочая директория golang, которую можно определить самостоятельно. После настройки GOPATH ваши go-проекты должны быть помещены в директорию GOPATH.

### Linux

```bash
# Загрузка
wget https://dl.google.com/go/go1.9.3.linux-amd64.tar.gz

# Распаковка
tar -C /usr/local -xzf go1.9.3.linux-amd64.tar.gz

# Переменные рабочей директории
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/path/to/workspace
```

## Сборка Nebulas

### Загрузка

Скопируйте исходный код с помощью следующих команд:

```bash
# вход в рабочую директорию
mkdir -p $GOPATH/src/github.com/nebulasio
cd $GOPATH/src/github.com/nebulasio

# Загрузка
git clone https://github.com/nebulasio/go-nebulas.git

# открытие репозитория
cd go-nebulas

# master-ветка самая актуальная и стабильная
git checkout master
```

### Установка RocksDB

* **OS X**:
* Установка rocksdb через [Homebrew](https://brew.sh/)
```bash
brew install rocksdb
```

* **Linux - Ubuntu**
* Установка зависимостей
```bash
apt-get update
apt-get -y install build-essential libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev liblz4-dev libzstd-dev
```
* Установка rocksdb через исходный код:
```bash
git clone https://github.com/facebook/rocksdb.git
cd rocksdb && make shared_lib && make install-shared
```

* **Linux - Centos**
* Установка зависимостей
```bash
yum -y install epel-release && yum -y update
yum -y install gflags-devel snappy-devel zlib-devel bzip2-devel gcc-c++  libstdc++-devel
```
* Установка rocksdb через исходный код:
```bash
git clone https://github.com/facebook/rocksdb.git
cd rocksdb && make shared_lib && make install-shared
```

### Установка зависимостей GO

Зависимости GO в Go-Nebulas определены через [Dep](https://github.com/golang/dep).

| Компоненты | Версия | Описание |
|----------|-------------|-------------|
[Dep](https://github.com/golang/dep) | >= 0.3.1 | Dep - это инструмент управления зависимостями для Go. |

#### Установка Dep

* **Mac OSX**
* Установка Dep через [Homebrew](https://brew.sh/)
```bash
brew install dep
brew upgrade dep
```

* **Linux**
* Установка dep
```bash
cd /usr/local/bin/
wget https://github.com/golang/dep/releases/download/v0.3.2/dep-linux-amd64
ln -s dep-linux-amd64 dep
```

#### Загрузка зависимостей

Перейдите в корневую директорию проекта для загрузки зависимостей Go-Nebulas:

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
make dep
```

> `make dep` загружает множество зависимостей. В первый раз загрузка зависимостей может длиться довольно долго. Некоторые зависимости могут не загружаться. Если загрузить зависимость не удается, попробуйте напрямую загрузить заархивированные файлы зависимостей, созданные dep [vendor.tar.gz](http://ory7cn4fx.bkt.clouddn.com/vendor.tar.gz) и извлечь их в кореневой каталог nebulas.
> ```bash
> vendor.tar.gz
> MD5: c2c1ff9311332f90e11fb81b48ca0984
> ```

Nebulas's NVM (Виртуальная машина Nebulas) зависит от движка V8 JavaScript. Мы создали зависимости v8 для Mac / Linux. Для их установки выполните следующие команды:

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
make deploy-v8
```

### Сборка Neb

После установки golang-зависимостей и зависимостей V8, вы можете собрать исполняемый файл для Nebulas.

Сборка в корневом каталоге проекта:

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
make build
```

После завершения сборки будет создан исполняемый файл `neb`, созданный в корневом каталоге.
![Сборка](resources/101-01-make-build.png)


## Запуск NEB

### Нулевой блок

Перед запуском новой сети Nebulas мы должны определить конфигурацию нулевого блока.

#### Конфигурация нулевого блока

```protobuf
# Текстовый файл neb genesis. Расположен в core / pb / genesis.proto.

meta {
# Идентификатор сети
chain_id: 100
}

consensus {
dpos {
# Назначение первых майнеров
dynasty: [
[ miner address ],
...
]
}
}

# Первичное распределение токенов
token_distribution [
{
address: [ адрес распределения ]
value: [ количество распределяемых токенов ]
},
...
]
```

Пример genesis.conf находится в `conf/default/genesis.conf`.

### Конфигурация

Прежде чем запустить ноду neb, необходимо настроить конфигурацию ноды.

#### Конфигурация Neb Node

```protobuf
# Файл конфигурация Neb. Схема определена в neblet/pb/config.proto:Config.

# Конфигурация сети
network {
# Для первого ноды в новой сети Nebulas «seed» не нужен.
# В противном случае каждой ноде нужны некоторые «seed-ноды», чтобы добавить ее в сеть Nebulas.
# seed: ["/ip4/127.0.0.1/tcp/8680/ipfs/QmP7HDFcYmJL12Ez4ZNVCKjKedfE7f48f1LAkUc3Whz4jP"]

# P2P хост сети. Поддерживает несколько IP-адресов и портов.
listen: ["0.0.0.0:8680"]

# Приватный ключ используется для генерации идентификатора ноды. Если вы не используете приватный ключ, нода будет генерировать новый идентификатор.
# private_key: "conf/network/id_ed25519"
}

# Configuration сети
chain {
# Идентификатор сети
chain_id: 100

# Расположение базы данных
datadir: "data.db"

# Расположение хранилища ключей всех аккаунтов
keydir: "keydir"

# Конфигурация нулевого блока
genesis: "conf/default/genesis.conf"

# Алгоритм подписи
signature_ciphers: ["ECC_SECP256K1"]

# Адрес майнера
miner: "n1SAQy3ix1pZj8MPzNeVqpAmu1nCVqb5w8c"

# Награждение за майнинг будет отправлено на указанный адрес
coinbase: "n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE"

# Кодовая фраза к приватному ключу майнера
passphrase: "passphrase"
}

# Конфигурация API 
rpc {
# Порт GRPC API
rpc_listen: ["127.0.0.1:8684"]

# Порт HTTP API
http_listen: ["127.0.0.1:8685"]

# Открытие модуля
http_module: ["api", "admin"]
}

# Конфигурация логов
app {
# Log level: [debug, info, warn, error, fatal]
log_level: "info"

# Расположение логов
log_file: "logs"

# Открытие краш-логов
enable_crash_report: false
}

# Конфигурация показателей
stats {
# Open node metrics
enable_metrics: false

# Конфигурация Influxdb 
influxdb: {
host: "http://localhost:8086"
db: "nebulas"
user: "admin"
password: "admin"
}
}

```

Множество примеров можно найти здесь `$GOPATH/src/github.com/nebulasio/go-nebulas/conf/`

## Запуск ноды

> Сеть Nebulas, с которой вы работаете в данный момент, является локальной и отличается от основной и тестовой сетей.

Запустите вашу первую ноду Nebulas с помощью следующих команд:

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
./neb -c conf/default/config.conf
```

После запуска, в терминале должно быть видно следующее:
![запуск seed ноды](resources/101-01-seed-node-start.png)

По умолчанию, нода, использующая `conf/default/config.conf`, не будет обрабатывать новые блоки.
Запустите майнинг-ноду Nebulas с помощью следующих команд:

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
./neb -c conf/example/miner.conf
```

После того, как нода запустится, и соединение с seed-нодой прошло успешно, вы можете увидеть данные логи, которые находятся в файле логов `logs/miner/neb.log`:
![запуск ноды](resources/101-01-node-start.png)

> Примечание. Вы можете запустить множество нод локально. Убедитесь, что порты в конфигурациях нод не конфликтуют друг с другом.

### Следующий шаг: Руководство 2

[Отправление транзакций в сети Nebulas](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2002%20Transaction.md)


