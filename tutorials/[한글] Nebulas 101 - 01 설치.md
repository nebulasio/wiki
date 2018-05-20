{\rtf1\ansi\ansicpg949\deff0\nouicompat\deflang1033\deflangfe1042{\fonttbl{\f0\fnil\fcharset129 \'b1\'bc\'b8\'b2;}}
{\*\generator Riched20 10.0.17134}\viewkind4\uc1 
\pard\f0\fs22\lang0 # Nebulas 101 - 01 \lang1042\'c4\'c4\'c6\'c4\'c0\'cf\'b0\'fa \'b3\'d7\'ba\'e6\'b7\'af\'bd\'ba \'bc\'b3\'c4\'a1\par
\par
[\'c0\'af\'c5\'f5\'ba\'ea \'c6\'a9\'c5\'e4\'b8\'ae\'be\'f3 \'b8\'b5\'c5\'a9](https://www.youtube.com/watch?v=qtjss2LzSI4&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R)\par
\par
\'b3\'d7\'ba\'e6\'b7\'af\'bd\'ba\'bf\'a1 \'b4\'eb\'c7\'d1 \'c7\'c1\'b7\'ce\'c1\'a7\'c6\'ae \'c4\'da\'b5\'e5(https://nebulas.io/)\'b4\'c2 \'bf\'a9\'b7\'af\'b0\'a1\'c1\'f6 \'b9\'f6\'c1\'af\'c0\'b8\'b7\'ce \'b9\'e8\'c6\'f7\'b5\'c7\'b0\'ed \'c0\'d6\'b0\'ed \'b7\'ce\'c4\'c3\'bf\'a1\'bc\'ad\'c0\'c7 \'b5\'bf\'c0\'db\'c0\'bb \'c0\'a7\'c7\'d8 \'c5\'d7\'bd\'ba\'c6\'ae\'c7\'cf\'b0\'ed \'c0\'d6\'bd\'c0\'b4\'cf\'b4\'d9. \'bf\'a9\'b7\'af\'ba\'d0\'c0\'ba \'c7\'c1\'b6\'f3\'c0\'cc\'ba\'f8 \'c3\'bc\'c0\'ce\'c0\'bb \'c4\'c4\'c6\'c4\'c0\'cf \'c7\'cf\'b1\'e2\'c0\'a7\'c7\'d8 \'b3\'d7\'ba\'e6\'b7\'af\'bd\'ba \'bc\'d2\'bd\'ba \'c4\'da\'b5\'e5\'b8\'a6 \'b4\'d9\'bf\'ee\'b7\'ce\'b5\'e5 \'c7\'d2 \'bc\'f6 \'c0\'d6\'bd\'c0\'b4\'cf\'b4\'d9.\par
\par

\pard\'b3\'d7\'ba\'e6\'b7\'af\'bd\'ba\'bf\'a1 \'b4\'eb\'c7\'d8 \'b4\'f5 \'be\'cb\'b0\'ed \'bd\'cd\'c0\'b8\'bd\'c3\'b8\'e9 [Non-Technical White Paper]\'b8\'a6 \'c0\'d0\'be\'ee\'ba\'b8\'bc\'bc\'bf\'e4.(https://nebulas.io/docs/NebulasWhitepaper.pdf).\par
\par
\'b8\'b8\'be\'e0 \'c1\'bb \'b4\'f5 \'b1\'e2\'bc\'fa\'c0\'fb\'c0\'b8\'b7\'ce \'be\'cb\'be\'c6\'ba\'b8\'b0\'ed \'bd\'cd\'c0\'b8\'bd\'c3\'b4\'d9\'b8\'e9 [Technical White Paper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)\'bf\'cd \'b3\'d7\'ba\'e6\'b7\'af\'bd\'ba \'b1\'ea\'c7\'e3\'ba\'ea \'c4\'da\'b5\'e5\'b8\'a6 \'c0\'d0\'be\'ee\'ba\'b8\'bc\'bc\'bf\'e4. (https://github.com/nebulasio/go-nebulas)\par
\par
>\'b3\'d7\'ba\'e6\'b7\'af\'bd\'ba\'b4\'c2 \'be\'c6\'c1\'f7\'b1\'ee\'c1\'f6\'b4\'c2 \'b8\'c6\'b0\'fa \'b8\'ae\'b4\'aa\'bd\'ba\'bf\'a1\'bc\'ad\'b8\'b8 \'b5\'bf\'c0\'db\'c7\'d5\'b4\'cf\'b4\'d9. \'b0\'f0 \'c0\'a9\'b5\'b5\'bf\'ec\'b9\'f6\'c1\'af\'b5\'b5 \'b9\'e8\'c6\'f7\'b5\'c9 \'bf\'b9\'c1\'a4\'c0\'d4\'b4\'cf\'b4\'d9.\par
\par

\pard\lang0 ## Golang \lang1042\'c8\'af\'b0\'e6\par
\lang0\par
\lang1042\'b3\'d7\'ba\'e6\'b7\'af\'bd\'ba\'b4\'c2 Golang\'c0\'bb \'c5\'eb\'c7\'d8 \'bd\'c3\'c7\'e0\'b5\'c7\'b0\'ed \'c0\'d6\'bd\'c0\'b4\'cf\'b4\'d9.\par
\par

\pard | Components | Version | Description |\par
|----------|-------------|-------------|\par
|[Golang](https://golang.org) | >= 1.9.2| The Go Programming Language |\par
\par
### \'b8\'c6 OSX\par
\par
\'b8\'c6\'bf\'a1\'bc\'ad golang\'c0\'bb \'bc\'b3\'c4\'a1\'c7\'cf\'b1\'e2 \'c0\'a7\'c7\'d8 [Homebrew](https://brew.sh/)\'b8\'a6 \'c3\'df\'c3\'b5\'c7\'d5\'b4\'cf\'b4\'d9.\par
\par
```bash\par
# install\par
go\'b8\'a6 \'bc\'b3\'c4\'a1\'c7\'d5\'b4\'cf\'b4\'d9.\par
\par
# \'c8\'af\'b0\'e6 \'ba\'af\'bc\'f6\par
export GOPATH=/path/to/workspace\par
```\par
\par
> \'be\'cb\'b8\'b2:GOPATH\'b4\'c2 \'bf\'a9\'b7\'af\'ba\'d0\'c0\'cc \'c1\'f7\'c1\'a2 \'c1\'a4\'c7\'d2 \'bc\'f6 \'c0\'d6\'b4\'c2 \'b7\'ce\'c4\'c3 golang \'b5\'f0\'b7\'ba\'c5\'e4\'b8\'ae\'c0\'d4\'b4\'cf\'b4\'d9. GOPATH\'b8\'a6 \'bc\'b3\'c1\'a4\'c7\'d1 \'c8\'c4, \'b4\'e7\'bd\'c5\'c0\'c7 go \'c7\'c1\'b7\'ce\'c1\'a7\'c6\'ae\'b4\'c2 \'b1\'d7 GOPATH \'b5\'f0\'b7\'ba\'c5\'e4\'b8\'ae\'bf\'a1 \'c0\'fa\'c0\'e5\'b5\'c7\'be\'ee\'be\'df \'c7\'d5\'b4\'cf\'b4\'d9.\par
\par
### Linux\par
\par
```bash\par
# \'b4\'d9\'bf\'ee\'b7\'ce\'b5\'e5\par
wget https://dl.google.com/go/go1.9.3.linux-amd64.tar.gz\par
\par
# \'c3\'df\'c3\'e2\par
tar -C /usr/local -xzf go1.9.3.linux-amd64.tar.gz\par
\par
# \'c8\'af\'b0\'e6\'ba\'af\'bc\'f6\par
export PATH=$PATH:/usr/local/go/bin\par
export GOPATH=/path/to/workspace\par
```\par
\par
## \'b3\'d7\'ba\'e6\'b7\'af\'bd\'ba \'c4\'c4\'c6\'c4\'c0\'cf\par
\par
### \'b4\'d9\'bf\'ee\'b7\'ce\'b5\'e5\par
\par
\'b4\'d9\'c0\'bd \'b8\'ed\'b7\'c9\'be\'ee\'b7\'ce \'bc\'d2\'bd\'ba \'c4\'da\'b5\'e5\'b8\'a6 \'ba\'b9\'bb\'e7\'c7\'d5\'b4\'cf\'b4\'d9.\par
\par
```bash\par
# \'c0\'db\'be\'f7\'bf\'b5\'be\'f7 \'c1\'f8\'c0\'d4\par
mkdir -p $GOPATH/src/github.com/nebulasio\par
cd $GOPATH/src/github.com/nebulasio\par
\par
# \'b4\'d9\'bf\'ee\'b7\'ce\'b5\'e5\par
git clone https://github.com/nebulasio/go-nebulas.git\par
\par
# \'c0\'fa\'c0\'e5\'bc\'d2 \'c1\'f8\'c0\'d4\par
cd go-nebulas\par
\par
# \'b8\'b6\'bd\'ba\'c5\'cd \'ba\'ea\'b7\'a3\'c4\'a1\'b4\'c2 \'b0\'a1\'c0\'e5 \'be\'c8\'c1\'a4\'c0\'fb\'c0\'d4\'b4\'cf\'b4\'d9\par
git checkout master\par
```\par
\par
### RocksDB \'bc\'b3\'c4\'a1\'c7\'cf\'b1\'e2\par
\par
* **OS X**:\par
* Install rocksdb via [Homebrew](https://brew.sh/)\par
```bash\par
brew install rocksdb\par
```\par
\par
* **Linux - Ubuntu**\par
* Install Dependencies\par
```bash\par
apt-get update\par
apt-get -y install build-essential libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev liblz4-dev libzstd-dev\par
```\par
* Install rocksdb by source code:\par
```bash\par
git clone https://github.com/facebook/rocksdb.git\par
cd rocksdb && make shared_lib && make install-shared\par
```\par
\par
* **Linux - Centos**\par
* Install Dependencies\par
```bash\par
yum -y install epel-release && yum -y update\par
yum -y install gflags-devel snappy-devel zlib-devel bzip2-devel gcc-c++  libstdc++-devel\par
```\par
* Install rocksdb by source code:\par
```bash\par
git clone https://github.com/facebook/rocksdb.git\par
cd rocksdb && make shared_lib && make install-shared\par
```\par
\par
### Go \'b5\'f0\'c6\'e6\'b4\'f8\'bd\'c3 \'bc\'b3\'c4\'a1\'c7\'cf\'b1\'e2\par
\par
Go dependencies in Go-Nebulas is managed by [Dep](https://github.com/golang/dep).\par
\par
| Components | Version | Description |\par
|----------|-------------|-------------|\par
[Dep](https://github.com/golang/dep) | >= 0.3.1 | Dep is a dependency management tool for Go. |\par
\par
#### Dep \'bc\'b3\'c4\'a1\'c7\'cf\'b1\'e2\par
\par
* **Mac OSX**\par
* Install Dep via [Homebrew](https://brew.sh/)\par
```bash\par
brew install dep\par
brew upgrade dep\par
```\par
\par
* **Linux**\par
* Install dep\par
```bash\par
cd /usr/local/bin/\par
wget https://github.com/golang/dep/releases/download/v0.3.2/dep-linux-amd64\par
ln -s dep-linux-amd64 dep\par
```\par
\par
#### \'b5\'f0\'c6\'e6\'b4\'f8\'bd\'c3 \'b4\'d9\'bf\'ee\'b7\'ce\'b5\'e5\'c7\'cf\'b1\'e2\par
\par
Go-Nebulas\'bf\'a1 \'b4\'eb\'c7\'d1 \'b5\'f0\'c6\'e6\'b4\'f8\'bd\'c3\'b8\'a6 \'b4\'d9\'bf\'ee\'b7\'ce\'b5\'e5\'c7\'cf\'b1\'e2 \'c0\'a7\'c7\'d8 \'b7\'e7\'c6\'ae \'b5\'f0\'b7\'ba\'c5\'e4\'b8\'ae\'b8\'a6 \'ba\'af\'b0\'e6\'c7\'d5\'b4\'cf\'b4\'d9.\par
\par
```bash\par
cd $GOPATH/src/github.com/nebulasio/go-nebulas\par
make dep\par
```\par
\par
> `make dep`\'c0\'ba \'b2\'cf \'b8\'b9\'c0\'ba \'b5\'f0\'c6\'e6\'b4\'f8\'bd\'c3\'b8\'a6 \'b4\'d9\'bf\'ee\'b7\'ce\'b5\'e5\'c7\'cf\'b1\'e2 \'b6\'a7\'b9\'ae\'bf\'a1 \'c3\'b3\'c0\'bd \'b4\'d9\'bf\'ee\'b7\'ce\'b5\'e5 \'c7\'cf\'b4\'c2 \'b0\'e6\'bf\'ec\'b6\'f3\'b8\'e9 \'bd\'c3\'b0\'a3\'c0\'cc \'c1\'bb \'b0\'c9\'b8\'b1\'bc\'f6 \'c0\'d6\'bd\'c0\'b4\'cf\'b4\'d9. \'b8\'ee\'b8\'ee \'b5\'f0\'c6\'e6\'b4\'f8\'bd\'c3\'b4\'c2 \'b4\'d9\'bf\'ee\'b7\'ce\'b5\'e5\'b0\'a1 \'be\'c8 \'b5\'c9\'bc\'f6 \'c0\'d6\'b4\'c2\'b5\'a5 \'c0\'cc \'b0\'e6\'bf\'ec\'bf\'a1\'b4\'c2 \'b5\'f0\'c6\'e6\'b4\'f8\'bd\'c3 \'be\'d0\'c3\'e0\'c6\'c4\'c0\'cf\'c0\'bb  (http://ory7cn4fx.bkt.clouddn.com/vendor.tar.gz)\'bf\'a1\'bc\'ad \'c1\'f7\'c1\'a2 \'b4\'d9\'bf\'ee\'b7\'ce\'b5\'e5\'c7\'cf\'b0\'ed \'b3\'d7\'ba\'e6\'b7\'af\'bd\'ba \'b7\'e7\'c6\'ae \'b5\'f0\'b7\'ba\'c5\'e4\'b8\'ae\'b7\'ce \'be\'d0\'c3\'e0\'c0\'bb \'c7\'ae\'b8\'e9 \'b5\'cb\'b4\'cf\'b4\'d9.\par
> ```bash\par
> vendor.tar.gz\par
> MD5: c2c1ff9311332f90e11fb81b48ca0984\par
> ```\par
\par
\'b3\'d7\'ba\'e6\'b7\'af\'bd\'ba\'c0\'c7 NVM (\'b3\'d7\'ba\'e6\'b7\'af\'bd\'ba \'b0\'a1\'bb\'f3 \'b8\'d3\'bd\'c5)\'c0\'ba V8 \'c0\'da\'b9\'d9\'bd\'ba\'c5\'a9\'b8\'b3\'c6\'ae \'bf\'a3\'c1\'f8\'c0\'b8\'b7\'ce \'b1\'b8\'b5\'bf\'b5\'cb\'b4\'cf\'b4\'d9.\par
\'bf\'ec\'b8\'ae\'b4\'c2 \'b8\'c6/\'b8\'ae\'b4\'aa\'bd\'ba\'bf\'eb v8 \'b5\'f0\'c6\'e6\'b4\'f8\'bd\'c3\'b8\'a6 \'b0\'b3\'b9\'df\'c7\'cf\'b0\'ed \'c0\'d6\'b0\'ed \'b4\'d9\'c0\'bd \'b8\'ed\'b7\'c9\'be\'ee\'b7\'ce v8 \'b5\'f0\'c6\'e6\'b4\'f8\'bd\'c3\'b8\'a6 \'bc\'b3\'c4\'a1\'c7\'d5\'b4\'cf\'b4\'d9.\par
\par
```bash\par
cd $GOPATH/src/github.com/nebulasio/go-nebulas\par
make deploy-v8\par
```\par
\par
### Neb \'b1\'b8\'bc\'ba\'c7\'cf\'b1\'e2\par
\par
Golang \'b5\'f0\'c6\'e6\'b4\'f8\'bd\'c3\'bf\'cd  V8 \'b5\'f0\'c6\'e6\'b4\'f8\'bd\'c3\'b8\'a6 \'bc\'b3\'c4\'a1 \'c8\'c4\'bf\'a1 \'b3\'d7\'ba\'e6\'b7\'af\'bd\'ba\'bf\'a1 \'b4\'eb\'c7\'d1 \'bd\'c7\'c7\'e0 \'c6\'c4\'c0\'cf\'c0\'bb \'b8\'b8\'b5\'e9 \'bc\'f6 \'c0\'d6\'bd\'c0\'b4\'cf\'b4\'d9.\par
\par
```bash\par
cd $GOPATH/src/github.com/nebulasio/go-nebulas\par
make build\par
```\par
\par
\'b1\'b8\'bc\'ba\'c0\'cc \'b3\'a1\'b3\'aa\'b0\'ed \'b3\'aa\'b8\'e9 \'b7\'e7\'c6\'ae \'b5\'f0\'b7\'ba\'c5\'e4\'b8\'ae \'b3\'bb\'bf\'a1  `neb` \'bd\'c7\'c7\'e0\'c6\'c4\'c0\'cf\'c0\'cc \'c0\'d6\'c0\'bb \'b0\'cd\'c0\'d4\'b4\'cf\'b4\'d9.\par
![make build](resources/101-01-make-build.png)\par
\par
## NEB \'bd\'c3\'c0\'db\'c7\'cf\'b1\'e2\par
\par
### \'c1\'a6\'b3\'ca\'bd\'c3\'bd\'ba \'ba\'ed\'b7\'cf\par
\par
\'bb\'f5\'b7\'ce\'bf\'ee \'b3\'d7\'ba\'e6\'b7\'af\'bd\'ba \'c3\'bc\'c0\'ce\'c0\'bb \'bd\'c3\'c0\'db\'c7\'cf\'b1\'e2 \'c0\'fc\'bf\'a1, \'b8\'d5\'c0\'fa \'c1\'a6\'b3\'ca\'bd\'c3\'bd\'ba \'ba\'ed\'b7\'cf\'bf\'a1 \'b4\'eb\'c7\'d1 \'bc\'b3\'c1\'a4\'c0\'bb \'c7\'d8\'be\'df \'c7\'d5\'b4\'cf\'b4\'d9.\par
\par
#### \'c1\'a6\'b3\'ca\'bd\'c3\'bd\'ba \'ba\'ed\'b7\'cf \'bc\'b3\'c1\'a4\par
\par
```protobuf\par
# Neb genesis text file. Scheme is defined in core/pb/genesis.proto.\par
\par
meta \{\par
# Chain identity\par
chain_id: 100\par
\}\par
\par
consensus \{\par
dpos \{\par
# Initial dynasty, including all initial miners\par
dynasty: [\par
[ miner address ],\par
...\par
]\par
\}\par
\}\par
\par
# \'c3\'b9 \'c5\'e4\'c5\'ab\'c0\'c7 \'bc\'b1\'b9\'e8\'ba\'d0(Pre-allocation)\par
token_distribution [\par
\{\par
address: [ allocation address ]\par
value: [ amount of allocation tokens ]\par
\},\par
...\par
]\par
```\par
\par
genesis.conf\'c0\'c7 \'bf\'b9\'bd\'c3\'b4\'c2 `conf/default/genesis.conf`\'bf\'a1 \'c0\'d6\'bd\'c0\'b4\'cf\'b4\'d9.\par
\par
\par
\par
\par
\par
\par
\par
\par
\par
\par
\par
\par
\par
\par
\par
\par
\par

\pard\lang0\par
}
 