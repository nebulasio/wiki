# Nebulas 101 - 02 Sending Transactions on Nebulas

*For this portion of the tutorial we will pick up where we left off in the [Installation tutorial](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2001%20Installation.md).*

Nebulas provides three ways to send transactions：

1. Through http port
2. Through console
3. Through Nebulas test framework

Here is an introduction to sending a transaction in Nebulas through the three methods above and verifying whether the transaction is successful.

## Setup

You will need two wallet addresses. An address that has NAS (the sending address, i.e. `from`) and an address you want to send NAS to (the receiving address, i.e. `to`).

### The sender
For this tutorial we will use the first address in the dpos dynasty `1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c` because it is initialized with NAS when we start our seed node, NAS that we can send to another wallet because we know the passphrase for this address. You can see this address in the `conf/default/genesis.conf` file as shown below.

![key](resources/101-02-genesis.png)

### The receiver
A new wallet address that we will create to be used as the receiving address of the transfer.

Generating a new Nebulas address:
```sh
$ ./neb account new
Your new account is locked with a passphrase. Please give a passphrase. Do not forget this passphrase.
Passphrase:
Repeat passphrase:
Address: e6dea0d0769fbf71ab01f8e0d78cd59e78361a450e1f4f88
```

**Note:** when you run this command you will have a different address than `e6dea0d0769fbf71ab01f8e0d78cd59e78361a450e1f4f88`. Be sure to use your address, which we will refer to as `<your address>` going forward.

The command above will generate a json file at this location: `src/github.com/go-nebulas/keydir/`

## Start your Node

This is a two part process. First you must start a seed node, then start a normal node based on that seed node. (If you completed the Installation tutorial then you should have already done this step once before)

**Starting the Seed Node**

`$ ./neb -c conf/default/config.conf`

**Starting the normal Node**

In a separate terminal window run the following:

`$ ./neb -c conf/example/config.1a2635.conf`

After a period of time (1 to 2 minutes), the mining reward will begin being sent to the coinbase account address used in `config.1a2635.conf` which is `1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c`.

## Using `curl` to interact with the network
Nebulas provides an RPC port, allowing developers to interact with the Nebulas network via HTTP or gRPC protocols for more complex operations. Here, we introduce how to check the balance of each account through the port of the HTTP protocol. The Nebulas HTTP port's address and port is configured via the `http_listen` attribute in the configuration file. The default port is `8685`.

### Check Address Balance (accountstate)

We can check the coinbase address account balance to see the initial token distribution amount plus tokens that have been mined. When the coinbase account address has a balance, transfer transactions can be made.

**Note:** If you get any proxy errors when using curl, run the following command:

`unset https_proxy `

In the terminal make the following curl request:

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c"}'


// Response
{
  "balance":"10080640000000000000000"，
  "nonce":"0"
}
```
Great! The above shows us the balance for the address that we sent the initial token distribution to and that we are sending mining rewards to.

Now, let's check the balance of the address we created ourselves. In the terminal make the following curl request:

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"<your address>"}'


// Response
{
  "balance":"0"，
  "nonce":"0"
}
```

As expected this address has a 0 balance as we are not mining to it and we have not yet transferred any NAS to it.

## Send and Verify Transfers

Now let’s transfer some NAS from one address to another!

Before we can transfer NAS out of an address it must be unlocked. To unlock an address you need to know the passphrase for that address. Currently, an address will stay unlocked for 5 minutes. Note: you do not need to unlock the receiving address.

Let's unlock `1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c`.

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c", "passphrase":"passphrase"}'

// Response
{
  "result":true
}
```

Now that the address is unlocked we can execute a transfer.

There are two addresses needed for this request, `from` and `to`. `from` is the address we just unlocked and `to` is the address we created earlier, `<your address>`. Be sure to replace <your address> in the curl request below.


Example 1:

```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/transaction -H 'Content-Type: application/json' -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"<your address>","nonce": 1,"value": "10"}'

// Response
{
  "txhash":"17657a7a574ea767bd1618f2392d7a212b71c8ca5bd688623085c257022d07aa"
}

```

**Note:** After the `from` and `to` addresses there is a `nonce ` and `value`. The `value` number should be a string, as the value is too big to use integers. For example "10" or "12" or "100" etc.. so if you get an error:

```
{"error":"json: cannot unmarshal number into Go value of type string","code":3} – this means you forgot to add “10” quotes around the numbers
```

If your transaction was successful and you see a response that includes a `txhash`, go ahead and try running the exact same curl request again.

You should now see an error saying that you used an invalid `nonce`.

```
{
  "code": 2,
  "error": "nonce is invalid"
}
```

Each time you execute a successful transaction you have to increment the `nonce` in the subsequent transaction. So the next time you want to transfer from this same account we would have to set `"nonce": 2`. Note that `nonce` is an integer while `value` is a string.

If you have forgotten what the most recent `nonce` was you can find it by checking the Balance with the `accountstate` endpoint.

```
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c"}'

{
  "balance":"10234719999999999999990",
  "nonce":"1"
}

```
Here we can now see that the `nonce` is 1 (assuming we have only executed 1 successful transaction so far), so the next `nonce` must be 2.


## Now back to the txHash we generated above

The `txhash` value can be used to query the transaction receipt to learn about the transaction.

* Wait for about 30s before running this request because the transfer requires the miner to confirm, so there will be a delay.

Note: Use the `txhash` that you generated in your last successful transaction.

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"<your txhash goes here>"}'

// Response
{
  "hash":"93930906f21282b4cd72de8292d122806f65e6803cddd9e9e203561996237ace",
  "from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c",
  "to":"<your address>",
  "nonce":"1",
  "timestamp":"1511519091",
  "chainId":1
}
```

If you see a response that is similar to the above (contains the same keys: `hash`, `from`, `to`, `nonce`, `timestamp`, `chainId`) that means the transaction was successfully executed!

### Check the Balance of the Receiver

Now were going to check the balance of the transfer receiving account to verify whether the transfer was successful.

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"<your address>"}'

// Response
{
  "balance":"10",
  "nonce":"0"
}
```

Here you should see a balance that is the total of all the successful transfers that you executed. Here we can see that this address has 10 NAS!


### Console
Nebulas also provides a JavaScript console. The console implements [API](https://github.com/nebulasio/wiki/blob/master/rpc.md) and [Admin](https://github.com/nebulasio/wiki/blob/master/management_rpc.md)

The console provides functions such as view account, create account address, unlock account, transaction signature, send transactions, etc.
The console needs to start the node locally, or connect to the remote node with `admin.setHost()` after starting the console.
The steps to send a transaction through the console console are basically similar to calling http requests, and commands are much simpler.

##### Start the Console
```
./neb console
```

Starting this way will connect the local neb nodes that are started by default. The console implements the auto-completion method, using `TAB` to see the existing methods:

```js
> api.
api.accounts              api.getBlockByHash        api.sendRawTransaction
api.blockDump             api.getNebState           api.sendTransaction
api.call                  api.getTransactionReceipt
api.getAccountState       api.nodeInfo
```
```js
> admin.
admin.lockAccount                   admin.setHost
admin.newAccount                    admin.signTransaction
admin.sendTransactionWithPassphrase admin.unlockAccount
```

##### Check Account Addresses
```js
>  api.accounts()
{
    "Addresses": [
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
```

##### Unlock Account

```js
> admin.unlockAccount("1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c")
Unlock account "1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c"
Passphrase:
{
  "result": true
}
```

##### Send Transaction

```js
 >  api.sendTransaction ("1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c", "<your address>", "1000000000000000000", 1)
{
  "txhash":  "4cfb6461873a478f10eb35424e03ab5abad3e10bd030d2f31b3c96a02b747d22"
}
```

##### Check Transactions

```js
>  api.getTransactionReceipt ("4cfb6461873a478f10eb35424e03ab5abad3e10bd030d2f31b3c96a02b747d22")
{
  "chainId":  100,
  "from":  "1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c",
  "gas_limit":  "20000",
  "gas_price":  "1000000",
  "hash":  "4cfb6461873a478f10eb35424e03ab5abad3e10bd030d2f31b3c96a02b747d22",
  "nonce":  "1",
  "timestamp":  "1514898795",
  "to":  "b49f30d0e5c9c88cade54cd1adecf6bc2c7e0e5af646d903",
  "type":  "binary",
  "value":  "1000000000000000000"
}
```

### Using Nebtestkit
[nebtestkit](https://github.com/nebulasio/go-nebulas/tree/develop/nebtestkit) is an integrated testing framework based on [mocha](https://github.com/mochajs/mocha). With `nebtestkit`, you can launch one or more Nebulas nodes, assemble a complete private chain, or join an existing network, then make transfer transactions, deploy and invoke smart contracts, etc.
The use of `nebtestkit` instructions can be referred to [nebtestkit instructions](https://github.com/nebulasio/go-nebulas/blob/develop/nebtestkit/README.md).



### Next step: Tutorial 3:

 [Write and run a smart contract with JavaScript](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md)
