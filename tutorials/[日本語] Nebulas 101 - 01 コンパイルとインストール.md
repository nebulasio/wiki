

# Nebulas 101 - 01 Nebulasのコンパイルとインストール

[YouTube チュートリアル](https://www.youtube.com/watch?v=qtjss2LzSI4&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R)

[Nebulas](https://nebulas.io/)のプロジェクトコードは多数のバーションにリリースされて、ローカルに運行されるようにテストした。Nebulasソースコードをダウンロードし、ローカルでプライベートチェンをコンパイルすることができる。

Nebulasを学びたい人は、Nebulas [Non-Technical White Paper](https://nebulas.io/docs/NebulasWhitepaper.pdf)を読んでください。

その技術を学びたい人は、Nebulas [Technical White Paper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)とNebulas [github code](https://github.com/nebulasio/go-nebulas)を読んでください。

> 今MacとLinuxでしかNebulasを運行および起動できない。Windowsバージョンはもうすぐに。

## Golangの環境構築

現時点NebulasはGolangで実装されている。

| Components | Version | Description |
|----------|-------------|-------------|
|[Golang](https://golang.org) | >= 1.9.2| The Go Programming Language |

### Mac OSX

[Homebrew](https://brew.sh/)はMacでgolangをインストールにオススメです。

```bash
# インストール
brew install go

# 環境変数
export GOPATH=/path/to/workspace
```

> 注意:GOPATHはローカルgolang作業ディレクトリで、自分でカスタマイズできる。GOPATH配置できたあと、goプロジェクトはGOPATHディレクトリに置くべきです。

### Linux

```bash
# ダウンロード
wget https://dl.google.com/go/go1.9.3.linux-amd64.tar.gz

# エキス
tar -C /usr/local -xzf go1.9.3.linux-amd64.tar.gz

# 環境変数
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/path/to/workspace
```

## コンパイルNebulas

### ダウンロード

以下のコマンドでソースコードをクローンする。

```bash
# ワークスペースに入る
mkdir -p $GOPATH/src/github.com/nebulasio
cd $GOPATH/src/github.com/nebulasio

# ダウンロード
git clone https://github.com/nebulasio/go-nebulas.git

# リポジトリに入る
cd go-nebulas

# もっとも安定的なmasterブランチに切り替える
git checkout master
```

### RocksDBをインストール

* **OS X**:
* [Homebrew](https://brew.sh/)を通じてrocksdbをインストールする
```bash
brew install rocksdb
```

* **Linux - Ubuntu**
* 依存をインストールする
```bash
apt-get update
apt-get -y install build-essential libgflags-dev libsnappy-dev zlib1g-dev libbz2-dev liblz4-dev libzstd-dev
```
* ソースコードでrocksdbをインストールする:
```bash
git clone https://github.com/facebook/rocksdb.git
cd rocksdb && make shared_lib && make install-shared
```

* **Linux - Centos**
* 依存をインストールする
```bash
yum -y install epel-release && yum -y update
yum -y install gflags-devel snappy-devel zlib-devel bzip2-devel gcc-c++  libstdc++-devel
```
* ソースコードでrocksdbをインストールする:
```bash
git clone https://github.com/facebook/rocksdb.git
cd rocksdb && make shared_lib && make install-shared
```

### Goの依存ライブラリをインストール

Go-NebulasのGo依存は[Dep](https://github.com/golang/dep)に管理される。

| Components | Version | Description |
|----------|-------------|-------------|
[Dep](https://github.com/golang/dep) | >= 0.3.1 | Dep is a dependency management tool for Go. |

#### Depをインストール

* **Mac OSX**
* [Homebrew](https://brew.sh/)を通じてDepをインストールする
```bash
brew install dep
brew upgrade dep
```

* **Linux**
* depをインストールする
```bash
cd /usr/local/bin/
wget https://github.com/golang/dep/releases/download/v0.3.2/dep-linux-amd64
ln -s dep-linux-amd64 dep
```

#### 依存をダウンロード

プロジェクトのルートディレクトリに切り替えて、Go-Nebulasの依存をダウンロードする:

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
make dep
```

> `make dep` がたくさんの依存をダウンロードする。はじめの時は、長い時間をかかるかもしれない。いくつかの依頼をダウンロード失敗の可能性もある。もしダウンロードできないなら、直接にdep [vendor.tar.gz](https://s3-us-west-1.amazonaws.com/develop-center/setup/vendor.tar.gz) が生成するジップした依存ファイルをダウンロードして、nebulasのルートディレクトリに抽出することができる。
> ```bash
> vendor.tar.gz
> MD5: a8ff50c9c01c67e37300a062edf7949d
> ```

NebulasのNVM (Nebulas Virtual Machine) はV8のJavaScriptエンジンを依頼する。v8の依存はもうMac/Linuxで構築されている。以下のコマンドを運行してインストールする。

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
make deploy-v8
```

### Nebを構築

golangの依存とV8の依存パッケージがインストールできたら、Nebulasの執行を立てることができる。

プロジェクトルートディレクトリに作る:

```bash
cd  $GOPATH/src/github.com/nebulasio/go-nebulas
make build
```

いったん工事が終わったら、一つの執行できるファイル`neb`がルートディレクトリに生成する。
![make build](resources/101-01-make-build.png)


## NEBを起動

### ジェネシスブロック

新しいNebulasチェンを起動する前に、ジェネシスブロックの配置を定義すること。

#### ジェネシスブロックの配置

```protobuf
# Nebジェネシステキストファイル。スキームはcore/pb/genesis.protoに定義されている。

meta {
# Chain identity
chain_id: 100
}

consensus {
dpos {
# 最初のネットワークが、全ての初期マイナーを含めっている
dynasty: [
[ miner address ],
...
]
}
}

# 最初トークンの事前配分
token_distribution [
{
address: [ allocation address ]
value: [ amount of allocation tokens ]
},
...
]
```

一つの例として genesis.conf は`conf/default/genesis.conf`に存在する。

### コンフィグレーション

nebノードを動き始める前に、このノードの配置を定義すること。

#### Nebノードの配置

```protobuf
# Nebコンフィグレーションテキストファイル。 スキームはneblet/pb/config.proto:Configに定義されている。

# ネットワークの配置
network {
# 新しいNebulasチェンにある最初のノードに対して、`seed`は要らない。
# でなければ、ノードごとにいくつかのシードノードが必要として、Nebulasチェンに導入する。
# シード: ["/ip4/127.0.0.1/tcp/8680/ipfs/QmP7HDFcYmJL12Ez4ZNVCKjKedfE7f48f1LAkUc3Whz4jP"]

# P2pネットワークサービスホスト。多数のipとポートを支持する。
listen: ["0.0.0.0:8680"]

# プライベートキーはノードIDの生成に使用する。プライベートキーを使用しないなら、ノードは新たなノードIDを生成する。
# private_key: "conf/network/id_ed25519"
}

# チェンの配置
chain {
# ネットワークチェンID
chain_id: 100

# データベースストレージ場所
datadir: "data.db"

# アカウントのキーストアファイル場所
keydir: "keydir"

# ジェネシスブロックの配置
genesis: "conf/default/genesis.conf"

# サインアルゴリズム
signature_ciphers: ["ECC_SECP256K1"]

# マイナーアドレス
miner: "n1SAQy3ix1pZj8MPzNeVqpAmu1nCVqb5w8c"

# コインベースアドレス、すべての発掘報酬が上層のマイナーに受け取って、このアドレスに発送する
coinbase: "n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE"

# マイナーのキーストアファイルにパスフレーズ
passphrase: "passphrase"
}

# APIの配置
rpc {
# GRPC APIポート
rpc_listen: ["127.0.0.1:8684"]

# HTTP APIポート
http_listen: ["127.0.0.1:8685"]

# 公開されるモジュール
http_module: ["api", "admin"]
}

# ログの配置
app {
# ログレベル: [debug, info, warn, error, fatal]
log_level: "info"

# ログの位置
log_file: "logs"

# クラッシュログをオープン
enable_crash_report: false
}

# メトリックの配置
stats {
# ノードメトリックをオープン
enable_metrics: false

# Influxdbの配置
influxdb: {
host: "http://localhost:8086"
db: "nebulas"
user: "admin"
password: "admin"
}
}

```

多くの例が`$GOPATH/src/github.com/nebulasio/go-nebulas/conf/`にある

## Nodesを運行

> 現在運行しているこのNebulasチェンはプライベートでオフィシャルのテストネットとマインネットとは異なっている。

以下のコマンドで最初のNebulasノードを起動する。

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
./neb -c conf/default/config.conf
```

起動した後、以下のようなテキストはターミナルに見えるはず:
![seed node start](resources/101-01-seed-node-start.png)

基本的に、`conf/default/config.conf`を使用するノードは新たなブロックを送らない。別のコマンドで初期のNebulasマイニングノードを起動する。

```bash
cd $GOPATH/src/github.com/nebulasio/go-nebulas
./neb -c conf/example/miner.conf
```

このノードが起動後、もしシードノードに連結できたら、ログファイル`logs/miner/neb.log`に以下のようなテキストログが見える:
![node start](resources/101-01-node-start.png)

> 注意: ローカルでいくたのノードが起動できる。ノード配置にあるポートは他のポートを衝突しないように確保してください。

### 次の章: チュートリアル 2

[Nebulasで交易を発信する](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2002%20Transaction.md)

