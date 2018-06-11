# Nebulas 101 - 02 Sending Transactions on Nebulas

[Youtube Tutorial](https://www.youtube.com/watch?v=-44tVVR6ETo&list=PLFipfN18ZQwsW1_dge4w7dfsVNdNZZ37R&index=1)

> For this portion of the tutorial we will pick up where we left off in the [Installation tutorial](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2001%20Installation.md).

Nebulas provides three methods to send transactionsï¼š

1. Sign & Send
2. Send with Passphrase
3. Unlock & Send

Here is an introduction to sending a transaction in Nebulas through the three methods above and verifying whether the transaction is successful.

## Prepare Accounts

In Nebulas, each address represents a unique account.

Prepare two accounts: an address to send tokens (the sending address, called "from") and an address to receive the tokens (the receiving address, called "to").

### The Sender

Here we will use the coinbase account in `conf/example/miner.conf`, which uses `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE` as the sender. As the miner's coinbase account, it will receive some tokens as the mining reward. Then we could send these tokens to another account later.

### The Recipient

Create a new wallet to receive the tokens.

```bash
$ ./neb account new
Your new account is locked with a passphrase. Please give a passphrase. Do not forget this passphrase.
Passphrase:
Repeat passphrase:
Address: n1SQe5d1NKHYFMKtJ5sNHPsSPVavGzW71Wy
```

> When you run this command you will have a different wallet address with `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`. Please use your generated address as the recipient.

The keystore file of the new wallet will be located in `$GOPATH/src/github.com/nebulasio/go-nebulas/keydir/`

## Start the Nodes

### Start Seed Node

Firstly, start a seed node as the first node in the local private chain.

```bash
./neb -c conf/default/config.conf
```

### Start Miner Node

Secondly, start a miner node connecting to the seed node. This node will generate new blocks in the local private chain.

```bash
./neb -c conf/example/miner.conf
```

<<<<<<< HEAD
<<<<<<< c7256dcfda7a584d97247088ed33f60964ce28f8
> **How long a new block will be minted?**
>
> In Nebulas, DPoS is chosen as the temporary consensus algorithm before Proof-of-Devotion(PoD, described in [Technical White Paper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)) is ready. In this consensus algorithm, each miner will mint new block one by one every 15 seconds.
>
> In current context, we have to wait for 315(=15*21) seconds to get a new block because there is only one miner among 21 miners defined in `conf/default/genesis.conf` working now.
=======
=======
>>>>>>> e7608a0196fd73e8674dbcfd8bafb4228a4465cd
> **How long until a new block will be minted?**
> 
> In Nebulas, DPoS is chosen as the temporary consensus algorithm before Proof-of-Devotion(PoD, described in [Technical White Paper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)) is ready. In this consensus algorithm, each miner will mint a new block every 15 seconds.
> 
> In current context, we have to wait 315(=15*21) seconds to get a new block because there is only 1 miner working, among the 21 miners defined in `conf/default/genesis.conf`.
<<<<<<< HEAD
>>>>>>> fixes typos, grammar, and beautifies the docs
=======
>>>>>>> e7608a0196fd73e8674dbcfd8bafb4228a4465cd

Once a new block has been minted by the miner, the mining reward will be added to the coinbase wallet address used in `conf/example/miner.conf`, which is `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`.

## Interacting with Nodes

<<<<<<< HEAD
<<<<<<< c7256dcfda7a584d97247088ed33f60964ce28f8
Nebulas provides developers with HTTP API, gRPC API and CLI to interact with the running nodes. Here, we will share how to send a transaction in three methods with HTTP API ([API Module](https://github.com/nebulasio/wiki/blob/master/rpc.md) | [Admin Module](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)).
=======
Nebulas provides developers with HTTP API, gRPC API and CLI to interact with the running nodes. Here, we will share how to send a transaction using three methods with HTTP API ([API Module](https://github.com/nebulasio/wiki/blob/master/rpc.md) | [Admin Module](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)). 
>>>>>>> fixes typos, grammar, and beautifies the docs
=======
Nebulas provides developers with HTTP API, gRPC API and CLI to interact with the running nodes. Here, we will share how to send a transaction using three methods with HTTP API ([API Module](https://github.com/nebulasio/wiki/blob/master/rpc.md) | [Admin Module](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md)). 
>>>>>>> e7608a0196fd73e8674dbcfd8bafb4228a4465cd

> The Nebulas HTTP Listener is defined in the node configuration. The default port of our seed node is `8685`.

Firstly, check the sender's balance before sending a transaction.

### Check Account State

Fetch the state of sender's account `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE` with `/v1/user/accountstate` in API Module using `curl`.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE"}'

{
    "result": {
        "balance": "67066180000000000000",
        "nonce": "0",
        "type": 87
    }
}
```

> **Note**
> Type is used to check if this account is a smart contract account. `88` represents a smart contract account, and `87` is a non-contract account.

As we can see, the recipient has been rewarded with some tokens for mining new blocks.

Then let's check the recipient's account state.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"your_address"}'


{
    "result": {
        "balance": "0",
        "nonce": "0",
        "type": 87
    }
}
```

The new account doesn't have tokens as expected.

### Send a Transaction

Now let’s send a transaction in three methods to transfer some tokens from the sender to the recipient!

#### Sign & Send

In this manner, we can sign a transaction in an offline environment and then submit it to another online node. This is the safest method for everyone to submit a transaction without exposing your own private key to the Internet.

First, sign the transaction to get the raw data.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}, "passphrase":"passphrase"}'

{"result":{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}}
```

> **Note**
> Nonce is a very important attribute in a transaction. It's designed to prevent [replay attacks](https://en.wikipedia.org/wiki/Replay_attack). For a given account, only after its transaction with nonce N has been accepted, will its transaction with nonce N+1 be processed. Thus, we have to check the latest nonce of the account on chain before preparing a new transaction.

Then, send the raw data to an online Nebulas node.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/user/rawtransaction -d '{"data":"CiAbjMP5dyVsTWILfXL1MbwZ8Q6xOgX/JKinks1dpToSdxIaGVcH+WT/SVMkY18ix7SG4F1+Z8evXJoA35caGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwuKuC1wU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBVVuRHWSNY1e3bigbVKd9i6ci4f1LruDC7AUtXDLirHlsmTDZXqjSMGLio1ziTmxYJiLj+Jht5RoZxFKqFncOIQA="}'

{"result":{"txhash":"1b8cc3f977256c4d620b7d72f531bc19f10eb13a05ff24a8a792cd5da53a1277","contract_address":""}}â?Ž
```

#### Send with Passphrase

If you trust a Nebulas node so much that you can delegate your keystore files to it, the second method is a good fit for you.

First, upload your keystore files to the keydir folders in the trusted Nebulas node.

Then, send the transaction with your passphrase.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -d '{"transaction":{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":2,"gasPrice":"1000000","gasLimit":"2000000"},"passphrase":"passphrase"}'

{"result":{"txhash":"3cdd38a66c8f399e2f28134e0eb556b292e19d48439f6afde384ca9b60c27010","contract_address":""}}
```

> **Note**
> Because we have sent a transaction with nonce 1 from the account `n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE`, the new transaction with the same `from` should be increased by 1, to, namely, 2.

#### Unlock & Send

This is the most dangerous method. You probably shouldn’t use it unless you have complete trust in the receiving Nebulas node.

First, upload your keystore files to the keydir folders in the trusted Nebulas node.

Then unlock your accounts with your passphrase for a given duration in the node.
The unit of the duration is nano seconds (300000000000=300s).

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","passphrase":"passphrase","duration":"300000000000"}'

{"result":{"result":true}}
```

After unlocking the account, everyone is able to send any transaction directly within the duration in that node without your authorization.

```bash
> curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transaction -d '{"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":3,"gasPrice":"1000000","gasLimit":"2000000"}'

{"result":{"txhash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","contract_address":""}}â?Ž
```

## Transaction Receipt

We'll get a `txhash` in three methods after sending a transaction successfully. The `txhash` value can be used to query the transaction status.

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf"}'

{"result":{"hash":"8d69dea784f0edfb2ee678c464d99e155bca04b3d7e6cdba6c5c189f731110cf","chainId":100,"from":"n1FF1nz6tarkDVwWQkMnnwFPuPKUaQTdptE","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","value":"1000000000000000000","nonce":"3","timestamp":"1524667888","type":"binary","data":null,"gas_price":"1000000","gas_limit":"2000000","contract_address":"","status":1,"gas_used":"20000"}}â?Ž
```

The `status` fields may be 0, 1 or 2.

- **0: Failed.** It means the transaction has been submitted on chain but its execution failed.
- **1: Successful.** It means the transaction has been submitted on chain and its execution succeeded.
- **2: Pending.** It means the transaction hasn't been packed into a block.

### Double Check

<<<<<<< HEAD
<<<<<<< c7256dcfda7a584d97247088ed33f60964ce28f8
Let's double check the receiver's balance.
=======
Let's double check the recipient's balance. 
>>>>>>> fixes typos, grammar, and beautifies the docs
=======
Let's double check the recipient's balance. 
>>>>>>> e7608a0196fd73e8674dbcfd8bafb4228a4465cd

```bash
> curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5"}'

{"result":{"balance":"3000000000000000000","nonce":"0","type":87}}
```

Here you should see a balance that is the total of all the successful transfers that you have executed.
<<<<<<< HEAD

### Troubleshooting Step 02

In case your machine cannot resolve `http://localhost` using the `curl` command, ensure your /etc/hosts file contains the following:
```bash
::1             localhost
```
=======
>>>>>>> e7608a0196fd73e8674dbcfd8bafb4228a4465cd

### Next step: Tutorial 3

 [Write and run a smart contract with JavaScript](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md)
