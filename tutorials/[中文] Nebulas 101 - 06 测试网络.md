# Nebulas 101 - 06 连接测试网络
星云链的测试网络有特定的chainID,在连接测试网络时需要将创世区块配置`genesis.conf`更新为测试网络的创世区块配置，同时更新配置信息。官方的介绍[Testnet](https://github.com/nebulasio/wiki/blob/master/testnet.md)。

## 创世区块配置

更新创世区块配置为测试网络的创世区块配置`testnet-genesis.conf`(必须与此保持一致):

```
meta {
  # Each Nebulas Net will have an unique chain id.
  # 1001-2000 are reserved for Nebulas Testnets.
  # Current chain ids of official Testnets only include 1001.
  chain_id: 1001
}

consensus {
  # Before PoD consensus is ready, Nebulas will use Dpos consensus instead.
  dpos {
    # Here is the initial validators in the first dynasty.
    dynasty: [
    "d06d3cce5ddd122bdfb245cd581244513b88d3b4b60b5269",
    "d1e142b8233acb243e7246f72f78443043e805ec4dfd1b80",
    "6cf0c56ddba75a23ae3c69ac9a0b8f84b1ecd5e7d94d0700",
    "190152c4c7caff32e3e26caf1cd9967e0af324cf4099e0f9",
    "fd4cc8b6481490a8ff3f9c67fc1bb62c72337b3c97c7cf52",
    "1e9aa5a2d9d2f24ed08defd755261dd9a1076866340753e2",
    "c27e4c91e8ca0dfcdd894be955fec93aa44cccd712576359",
    "afde759c73591600e5855039b12c6a5f9a8950cc5da952a3",
    "f27c2086c59617ae31366e7a7ce6697d05d0192d3ba3dad1",
    "8e597e6dc45dca8b34a61ec2fc383081b605b73fd3edc348",
    "6b4a660a49014232d0a91e078b87301c982b017b499633af",
    "a12e917e9518344c39d0baa6b7ae9fd460a05ab46afd4e7b",
    "a59da2431ef695f14c39ca030b85da3af24900d98524d548",
    "955b7e78c6034ffdb021a48ec351d676aac6e730e38747ad",
    "9684b3d17cd7467c454066da86f7cc16f917d49ddc7e05fa",
    "ad2c359ad3b6f552b26461cefa47df7c1d2d534713b2e1a2",
    "a7115c6e605587e5f442ef2e3c55e5f7e2fd917cd44a76e1",
    "a5f1a47fe5e43b76a7a66da5e0427e5ef22f9f520753363d",
    "5cfdd8be8a9cf26ff34a6fa499a0a5bb2e9f7bd9e4922fd6",
    "c220a3c759caf1de6d6262f4352d2b64447733c5a2c00296",
    "233805a3b6eb9e5501ecbf53df90978e62302142b2d52806"
    ]
  }
}

# Initial Token Distribution is described here
token_distribution [
  {
    address: "0b9cd051a6d7129ab44b17833c63fe4abead40c3714cde6d"
    value: "10000000000000000000000000"
  },
  {
    address: "4aa6312b10dd81f68eb62a90e234a48eb83c6b0e568b5eb8"
    value: "10000000000000000000000000"
  }
]
```

## 配置文件配置

更新配置文件内容并保存到`testnet-config.conf`：

```
# Neb configuration text file. Scheme is defined in neblet/pb/config.proto:Config.
#

network {
  # 测试网络的种子节点
  seed:["/ip4/13.57.245.249/tcp/8680/ipfs/Qme7PkuEWLajPmPmKNPZ7MAonK7MjDomReLbEzdy8yJHo4", "/ip4/54.219.151.126/tcp/8680/ipfs/QmVUPFV9DEwaNBM5p8CrEJeXSHFGLdST5cbRaTkUjVrF6r", "/ip4/18.218.165.90/tcp/8680/ipfs/Qmac11jvtGpFt9Ptevn4SHHQpvJjNsC17ZX7VmuHvsHM8o", "/ip4/35.176.94.224/tcp/8680/ipfs/QmTZDELLwtAYzC7U1WpofKrfT9c652VkMoTSum3cfadHM5"]
  listen: ["0.0.0.0:8680"]
  private_key: "conf/network/ed25519key"
  network_id: 1
}

chain {
  # 测试网络的chainID
  chain_id: 1001
  # 测试网络的数据保存位置，需要与本地网络做区分
  datadir: "testnet.db"
  keydir: "keydir"
  # 测试网络的创世区块配置，内容为上面的测试创世区块配置
  genesis: "conf/default/testnet-genesis.conf"
  coinbase: "eb31ad2d8a89a0ca6935c308d5425730430bc2d63f2573b8"
  signature_ciphers: ["ECC_SECP256K1"]
  miner: "9341709022928b38dae1f9e1cfbad25611e81f736fd192c5"
  passphrase: "passphrase"
}

rpc {
    rpc_listen: ["127.0.0.1:8684"]
    http_listen: ["127.0.0.1:8685"]
    http_module: ["api","admin"]
}

app {
    log_level: "info"
    # 测试网络的log文件夹，可以修改做区分
    log_file: "logs/testnet"
    enable_crash_report: false
}

stats {
    enable_metrics: false
    influxdb: {
        host: "http://localhost:8086"
        db: "nebulas"
        user: "admin"
        password: "admin"
    }
}

```

使用上面的配置文件启动节点:

```
./neb -c <path>/testnet-config.conf
```

测试网络的NAS可以从官方的网站分发获取，用于测试。分发地址：[https://testnet.nebulas.io/claim/](https://testnet.nebulas.io/claim/).
