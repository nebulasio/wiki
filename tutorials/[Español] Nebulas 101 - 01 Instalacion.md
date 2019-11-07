

# Nebulas 101 - 01 Compilar y Instalar Nebulas

Este proyecto código [Nebulas](https://nebulas.io/) ha sido lanzado en algunos versiones y probado para ejecutar localmente. Se puede descargar el código fuente de Nebulas para compilar la cadena privada localemente.

Para aprender sobre Nebulas, por favor lea el libro blanco. [Non-Technical White Paper](https://nebulas.io/docs/NebulasWhitepaper.pdf).

Para aprender sobre la tecnologia, por favor lea el documento de Nebulas [Technical White Paper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf) y el código de Nebulas [github code](https://github.com/nebulasio/go-nebulas).

> Nebulas puede correr solamente en Mac y Linux en este momento. El version de Windows llegará pronto.

## Golang Environment

Nebulas está implementado en Golango ahora.

| Componentes | Version | Descripción |
|----------|-------------|-------------|
|[Golang](https://golang.org) | >= 1.12| Lenguaje de programación el Go |

### Mac OSX

[Homebrew](https://brew.sh/) está sugerida para instalar golang en Mac.

```bash
# install
brew install go

# environment variables
export GOPATH=/directorio/de/trabajo
```

> Aviso:GOPATH es un directorio de golang que se puede ser cambiado por su mismo. Después GOPATH está configurado, el go proyectos tuyos necesitan ser colocados dentro GOPATH directorio.

### Linux

```bash
# descargar
wget https://dl.google.com/go/go1.12.linux-amd64.tar.gz

# extraer
tar -C /usr/local -xzf go1.12.linux-amd64.tar.gz

# environment variables
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/directorio/de/trabajo
```

## Compilar Nebulas

### Descargar

Clona el código fuente con los siguientes comandos.

```bash
# entra espaciotrabajo
cd workspace

# descargar
git clone https://github.com/nebulasio/go-nebulas.git

# entra repositorio
cd go-nebulas

# master branch esta más etable.
git checkout master
```

### Instalar RocksDB

* **OS X**:
* Instalar rocksdb via [Homebrew](https://brew.sh/)
```bash
brew install rocksdb
```

* **Linux - Ubuntu**
* Instalar Dependencias
```bash
apt-get update
apt-get -y install build-essential libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev liblz4-dev libzstd-dev
```
* Instalar rocksdb por código fuente:
```bash
git clone https://github.com/facebook/rocksdb.git
cd rocksdb && make shared_lib && make install-shared
```

* **Linux - Centos**
* Instalar Dependencias
```bash
yum -y install epel-release && yum -y update
yum -y install gflags-devel snappy-devel zlib-devel bzip2-devel gcc-c++  libstdc++-devel
```
* Instalar rocksdb por código fuente:
```bash
git clone https://github.com/facebook/rocksdb.git
cd rocksdb && make shared_lib && make install-shared
```

Nebulas de NVM (Nebulas Virtual Machine) depende del V8 JavaScript máquina. Podemos construir las dependencias de V8 para Mac/Linux con los siguientes comandos:


```bash
cd workspace/go-nebulas
source setup.sh
```

### Construir Neb
Ahora tu puedes construir el ejecutable para Nebulas despues dependencies de golang y V8 paquetes estan instalados.

Construir bajo el directorio raíz del proyecto:

```bash
cd workspace/go-nebulas
make build
```

Despues el construcción está terminado, habrá uno archivo ejecutable `neb` generado bajo el directorio raíz.
![make build](resources/101-01-make-build.png)


## Comienza NEB

### Genesis Bloque

Antes lanzando una nueva cadena Nebulas, hay que definir el configuracion de genesis bloque.

#### Genesis Bloque Configuracion

```protobuf
# Neb genesis text file. Scheme is defined in core/pb/genesis.proto.

meta {
# Chain identity
chain_id: 100
}

consensus {
dpos {
# Initial dynasty, including all initial miners
dynasty: [
[ miner address ],
...
]
}
}

# Pre-allocation of initial tokens
token_distribution [
{
address: [ allocation address ]
value: [ amount of allocation tokens ]
},
...
]
```

Un ejemplo genesis.conf está ubicado en `conf/default/genesis.conf`.

### Configuracion

Antes lanzando un nodo, hay que definir el configuracion de este nodo.

#### Nodo Neb Configuración

```protobuf
# Neb configuration text file. Scheme is defined in neblet/pb/config.proto:Config.

# Network Configuration
network {
# For the first node in a new Nebulas chain, `seed` is not need.
# Otherwise, every node need some seed nodes to introduce it into the Nebulas chain.
# seed: ["/ip4/127.0.0.1/tcp/8680/ipfs/QmP7HDFcYmJL12Ez4ZNVCKjKedfE7f48f1LAkUc3Whz4jP"]

# P2p network service host. support mutiple ip and ports.
listen: ["0.0.0.0:8680"]

# The private key is used to generate a node ID. If you don't use the private key, the node will generate a new node ID.
# private_key: "conf/network/id_ed25519"
}

# Chain Configuration
chain {
# Network chain ID
chain_id: 100

# Database storage location
datadir: "data.db"

# Accounts' keystore files location
keydir: "keydir"

# The genesis bloque configuration
genesis: "conf/default/genesis.conf"

# Signature algorithm
signature_ciphers: ["ECC_SECP256K1"]

# Miner address
miner: "n1SAQy3ix1pZj8MPzNeVqpAmu1nCVqb5w8c"

# Coinbase address, all mining reward received by the above miner will be send to this address
coinbase: "n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE"

# The passphrase to miner's keystore file
passphrase: "passphrase"
}

# API Configuration
rpc {
# GRPC API port
rpc_listen: ["127.0.0.1:8684"]

# HTTP API port
http_listen: ["127.0.0.1:8685"]

# The module opened
http_module: ["api", "admin"]
}

# Log Configuration
app {
# Log level: [debug, info, warn, error, fatal]
log_level: "info"

# Log location
log_file: "logs"

# Open crash log
enable_crash_report: false
}

# Metrics Configuration
stats {
# Open node metrics
enable_metrics: false

# Influxdb configuration
influxdb: {
host: "http://localhost:8086"
db: "nebulas"
user: "admin"
password: "admin"
}
}

```
Muchos ejemplos se encuentra aqui `$GOPATH/src/github.com/nebulasio/go-nebulas/conf/`

## Lanzando Nodos

> La Nebulas cadena que esta funcionando en este momento es privado y diferente entre testnet oficíal y el mainnet.

Lanca su primero nodo Nebulas con los siguientes comandos.

```bash
cd workspace/go-nebulas
./neb -c conf/default/config.conf
```

Después de comenzar, el siguiente debe ser visible en el terminal.
![seed node start](resources/101-01-seed-node-start.png)

Por defecto, el nodo usando `conf/default/config.conf` no minará nuevos bloques.
Comienca su primera mineriá con un otro comando.

```bash
cd workspace/go-nebulas
./neb -c conf/example/miner.conf
```

Después de comenzar el nodo, si los conexiones con nodo semilla tiene éxito, tu puedes ver el siguiente registro que se encuentra en el archivo `logs/miner/neb.log`:
![node start](resources/101-01-node-start.png)

> Aviso: Puedes lanzar muchos nodes localmente. Por favor verificar los puertos en su configuracion de nodo para que no entren en conflicto el un con el otro.

### Próximo Paso: Tutorial 2

[Enviando Transacciones en Nebulas](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEspa%C3%B1ol%5D%20Nebulas%20101%20-%2002%20Transacciones.md)
