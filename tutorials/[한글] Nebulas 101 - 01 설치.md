# Nebulas 101 - 01 컴파일과 네뷸러스 설치

[유투브 튜토리얼 링크](https://www.youtube.com/watch?v=qtjss2LzSI4&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R)

네뷸러스에 대한 프로젝트 코드(https://nebulas.io/)는 여러가지 버젼으로 배포되고 있고 로컬에서의 동작을 위해 테스트하고 있습니다. 여러분은 프라이빗 체인을 컴파일 하기위해 네뷸러스 소스 코드를 다운로드 할 수 있습니다.

네뷸러스에 대해 더 알고 싶으시면 [Non-Technical White Paper]를 읽어보세요.(https://nebulas.io/docs/NebulasWhitepaper.pdf).

만약 좀 더 기술적으로 알아보고 싶으시다면 [Technical White Paper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)와 네뷸러스 깃허브 코드를 읽어보세요. (https://github.com/nebulasio/go-nebulas)

>네뷸러스는 아직까지는 맥과 리눅스에서만 동작합니다. 곧 윈도우버젼도 배포될 예정입니다.

## Golang 환경

네뷸러스는 Golang을 통해 시행되고 있습니다.

| Components | Version | Description |
|----------|-------------|-------------|
|[Golang](https://golang.org) | >= 1.9.2| The Go Programming Language |

### 맥 OSX

맥에서 golang을 설치하기 위해 [Homebrew](https://brew.sh/)를 추천합니다.

```bash
# install
go를 설치합니다.

# 환경 변수
export GOPATH=/path/to/workspace
```

> 알림:GOPATH는 여러분이 직접 정할 수 있는 로컬 golang 디렉토리입니다. GOPATH를 설정한 후, 당신의 go 프로젝트는 그 GOPATH 디렉토리에 저장되어야 합니다.

### Linux

```bash
# 다운로드
wget https://dl.google.com/go/go1.9.3.linux-amd64.tar.gz

# 추출
tar -C /usr/local -xzf go1.9.3.linux-amd64.tar.gz

# 환경변수
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/path/to/workspace
```

## 네뷸러스 컴파일

### 다운로드

다음 명령어로 소스 코드를 복사합니다.

```bash
# 작업영업 진입
mkdir -p $GOPATH/src/github.com/nebulasio
cd $GOPATH/src/github.com/nebulasio

# 다운로드
git clone https://github.com/nebulasio/go-nebulas.git

# 저장소 진입
cd go-nebulas

# 마스터 브랜치는 가장 안정적입니다
git checkout master
```

### RocksDB 설치하기

* **OS X**:
* Install rocksdb via [Homebrew](https://brew.sh/)
```bash
brew install rocksdb
```

* **Linux - Ubuntu**
* Install Dependencies
```bash
apt-get update
apt-get -y install build-essential libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev liblz4-dev libzstd-dev
```
* Install rocksdb by source code:
```bash
git clone https://github.com/facebook/rocksdb.git
cd rocksdb && make shared_lib && make install-shared
```

* **Linux - Centos**
* Install Dependencies
```bash
yum -y install epel-release && yum -y update
yum -y install gflags-devel snappy-devel zlib-devel bzip2-devel gcc-c++  libstdc++-devel
```
* Install rocksdb by source code:
```bash
git clone https://github.com/facebook/rocksdb.git
cd rocksdb && make shared_lib && make install-shared
```

### Go 디펜던시 설치하기

Go dependencies in Go-Nebulas is managed by [Dep](https://github.com/golang/dep).

| Components | Version | Description |
|----------|-------------|-------------|
[Dep](https://github.com/golang/dep) | >= 0.3.1 | Dep is a dependency management tool for Go. |

#### Dep 설치하기

* **Mac OSX**
* Install Dep via [Homebrew](https://brew.sh/)
```bash
brew install dep
brew upgrade dep
```

* **Linux**
* Install dep
```bash
cd /usr/local/bin/
wget https://github.com/golang/dep/releases/download/v0.3.2/dep-linux-amd64
ln -s dep-linux-amd64 dep
```

#### 디펜던시 다운로드하기

Go-Nebulas에 대한 디펜던시를 다운로드하기 위해 루트 디렉토리를 변경합니다.

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
make dep
```

> `make dep`은 꽤 많은 디펜던시를 다운로드하기 때문에 처음 다운로드 하는 경우라면 시간이 좀 걸릴수 있습니다. 몇몇 디펜던시는 다운로드가 안 될수 있는데 이 경우에는 디펜던시 압축파일을  (http://ory7cn4fx.bkt.clouddn.com/vendor.tar.gz)에서 직접 다운로드하고 네뷸러스 루트 디렉토리로 압축을 풀면 됩니다.
> ```bash
> vendor.tar.gz
> MD5: c2c1ff9311332f90e11fb81b48ca0984
> ```

네뷸러스의 NVM (네뷸러스 가상 머신)은 V8 자바스크립트 엔진으로 구동됩니다.
우리는 맥/리눅스용 v8 디펜던시를 개발하고 있고 다음 명령어로 v8 디펜던시를 설치합니다.

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
make deploy-v8
```

### Neb 구성하기

Golang 디펜던시와  V8 디펜던시를 설치 후에 네뷸러스에 대한 실행 파일을 만들 수 있습니다.

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
make build
```

구성이 끝나고 나면 루트 디렉토리 내에  `neb` 실행파일이 있을 것입니다.
![make build](resources/101-01-make-build.png)

## NEB 시작하기

### 제너시스 블록

새로운 네뷸러스 체인을 시작하기 전에, 먼저 제너시스 블록에 대한 설정을 해야 합니다.

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

genesis.conf의 예시는 `conf/default/genesis.conf`에 있습니다.
