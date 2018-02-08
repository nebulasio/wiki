# Connecting to the test network i.e. Testnet

The configuration information will be updated at the same time. Official introduction to [Testnet](https://github.com/nebulasio/wiki/blob/master/testnet.md)

The configaration files are in folder `testnet/conf`.

## Configuration steps

Generate your own private key for testnet:
```
./neb network ssh-keygen testnet/conf/network.key
```

Start the Testnet connected Node:

```
./neb -c testnet/conf/config.conf
```

## Testnet NAS Tokens


NAS testing network can be obtained from the official website for testing. Distribution Address: [https://testnet.nebulas.io/claim/](https://testnet.nebulas.io/claim/).


#### code playground coming soon!
