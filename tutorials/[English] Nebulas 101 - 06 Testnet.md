# Connecting to the test network i.e. Testnet

The configuration information will be updated at the same time. Official introduction to [Testnet](testnet.md)

## Configuration steps for branch `develop`

The configaration files are in folder `testnet/conf`:
 - config.conf
 - genesis.conf

Generate your own private key for testnet:
```
./neb network ssh-keygen testnet/conf/network.key
```

Start the Testnet connected Node:
```
./neb -c testnet/conf/config.conf
```

## Configuration steps for branch `master`

There are no configuration files for Tesnet. You need to create them by youself.

You could use instructions from [Testnet](testnet.md): just copy the files from this wiki to you computer:
- [testnet-config.conf](resources/conf/testnet-config.conf)
- [testnet-genesis.conf](resources/conf/testnet-genesis.conf)

For example, you want to save this files in `conf/default` folder (where config files for mainnet saved) use:
```
cd conf/default
wget https://raw.githubusercontent.com/nebulasio/wiki/master/resources/conf/testnet-config.conf
wget https://raw.githubusercontent.com/nebulasio/wiki/master/resources/conf/testnet-genesis.conf
```
If you prefer `curl` instead `wget` use:
```
curl -O https://raw.githubusercontent.com/nebulasio/wiki/master/resources/conf/testnet-config.conf
curl -O https://raw.githubusercontent.com/nebulasio/wiki/master/resources/conf/testnet-genesis.conf
```

Return to project root folder:
```
cd ../..
```
And start your Testnet connected node (you need to build `neb` before, see [Nebulas 101 - 01 Compile and Install Nebulas](tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2001%20Installation.md)):
```
./neb -c conf/default/config-testnet.conf
```

## Testnet NAS Tokens

NAS testing network can be obtained from the official website for testing. Distribution Address: [https://testnet.nebulas.io/claim/](https://testnet.nebulas.io/claim/).

#### code playground coming soon!
