

# Nebulas 101 - 01 Compile and Install Nebulas

The project code for [Nebulas](https://nebulas.io/) has been released in several versions and tested to run locally. You can download the Nebulas source code to compile the private chain locally.

To learn about Nebulas, please read the Nebulas [Non-Technical White Paper](https://nebulas.io/docs/NebulasWhitepaper.pdf).

To learn about the technology, please read the Nebulas [Technical White Paper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf) and the Nebulas [github code](https://github.com/nebulasio/go-nebulas).

### Installing prerequisites
#### Install prerequisites on Linux (Ubuntu)
We'll install some basic tools needed for this guide:
```
sudo apt-get install git make wget
```

### Nebulas Environment Set Up

Nebulas is implemented in Go Language. Installing Go Lang is required. Go Lang version `>=1.8`.

*NVM (Nebulas Virtual Machine) uses JavaScript v8 engine. As currently v8 only run on Mac and Linux versions, Nebulas can only run on Mac and Linux at this stage. The Windows version will be coming soon.*

[Go Lang Installation](https://golang.org/doc/install)

#### Install Go Environment on Mac 
`brew` is recommended for installation. To install `brew`, pleace go to [homebrew](https://brew.sh/).

Installation command:

```
brew install go
```

After installation, use `go env` to check version for Go
![go env](resources/101-01-go-env.png)

#### Setup project folders on Mac
first we should create a folder named "go" and place that in the documents folder. Then we should open up go folder and create another folder named src. So your project folder should be on Mac; Users/yourUserName/Documents/go/src/. For now just remember this location; Users/yourUserName/Documents/go

The environment variables that need to be set after installation are: `GOPATH`,`GOBIN`. Add `GOBIN` to `PATH`.

Find you .bash_profile in Mac- Users/myUserName/.bash_profile
If you you don’t see the file you need to go to open up terminal and type

```
defaults write com.apple.finder AppleShowAllFiles TRUE
```

Open up .bash_profile and copy paste this below.

Edit `~/.bash_profile`:

```
    export GOPATH=/Users/yourUserName/Documents/go
    export GOBIN=$GOPATH/bin
    export PATH=$PATH:$GOBIN
```
 **Note the export GOPATH=/Your/location/of/go (refer to Setup project folders) you must get the exact location then add it to the .bash_profile**

### Save the changes to your .bash_profile file.
Now restart the terminal: this is important to continue in the next step

**Notice:GOPATH is a local go work directory and can be configured on its own. After GOPATH is configured, your go projects need to be placed in GOPATH directory.**

#### Install Go Environment on Linux (Ubuntu)
See https://golang.org/doc/install
Download the golang archive
```
wget https://dl.google.com/go/go1.9.3.linux-amd64.tar.gz
```
 Extract it into /usr/local, creating a Go tree in /usr/local/go. For example: 
```
tar -C /usr/local -xzf go1.9.3.linux-amd64.tar.gz
```
Add /usr/local/go/bin to the PATH environment variable in your $HOME/.bashrc (or your profile or wherever you want it to be):
```
# Added for golang
export PATH=$PATH:/usr/local/go/bin
```
#### Setup project folders on linux (Ubuntu)
Create your project folder (Taking $HOME/go as an example, but you can basicly put it anywhere you want)
```
mkdir -p $HOME/go/src
```
Set GOPATH in your $HOME/.bashrc (Make sure to change <change_this_path to where you've put the project folder!)
```
export GOPATH=<change_this_path>/go
```

### Compile Nebulas

#### Download Source Code (Mac and Linux)：
Clone from GitHub (This tutorial uses [v0.5.0](https://github.com/nebulasio/go-nebulas/tree/v0.5.0))

```
git clone -b v0.5.0 https://github.com/nebulasio/go-nebulas.git --depth=1
```

If you need the full commit history, clone all to local:

```
git clone https://github.com/nebulasio/go-nebulas.git
git checkout master
```

Note: If you opt for the second option, don't forget to checkout to `master` branch or to commit `ed37bea` (v0.5.0.), otherwise this tutorial might not work for you.

Since Go must compile in `$GOPATH`,  Nebulas code must be in `/src/github.com/nebulasio/go-nebulas` under `$GOPATH`。

Note: make sure you create a src folder and put go-nebulas inside it. The folder structure that includes `github.com` is a best practice not a requirement.

![folder setup](https://user-images.githubusercontent.com/21117852/34554205-c2e7b4c8-f166-11e7-9ab8-8d0da4718355.png)

#### Install Go Dependencies
##### Install dep for mac
Nebulas' Go code dependency uses [dep](https://github.com/golang/dep). Third-party packages used in development can be downloaded using dep.

Use `brew` to install `dep`:

```
$ brew install dep
$ brew upgrade dep
```
##### Install dep for linux
Install dep in /usr/local/bin so you can use it to download third-party packages
```
cd /usr/local/bin/
wget https://github.com/golang/dep/releases/download/v0.3.2/dep-linux-amd64
ln -s dep-linux-amd64 dep
```
###### Install dependencies for Mac/linux

Switch to the root directory of the project Install dependencies for Go:

```
cd <path>/go-nebulas
make dep
```

**PS: `make dep` downloads many dependencies. It would take a long time to download the first time. Some dependencies may fail to download. If you can not download, you can directly download the file generated by dep [vendor.tar.gz](http://ory7cn4fx.bkt.clouddn.com/vendor.tar.gz) and extract it to the code root directory.**

```
vendor.tar.gz
SHA1: 250eccb8f0a48277765c6266bed55187b6c77b2b
MD5: 13ef26ab05aad391f540f82af07716df
```

**PS: If it doesn’t work and you get an error: validateParams: could not deduce external imports' project roots
make:[dep] Error 1.
You should use a vpn and route all information through the vpn or export proxy in the terminal: example; `export http_proxy=http://127.0.0.1:1087`; then try `make dep`**

#### Install V8 for Mac/linux

Nebulas's NVM (Nebulas Virtual Machine) uses the JavaScript V8 engine, and the V8 dependencies for NVM need to be run with `neb` installed. The Mac version of the dynamic link library `libnebulasv8.dylib` and the Linux version of the static link library` libnebulasv8.so` and other libraries are provided by the official V8 Dependency Library for Nebulas. A make command to install V8 dependency libraries has been added. Execute the installation command in the go-nebulas folder which is the project root directory using the terminal:

```
cd go/src/github.com/go-nebulas
make deploy-v8
```

After running make deploy-v8 you should see: 
* On Mac:
```
install nf/nvm/native-lib/*.dylib /usr/local/lib/`
```
* On linux:
```
sudo install nf/nvm/native-lib/*.so /usr/local/lib/
sudo /sbin/ldconfig
```

If you do not use make to integrate V8 link library, you can also install it separately:

* Mac
	* `install nf/nvm/native-lib/libnebulasv8.dylib /usr/local/lib/`
* Linux
	* `sudo install nf/nvm/native-lib/*.so /usr/local/lib/`
	* `sudo /sbin/ldconfig`


#### Make Build for Mac/linux

```
cd go/src/github.com/go-nebulas
make build
```

**PS: if you get an error like `cannot find package`. Step 1: export proxy to terminal $ `export https_proxy=http://127.0.0.1:1087`. Step 2 `make dep` or `dep ensure`. Now you should be able to `make build`.**

Developers note: The Nebulas Go main function is in `cmd/neb/main.go`



Once the build is complete，you will see a generated `neb` file in go-nebulas folder, which is the root directory.
![make build](resources/101-01-make-build.png)


## Starting a Node

### Creation block (i.e. Genesis block) configuration

When the nebula chain is started, configure the information of the **creation block** and use the creation block to configure the **initialization block** information when it is started for the first time. At present, the nebula chain temporarily uses **dpos** (Distributed Proof of Stake) as a consensus algorithm. The pre-allocation of initial mining members and NAS can be set in the creation block configuration.

#### Configuration Information

The default configuration shown below can be found at `conf/default/genesis.conf`

```
# Neb genesis text file. Scheme is defined in core/pb/genesis.proto.
#

meta {
  # chainID，private default 100，Testnet 1001
  chain_id: 100
}

consensus {
  # dpos genesis miner
  dpos {
    dynasty: [
    "1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c",
    "2fe3f9f51f9a05dd5f7c5329127f7c917917149b4e16b0b8",
    "333cb3ed8c417971845382ede3cf67a0a96270c05fe2f700",
    "48f981ed38910f1232c1bab124f650c482a57271632db9e3",
    "59fc526072b09af8a8ca9732dae17132c4e9127e43cf2232",
    "75e4e5a71d647298b88928d8cb5da43d90ab1a6c52d0905f",
    "7da9dabedb4c6e121146fb4250a9883d6180570e63d6b080",
    "98a3eed687640b75ec55bf5c9e284371bdcaeab943524d51",
    "a8f1f53952c535c6600c77cf92b65e0c9b64496a8a328569",
    "b040353ec0f2c113d5639444f7253681aecda1f8b91f179f",
    "b414432e15f21237013017fa6ee90fc99433dec82c1c8370",
    "b49f30d0e5c9c88cade54cd1adecf6bc2c7e0e5af646d903",
    "b7d83b44a3719720ec54cdb9f54c0202de68f1ebcb927b4f",
    "ba56cc452e450551b7b9cffe25084a069e8c1e94412aad22",
    "c5bcfcb3fa8250be4f2bf2b1e70e1da500c668377ba8cd4a",
    "c79d9667c71bb09d6ca7c3ed12bfe5e7be24e2ffe13a833d",
    "d1abde197e97398864ba74511f02832726edad596775420a",
    "d86f99d97a394fa7a623fdf84fdc7446b99c3cb335fca4bf",
    "e0f78b011e639ce6d8b76f97712118f3fe4a12dd954eba49",
    "f38db3b6c801dddd624d6ddc2088aa64b5a24936619e4848",
    "fc751b484bd5296f8d267a8537d33f25a848f7f7af8cfcf6"
    ]
  }
}

# NAS dispatch，
token_distribution [
  {
    address: "1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c"
    value: "10000000000000000000000"
  },
  {
    # If the first address goes to 0 then this address is used as a backup
    address: "2fe3f9f51f9a05dd5f7c5329127f7c917917149b4e16b0b8"
    value: "10000000000000000000000"
  }
]
```

### Nodes

Neb nodes include **seed nodes** and ordinary **nodes**.

**Seed Node**: a Nebula chain network **seed node** is used to provide initial synchronization services for other nodes;

**Node**: a Nebula chain ordinary **node**, starts from the **seed node** synchronization routing and block information.
Nebula chain seed nodes and nodes start to distinguish through the configuration file. When the node is started, the seed node needs to be started first. After the seed node is started, the network address information of the seed node is updated to the configuration file of the ordinary node, and the network can be mined.

### Review config.conf

Now we need to review the config.conf file which is located in `go-nebulas/conf/default/config.conf`

*This version of `config.conf` includes inline comments that are not included in the file in the go-nebulas repo.*

```
# Neb configuration text file. Scheme is defined in neblet/pb/config.proto:Config.
#

network {
  # If this is the seed node, configuration is not needed. The normal node needs the seed node seed
  # seed: "UNCOMMENT_AND_SET_SEED_NODE_ADDRESS"
  # p2p network service host. support mutiple ip and ports.
  listen: ["127.0.0.1:8680"]
  # the private key is used to generate a node ID. If you don't use the private key, the node will get a new ID.
  # private_key: "conf/network/id_ed25519"
  # network group ID. nodes can't connect to each other in different network groups.
  network_id: 1
}

chain {
  # Network chain ID. defult: 100
  chain_id: 100
  # Database storage location
  datadir: "data.db"
  # Node private key location
  keydir: "keydir"
  # The genesis block configuration
  genesis: "conf/default/genesis.conf"
  # Mining machine's mining address, the reward will be send to a Coinbase address
  coinbase: "eb31ad2d8a89a0ca6935c308d5425730430bc2d63f2573b8"
  # Node signature algorithm
  signature_ciphers: ["ECC_SECP256K1"]
  # Mining machine's mining address
  miner: "eb31ad2d8a89a0ca6935c308d5425730430bc2d63f2573b8"
  # The passphrase used to unlock account
  passphrase: "passphrase"
}

# Service configuration of interaction between user and node.
rpc {
    # GRPC API port
    rpc_listen: ["127.0.0.1:8684"]
    # HTTP API port
    http_listen: ["127.0.0.1:8685"]
    # The module that user can access by HTTP
    http_module: ["api","admin"]
}

# Configuration of log
app {
    # Log level {debug, info, warn, error, fatal}
    log_level: "info"
    # The output path of Log
    log_file: "logs"
    # Crash log enabling
    enable_crash_report: false
}

# Other Configuration
stats {
    # Node Metrics Enabling
    enable_metrics: false
    # Configuration of influxdb
    influxdb: {
        host: "http://localhost:8086"
        db: "nebulas"
        user: "admin"
        password: "admin"
    }
}

```

## Running your Node

### Starting a Seed Node

**Note: the Node network you are running at this point is private and is not connected to the Testnet**

The default command to start a seed node:

```
./neb
```

By default neb looks for a configuration file in `conf/default/config.conf` to start a seed node if no configuration file is specified.

If you are using a different configuration file than the default, add the `-c` flag at startup to specify the configuration file. For example, to specify a node configuration file when starting a seed node:

```
./neb -c <path>/new-config.conf

```

After starting, the following should be visible in the terminal:
![seed node start](resources/101-01-seed-node-start.png)

### Starting a Node

After starting the seed node, if you need to start a normal node network connected with the seed node you must add the seed node address information in the normal node's configuration file.

Use this example configuration file `conf/example/config.1a2635.conf` to start a Node that is connected to the seed node you started above. Leave the seed node running, open another terminal window inside of the `go-nebulas` directory and run the following command:

```
./neb -c conf/example/config.1a2635.conf
```

After the node starts, if the connection with the seed node is successful, you can see the following log:
![node start](resources/101-01-node-start.png)

#### Creating your own Node config

**Note: this step is optional for this tutorial but is useful for better understanding how Nodes reference seed nodes.**

To create a Node that connects with your seed node you must create a config file that references the seed address.

The seed node `address` and `id` can be found from the seed node log on the line beginning with **node start**:

```
INFO[2017-12-25T15:04:52+08:00] node start       addrs="[/ip4/127.0.0.1/tcp/8680]" file=net_service.go func="p2p.(*NetService).Start" id=QmPyr4ZbDmwF1nWxymTktdzspcBFPL6X1v3Q5nT7PGNtUN line=693
```
In the log above, the **address** is `/ip4/127.0.0.1/tcp/8680`, the **id** is `QmPyr4ZbDmwF1nWxymTktdzspcBFPL6X1v3Q5nT7PGNtUN `. The Nebulas p2p network uses IPFS's libp2p network library. The format of the seed address is:

```
<address>/ipfs/<id>
```

Now copy the configuration file `conf/default/config.conf` to `conf/my-node-config.conf` and ensure that the network block looks like the following:

```
network {
  # seed: "UNCOMMENT_AND_SET_SEED_NODE_ADDRESS"
  seed: ["/ip4/127.0.0.1/tcp/8680/ipfs/QmPyr4ZbDmwF1nWxymTktdzspcBFPL6X1v3Q5nT7PGNtUN"]
  listen: ["127.0.0.1:10001"]
  network_id: 1
}@
...
```
You can now run this node with:

```
./neb -c conf/my-node-config.conf
```

**Note: If configuring multiple nodes on the same machine, make sure to avoid port override**

In order to avoid port override ensure that the port `10001` in the network block above `listen: ["127.0.0.1:10001"]` is different for each node you start.

### Next step: Tutorial 2:

[Sending Transactions on Nebulas](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2002%20Transaction.md)
