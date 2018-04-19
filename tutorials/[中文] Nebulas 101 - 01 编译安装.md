
# Nebulas 101 - 01 编译安装星云链

[星云链](https://nebulas.io/)的项目代码已经发布了几个版本，经过测试可以在本地运行，大家可以下载星云链源代码在本地编译私有链。

想了解星云链的同学可以阅读星云链[非技术白皮书](https://nebulas.io/docs/NebulasWhitepaperZh.pdf)。

对技术感兴趣的同学可以看星云链[技术白皮书](https://nebulas.io/docs/NebulasTechnicalWhitepaperZh.pdf)和星云链[github代码](https://github.com/nebulasio/go-nebulas)。

## 星云链环境搭建
星云链使用go语言开发，首先需要安装go语言的开发环境。推荐使用Mac搭建星云链环境。星云链对go版本的要求`>=1.9.2`。

*由于星云链的NVM(星云链虚拟机)使用了JavaScript的v8引擎，目前官方代码提供的v8依赖库只有Mac和Linux版本；星云链现阶段只能在Mac和Linux上运行，后续会推出windows版本。*

[官方go安装文档](https://golang.org/doc/install)

#### Mac系统安装go环境
Mac系统下安装go环境推荐使用`brew`安装。系统未安装`brew`的请参考[homebrew官网](https://brew.sh/)。

安装命令：([参考教程](http://www.jianshu.com/p/358cbc939569))

```
brew install go
```

成功安装后可以使用`go env`查看go的版本信息
![go env](resources/101-01-go-env.png)

安装完成后需要设置的环境变量包括:`GOPATH ` ,`GOBIN ` 以及把`GOBIN`加入到`PATH`中。

编辑`~/.bash_profile`:

```
    export GOPATH=<gopath>
    export GOBIN=$GOPATH/bin
    export PATH=$PATH:$GOBIN
```
**注意：GOPATH是本地的go工作目录，可以自行配置，在GOPATH配置完后，go项目工程需要放置到GOPATH目录中。**

#### Linux系统安装go
Linux安装建议使用源代码安装go，安装教程可以参考[在Linux上安装Go](https://github.com/Unknwon/the-way-to-go_ZH_CN/blob/master/eBook/02.3.md)。

## 星云链编译

#### 源代码下载
首先从github网站conle代码到本地(本教程使用[v1.0.0版本](https://github.com/nebulasio/go-nebulas/tree/v1.0.0))

```
git clone -b v1.0.0 https://github.com/nebulasio/go-nebulas.git --depth=1
```
如果需要完整代码的提交历史，可以全部clone到本地：

```
git clone https://github.com/nebulasio/go-nebulas.git
git checkout master
```
由于go代码编译必须在`$GOPATH`中, 星云链代码要放到go的`$GOPATH`中`/src/github.com/nebulasio/go-nebulas`的位置。请使用`master`分支来运行星云节点。

#### 安装rocksdb依赖库

可参考 [rocksdb 安装说明](https://github.com/facebook/rocksdb/blob/master/INSTALL.md)。

** 对于Mac系统**
 * 首先要安装支持 C++11 的 C++ 编译器。具体步骤如下。

    * 升级 XCode。可以通过 XCode 应用设置（XCode App's settting）来更新，或者通过以下指令：
    
       `  xcode-select --install`
    
    *  通过 [homebrew](http://brew.sh/)安装.

        * 如果你是首次用Mac开发，你仍需要运行一次`xcode-select --install`.
        * 安装GCC 4.8 或更高版本:
    
            `brew tap homebrew/versions`        
            `brew install gcc48 --use-llvm`
          
 * 安装 rocksdb:

      `brew install rocksdb`

** 对于Linux系统**
   * 升级 gcc 到4.8及以上，以支持C++11
   
        查询版本 gcc :   `gcc --version`
   
        安装 gcc-4.9 :   `sudo apt-get install gcc-4.9`
   * 安装 gflags 。
    
        `sudo apt-get install libgflags-de`
          
        如果该指令无效，对于Ubuntu，您可以参考这个[指南](http://askubuntu.com/questions/312173/installing-gflags-12-04)。
   * 安装 snappy 。
   
        `sudo apt-get install libsnappy-dev`
   * 安装 zlib 。
   
        `sudo apt-get install zlib1g-dev`
   * 安装 bzip2 。
   
        `sudo apt-get install libbz2-dev`
   * 安装 lz4 。
   
        `sudo apt-get install liblz4-dev`
   * 安装 zstandard 。
   
        `sudo apt-get install libzstd-dev`


#### 安装go依赖库
Nebulas的go代码库依赖管理使用[dep](https://github.com/golang/dep)，开发中使用的第三方包可以使用dep下载。

使用`brew`安装`dep`:

```
$ brew install dep
$ brew upgrade dep
```
切换至项目根目录安装go的依赖库：

```
cd <path>/go-nebulas
make dep
```
**PS：make dep需要下载的依赖较多，初次下载较慢，部分依赖库国内网络下载可能失败。若无法下载，可以直接将dep后生成的文件[vendor.tar.gz](http://ory7cn4fx.bkt.clouddn.com/vendor.tar.gz)下载后解压到代码根目录中。**

```
vendor.tar.gz
SHA1: 156d0cf7bcf1fbd96c6ee87ff01b77f8a9227eda
MD5: c2c1ff9311332f90e11fb81b48ca0984
```
也可以在终端设置代理来下载，代理设置指令为(代理地址只是举例)：
```$xslt
export http_proxy = http://127.0.0.1:1080;
```
下载完成后取消代理设置，不然有可能影响 Nebulas 节点运行：
```$xslt
unset http_proxy
```
#### 安装v8
Nebulas的NVM(星云链虚拟机)使用了JavaScript的v8引擎，需要安装为NVM编译的v8依赖库后`neb`才能运行。v8依赖库星云链官方目前提供了Mac版本的动态链接库`libnebulasv8.dylib`和Linux版本的静态链接库`libnebulasv8.so`及其他so库。项目中已经添加了make命令安装v8依赖库，在项目根目录执行安装指令：

```
make deploy-v8
```
如果不使用make集成v8链接库，也可以单独安装：

* Mac系统
	* `install nf/nvm/native-lib/libnebulasv8.dylib /usr/local/lib/`
* Linux系统
	* `sudo install nf/nvm/native-lib/*.so /usr/local/lib/`
	* `sudo /sbin/ldconfig`
	
#### 编译可执行文件
完成go依赖库和v8依赖包后，可以编译星云链的可执行文件。Nebulas的go工程main函数在`cmd/neb/main.go`中，在项目根目录执行make命令进行编译：

``` make build
```
编译成功后，在根目录生成`neb`可执行文件。
![make build](resources/101-01-make-build.png)

## 搭建本地测试环境

#### 创世区块配置
星云链启动时要配置创世区块的信息，在第一次启动时会使用创世区块配置初始化区块信息。目前星云链暂时使用dpos作为共识算法，初始挖矿成员和NAS的预分配可以在创世区块配置中设置。

创世区块配置信息：

```
# Neb genesis text file. Scheme is defined in core/pb/genesis.proto.
#

meta {
  # 星云链ID，私有网络默认为100，测试网络为1001
  chain_id: 100
}

consensus {
  # dpos 初始挖矿成员配置
  dpos {
    dynasty: [
       "n1FkntVUMPAsESuCAAPK711omQk19JotBjM",
       "n1JNHZJEUvfBYfjDRD14Q73FX62nJAzXkMR",
       "n1Kjom3J4KPsHKKzZ2xtt8Lc9W5pRDjeLcW",
       "n1TV3sU6jyzR4rJ1D7jCAmtVGSntJagXZHC",
       "n1WwqBXVMuYC3mFCEEuFFtAXad6yxqj4as4",
       "n1Zn6iyyQRhqthmCfqGBzWfip1Wx8wEvtrJ"
    ]
  }
}

# NAS预分配地址金额，
token_distribution [
  {
    address: "n1Z6SbjLuAEXfhX1UJvXT6BB5osWYxVg3F3"
    value: "10000000000000000000000"
  },
  {
    address: "n1NHcbEus81PJxybnyg4aJgHAaSLDx9Vtf8"
    value: "10000000000000000000000"
  }
]
```
创世区块配置默认放在`conf/default/genesis.conf`中，配置创世区块路径可以在种子节点的配置文件中设置。

#### 节点
星云链节点可以通过执行编译后的`neb`可执行文件启动。节点启动需在终端执行，Neb节点包括种子节点和节点：

* 种子节点：星云链网络种子节点，为其他节点提供初始同步服务；
* 节点：星云链网络普通节点，启动后会先从种子节点同步路由和区块信息。

星云链的种子节点和节点启动通过配置文件来区分。节点启动需要先启动种子节点，在种子节点启动后，将种子节点的网络地址信息更新到普通节点的配置文件中，可以组网挖矿。

#### 启动种子节点
启动星云链的节点需要配置文件提供部分配置参数。配置文件使用了使用[Protocol Buffer](https://github.com/google/protobuf)的格式读取配置信息。工程根目录下有默认种子节点配置文件：

`conf/default/config_local.conf`

种子节点配置文件内容如下：

```
# 节点网络配置，种子节点和节点的配置在此区分
network {
  # 若为种子节点，不需要配置seed，普通节点需要配置种子节点seed信息
  # seed: "UNCOMMENT_AND_SET_SEED_NODE_ADDRESS"
  # p2p网络服务ip和端口，服务启动的时候可以listen多组不同的ip和端口
  listen: ["127.0.0.1:8680"]
  # 生成节点ID时候用到的私钥路径，如果不配置，每次都会生成新的不同的节点ID；配置了，会使用配置的私钥生成节点ID
  #private_key: "conf/network/ed25519key"
}

# blockchain相关配置
chain {
  # 网络中的chainID,测试网络为1001，此处ID需要与创世区块配置中的ID一致
  chain_id: 100
  # 数据库存放的位置
  datadir: "data.db"
  # 节点私钥保存位置
  keydir: "keydir"
  # genesis创世区块的默认配置
  genesis: "conf/default/genesis.conf"
  # 矿机的挖矿地址，获取的奖励将发放给coinbase
  coinbase: "n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5"
  # 节点签名算法
  signature_ciphers: ["ECC_SECP256K1"]
  # 节点挖矿的地址，key文件需要放在`keydir`中
  miner: "n1Zn6iyyQRhqthmCfqGBzWfip1Wx8wEvtrJ"
  # 用于解锁挖矿账户的passphrase
  passphrase: "passphrase"
}

# 用户与节点交互的服务配置
rpc {
    # gRPC API服务端口
    rpc_listen: ["127.0.0.1:8684"]
    # HTTP API服务端口
    http_listen: ["127.0.0.1:8685"]
    # 开放可对外提供http服务的模块
    http_module: ["api","admin"]
}

# app日志相关配置
app {
    # 配置记录的log级别{debug, info, warn, error, fatal}
    log_level: "info"
    # 配置log的输出文件夹
    log_file: "logs"
    # 配置是否输出crash日志
    enable_crash_report: false
}

# 其他配置信息
stats {
    # 是否开启节点监控数据统计
    enable_metrics: false
    # 节点监控数据统计用到的inluxdb的配置
    influxdb: {
        host: "http://localhost:8086"
        db: "nebulas"
        user: "admin"
        password: "admin"
    }
}

```

详细的配置文件说明请参考[配置文件示例](https://github.com/nebulasio/wiki/blob/master/resources/conf/template.conf)。

在不指定配置文件时默认读取工程根目录下的`conf/default/config.conf`启动种子节点。默认启动种子节点命令:

```
./neb
```
若需要使用不同的配置文件，仅需在启动时添加`-c`标记，指定配置文件。例如启动种子节点时指定节点配置文件：

```
./neb -c <path>/config.conf
```
在完成配置文件修改后可以启动节点。启动后可以在终端上看到类似如下信息：
![seed node start](resources/101-01-seed-node-start.png)

#### 启动节点
在种子节点启动后如果需要启动普通节点组网与种子节点连接，需要在普通节点配置文件中配置种子节点地址信息，种子节点地址可以从终端打印信息log:**Started NebService Node.**中获取：
![seed node start](resources/101-01-started-nebService-node.png)

上面的log中，地址信息为`/ip4/127.0.0.1/tcp/8680`,id为`QmP7HDFcYmJL12Ez4ZNVCKjKedfE7f48f1LAkUc3Whz4jP `，星云链p2p网络使用了ipfs的libp2p网络库，所以种子地址的格式为下述所示:

```
<address>/ipfs/<id>
```
在工程根目录的`conf/examples/`目录下预置了几个普通节点的配置文件。这里我们以普通节点配置文件`config.1a2635.conf`为例，在配置文件中有种子节点地址信息：

```
network {
  # seed: "UNCOMMENT_AND_SET_SEED_NODE_ADDRESS"
  seed: ["/ip4/127.0.0.1/tcp/8680/ipfs/QmP7HDFcYmJL12Ez4ZNVCKjKedfE7f48f1LAkUc3Whz4jP"]
  listen: ["127.0.0.1:10001"]
}
...
```
**Note:若在同一台机器上配置多个节点，注意避免端口占用。**

启动普通子节点时，使用此配置文件启动节点：

```
./neb -c conf/example/config.1a2635.conf
```
节点启动后，如果与种子节点连接成功，可以在logs/normal.1a2635/目录下的log文件中找到下面的log信息：
![node start](resources/101-01-node-start.png)
