# Nebulas 101 - 01 네뷸러스 컴파일과 설치하기

[유튜브 튜토리얼 링크](https://www.youtube.com/watch?v=qtjss2LzSI4&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R)

[네뷸러스](https://nebulas.io/) 프로젝트 코드는 여러가지 버전으로 배포되었고 로컬에서의 실행을 위해 테스트되었습니다. 로컬에서 프라이빗 체인을 컴파일하기 위해 네뷸러스 소스 코드를 설치할 수 있습니다.

네뷸러스에 대해 더 알고 싶으면 [Non-Technical White Paper](https://nebulas.io/docs/NebulasWhitepaper.pdf)를 읽어보세요.

좀 더 기술적으로 알아보고 싶다면 [Technical White Paper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)와 네뷸러스 [깃헙 코드](https://github.com/nebulasio/go-nebulas)를 읽어보세요.

> 네뷸러스는 아직 맥과 리눅스에서만 동작합니다. 곧 윈도우 버전도 배포될 예정입니다.

## Golang 환경

네뷸러스는 Golang으로 구현되었습니다.

| Components | Version | Description |
|----------|-------------|-------------|
|[Golang](https://golang.org) | >= 1.9.2| The Go Programming Language |

### 맥 OSX

맥에서 golang을 설치하려면 [Homebrew](https://brew.sh/)가 필요합니다.

```bash
# 설치
brew install go

# 환경 변수
export GOPATH=/path/to/workspace
```

> 유의사항: GOPATH는 직접 정할 수 있는 로컬 golang 디렉토리입니다. GOPATH를 설정한 후, go 프로젝트는 GOPATH 디렉토리에 있어야 합니다.

### Linux

```bash
# 다운로드
wget https://dl.google.com/go/go1.9.3.linux-amd64.tar.gz

# 압축 해제
tar -C /usr/local -xzf go1.9.3.linux-amd64.tar.gz

# 환경변수
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/path/to/workspace
```

## 네뷸러스 컴파일하기

### 다운로드

다음 명령어로 소스코드를 클론합니다.

```bash
# 작업영역 진입
mkdir -p $GOPATH/src/github.com/nebulasio
cd $GOPATH/src/github.com/nebulasio

# 다운로드
git clone https://github.com/nebulasio/go-nebulas.git

# 저장소 진입
cd go-nebulas

# master 브랜치가 가장 안정적입니다
git checkout master
```

### RocksDB 설치하기

* **OS X**:
* [Homebrew](https://brew.sh/)를 통해 rocksdb를 설치합니다.
```bash
brew install rocksdb
```

* **Linux - Ubuntu**
* 의존성 패키지 설치하기
```bash
apt-get update
apt-get -y install build-essential libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev liblz4-dev libzstd-dev
```
* 소스코드로 rocksdb 설치하기:
```bash
git clone https://github.com/facebook/rocksdb.git
cd rocksdb && make shared_lib && make install-shared
```

* **Linux - Centos**
* 의존성 패키지 설치하기
```bash
yum -y install epel-release && yum -y update
yum -y install gflags-devel snappy-devel zlib-devel bzip2-devel gcc-c++  libstdc++-devel
```
* 소스코드로 rocksdb 설치하기:
```bash
git clone https://github.com/facebook/rocksdb.git
cd rocksdb && make shared_lib && make install-shared
```

### Go 의존성 모듈 설치하기

Go-Nebulas의 Go 의존성 모듈은 [Dep](https://github.com/golang/dep)에 의해 관리됩니다.

| Components | Version | Description |
|----------|-------------|-------------|
[Dep](https://github.com/golang/dep) | >= 0.3.1 | Dep is a dependency management tool for Go. |

#### Dep 설치하기

* **Mac OSX**
* [Homebrew](https://brew.sh/)를 통해 Dep 설치하기
```bash
brew install dep
brew upgrade dep
```

* **Linux**
* dep 설치하기
```bash
cd /usr/local/bin/
wget https://github.com/golang/dep/releases/download/v0.3.2/dep-linux-amd64
ln -s dep-linux-amd64 dep
```

#### 의존성 모듈 다운로드하기

Go-Nebulas의 의존성 모듈을 다운로드하기 위해 프로젝트의 루트 디렉토리로 들어갑니다.

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
make dep
```

> `make dep`은 꽤 많은 의존성 모듈을 다운로드하기 때문에 처음 다운로드하는 경우 시간이 걸릴 수 있습니다. 몇몇 의존성 모듈은 다운로드가 실패할 수 있는데 이 경우에는 압축된 의존성 모듈 파일을 [vendor.tar.gz](http://ory7cn4fx.bkt.clouddn.com/vendor.tar.gz)에서 직접 다운로드하고 네뷸러스 루트 디렉토리에서 압축을 풀면 됩니다.
> ```bash
> vendor.tar.gz
> MD5: c2c1ff9311332f90e11fb81b48ca0984
> ```

네뷸러스의 NVM(네뷸러스 가상 머신)은 V8 자바스크립트 엔진에 의존합니다.
우리는 맥/리눅스용 v8 의존성 모듈을 개발했고 다음 명령어로 v8 의존성 패키지를 설치합니다.

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
make deploy-v8
```

### Neb 빌드하기

Golang 의존성 모듈과 V8 의존성 패키지를 설치한 후에 네뷸러스를 위한 실행 파일을 빌드할 수 있습니다.

프로젝트 루트 디렉토리에서 빌드하세요:

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
make build
```

빌드가 끝나고 나면 루트 디렉토리 내에 `neb` 실행파일이 있을 것입니다.
![make build](resources/101-01-make-build.png)

## NEB 시작하기

### 제너시스 블록

새로운 네뷸러스 블록체인을 시작하기 전에, 먼저 제너시스 블록의 구성을 정의해야 합니다.

#### 제너시스 블록 설정

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

# 첫 토큰의 선배분(Pre-allocation)
token_distribution [
{
address: [ allocation address ]
value: [ amount of allocation tokens ]
},
...
]
```

genesis.conf의 많은 예시는 `conf/default/genesis.conf`에 있습니다.

### 설정

neb 노드를 시작하기 전에, 이 노드의 구성을 정의해야 합니다.

#### Neb 노드 설정

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

# The genesis block configuration
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

많은 예시가 `$GOPATH/src/github.com/nebulasio/go-nebulas/conf/` 안에 있습니다.

## 노드 실행하기

> 여기서 실행한 네뷸러스 블록체인은 로컬에서 돌며, 공식적인 테스트넷과 메인넷의 블록체인과는 별개입니다.

다음 명령어로 첫 네뷸러스 노드를 실행하세요.

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
./neb -c conf/default/config.conf
```

시작한 후, 다음과 같이 터미널에서 보여야 합니다:
![seed node start](resources/101-01-seed-node-start.png)

기본적으로, `conf/default/config.conf`를 사용하는 노드는 새 블럭을 채굴하지 않을 것입니다. 아래 명령어로 첫 네뷸러스 마이닝 노드를 시작하세요.

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
./neb -c conf/example/miner.conf
```

노드가 시작한 후, 시드 노드와의 연결이 성공했다면, `logs/miner/neb.log`에서 다음 로그를 볼 수 있을 것입니다:
![node start](resources/101-01-node-start.png)

> 유의사항: 당신은 많은 노드를 로컬에서 시작할 수 있습니다. 노드 설정에 있는 포트가 서로 충돌이 나지 않도록 하세요.

### 다음 단계: 튜토리얼 2

[네뷸러스에서 트랜잭션 전송하기](https://github.com/nebulasio/wiki/blob/master/tutorials/%5B한글%5D%20Nebulas%20101%20-%2002%20트랜잭션.md.md)

