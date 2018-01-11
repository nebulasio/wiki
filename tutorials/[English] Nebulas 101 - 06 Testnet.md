# Connecting to the test network i.e. Testnet

The test chain of the nebula chain has a specific chainID. When connecting to the test network, the configuration of the creation block in `genesis.conf` needs to be updated to the creation block configuration of the testing network and the configuration information is updated at the same time. Official introduction to [Testnet](https://github.com/nebulasio/wiki/blob/master/testnet.md)。

## Configuration steps

Create the file `conf/testnet-genesis.conf` and copy the following into it:

```
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

Create the file `conf/testnet-config.conf` and copy the following into it:

```
# Neb configuration text file. Scheme is defined in neblet/pb/config.proto:Config.
#

network {
  # Testnet seed node config
  seed:["/ip4/13.56.18.241/tcp/8680/ipfs/QmYcBY52pnuNQNMtsLUdKYQeLzDHZqfTj1RQGYs4Gujuqi", "/ip4/54.206.110.30/tcp/8680/ipfs/QmcFzHfFRHbp6o2WbTYvxv7uLH5mjSJXpRDMc3jKfy5ze4", "/ip4/54.238.223.81/tcp/8680/ipfs/Qmac11jvtGpFt9Ptevn4SHHQpvJjNsC17ZX7VmuHvsHM8o", "/ip4/13.250.10.239/tcp/8680/ipfs/QmY6d8qdHaa1XoMs76uQt8UpCcNJL77kx9R5ACwQPhZCF4", "/ip4/47.52.174.176/tcp/8680/ipfs/QmUQ77Jmqs99R8gjrJHNmz8LEf6HQMghUxbZNzwviR1LJn", "/ip4/35.182.48.19/tcp/8680/ipfs/QmW5HY9ef16pGvdryyJSDCz42ZiHEEmpmFuzYHpEBWvySG", "/ip4/35.177.86.207/tcp/8680/ipfs/QmYpPgrwzxcE1jbVfwqQmM7eSGd6LufpRmV76nGKT2kY7M"]
  listen: ["0.0.0.0:8680"]
  private_key: "conf/network/ed25519key"
  network_id: 1
}

chain {
  # Testnet chainID
  chain_id: 1001
  # TestNet data storage, different with private network
  datadir: "testnet.db"
  keydir: "keydir"
  # Testnet genesis conf
  genesis: "conf/testnet-genesis.conf"
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
    # Testnet log file dir
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
Start the Testnet connected Node:

```
./neb -c conf/testnet-config.conf
```

## Testnet NAS Tokens


NAS testing network can be obtained from the official website for testing. Distribution Address: [https://testnet.nebulas.io/claim/](https://testnet.nebulas.io/claim/).
