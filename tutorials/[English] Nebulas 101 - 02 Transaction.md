# Nebulas 101 - 02 Sending Transactions on Nebulas
[Youtube Tutorial](https://www.youtube.com/watch?v=_Njq8LX2r-4)

*For this portion of the tutorial we will pick up where we left off in the [Installation tutorial](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2001%20Installation.md).*

Nebulas provides three ways to send transactions：

1. Through http port
2. Through console
3. Through Nebulas test framework

Here is an introduction to sending a transaction in Nebulas through the three methods above and verifying whether the transaction is successful.

## Setup

You will need two wallet addresses. An address that has NAS (the sending address, i.e. `from`) and an address you want to send NAS to (the receiving address, i.e. `to`).

### The sender
For this tutorial we will use the coinbase address in the `conf/example/config.1a2635.conf`, which is `n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5` . It will get NAS rewards by mining blocks, Then we can send to another wallet because we know the passphrase for this address. 

### The receiver
A new wallet address that we will create to be used as the receiving address of the transfer.

Generating a new Nebulas address:
```sh
$ ./neb -c conf/default/config_local.conf account new
Your new account is locked with a passphrase. Please give a passphrase. Do not forget this passphrase.
Passphrase:
Repeat passphrase:
Address: n1SQe5d1NKHYFMKtJ5sNHPsSPVavGzW71Wy
```

**Note:** when you run this command you will have a different address than `n1SQe5d1NKHYFMKtJ5sNHPsSPVavGzW71Wy`. Be sure to use your address, which we will refer to as `your_address` going forward.

The command above will generate a json file at this location: `src/github.com/go-nebulas/keydir/`

## Start your Node

This is a two part process. First you must start a seed node, then start a normal node based on that seed node. (If you completed the Installation tutorial then you should have already done this step once before)

**Starting the Seed Node**

`$ ./neb -c conf/default/config_local.conf`

**Starting the normal Node**

In a separate terminal window run the following:

`$ ./neb -c conf/example/config.1a2635.conf`

After a period of time (1 to 2 minutes), the mining reward will begin being sent to the coinbase account address used in `config.1a2635.conf` which is `n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5`.

## Using `curl` to interact with the network
Nebulas provides an RPC port, allowing developers to interact with the Nebulas network via HTTP or gRPC protocols for more complex operations. Here, we introduce how to check the balance of each account through the port of the HTTP protocol. The Nebulas HTTP port's address and port is configured via the `http_listen` attribute in the configuration file. The default port is `8685`.

### Check Address Balance (accountstate)

We can check the coinbase address account balance to see the initial token distribution amount plus tokens that have been mined. When the coinbase account address has a balance, transfer transactions can be made.

**Note:** If you get any proxy errors when using curl, run the following command:

`unset https_proxy `

In the terminal make the following curl request:

```
/// Request
 curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5"}'
 
 // Result
 {
 	"result": {
 		"balance": "67066180000000000000",
 		"nonce": "0",
 		"type": 87
 	}
 }
```
Great! The above shows us the balance for the address that we are sending mining rewards to.

Now, let's check the balance of the address we created ourselves. In the terminal make the following curl request:

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5"}'

// Result
{
	"result": {
		"balance": "0",
		"nonce": "0",
		"type": 87
	}
}
```

As expected this address has a 0 balance as we are not mining to it and we have not yet transferred any NAS to it.

## Send and Verify Transfers

Now let’s transfer some NAS from one address to another!

Before we can transfer NAS out of an address it must be unlocked. To unlock an address you need to know the passphrase for that address. Currently, an address will stay unlocked for 5 minutes. Note: you do not need to unlock the receiving address.

Let's unlock `n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5`.

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "passphrase":"passphrase"}'

// Result
{
	"result": {
		"result": true
	}
}
```

Now that the address is unlocked we can execute a transfer.

There are two addresses needed for this request, `from` and `to`. `from` is the address we just unlocked and `to` is the address we created earlier, `<your address>`. Be sure to replace <your address> in the curl request below.


Example 1:

```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transaction -d '{"from":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","to":"your_address", "value":"10","nonce":0,"gasPrice":"1000000","gasLimit":"2000000"}'

// Result
{
    "result":{
      "txhash":"fb5204e106168549465ea38c040df0eacaa7cbd461454621867eb5abba92b4a5",
      "contract_address":""
    }
}

```

**Note:** After the `from` and `to` addresses there is `nonce `, `value`, `gasPrice` and `gasLimit`. The `value`, `gasPrice` and `gasLimit` number should be a string, as the value is too big to use integers. For example "10" or "12" or "100" etc.. so if you get an error:

```
{"error":"json: cannot unmarshal number into Go value of type string"} – this means you forgot to add “10” quotes around the numbers
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
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"8b1b0928bb7b5dea3f7b1e88a0d0896b8fa3035534ff64885d8551c37cbd294d"}'

// Result
{
	"result": {
		"hash": "8b1b0928bb7b5dea3f7b1e88a0d0896b8fa3035534ff64885d8551c37cbd294d",
		"chainId": 100,
		"from": "n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5",
		"to": "your_address",
		"value": "10",
		"nonce": "1",
		"timestamp": "1522339267",
		"type": "binary",
		"data": null,
		"gas_price": "1000000",
		"gas_limit": "2000000",
		"contract_address": "",
		"status": 1,
		"gas_used": "20000"
	}
}
```

If you see a response that is similar to the above (contains the same keys: `hash`, `from`, `to`, `nonce`, `timestamp`, `chainId`) that means the transaction was successfully executed!

### Check the Balance of the Receiver

Now were going to check the balance of the transfer receiving account to verify whether the transfer was successful.

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"your_address"}'

// Result
{
	"result": {
		"balance": "10",
		"nonce": "0",
		"type": 87
	}
}
```

Here you should see a balance that is the total of all the successful transfers that you executed. Here we can see that this address has 10 NAS!


## Console
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
api.call                    api.getBlockByHeight        api.latestIrreversibleBlock 
api.estimateGas             api.getDynasty              api.sendRawTransaction      
api.gasPrice                api.getEventsByHash         api.subscribe               
api.getAccountState         api.getNebState             
api.getBlockByHash          api.getTransactionReceipt
```
```js
> admin.
admin.accounts                      admin.nodeInfo                      admin.signHash                      
admin.getConfig                     admin.sendTransaction               admin.signTransactionWithPassphrase 
admin.lockAccount                   admin.sendTransactionWithPassphrase admin.startPprof                    
admin.newAccount                    admin.setHost                       admin.unlockAccount  
```

##### Check Account Addresses
```js
>  admin.accounts()
   {
       "result": {
           "addresses": [
               "n1FkntVUMPAsESuCAAPK711omQk19JotBjM",
               "n1JNHZJEUvfBYfjDRD14Q73FX62nJAzXkMR",
               "n1Kjom3J4KPsHKKzZ2xtt8Lc9W5pRDjeLcW",
               "n1NHcbEus81PJxybnyg4aJgHAaSLDx9Vtf8",
               "n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5",
               "your_address",
               "n1TV3sU6jyzR4rJ1D7jCAmtVGSntJagXZHC",
               "n1WwqBXVMuYC3mFCEEuFFtAXad6yxqj4as4",
               "n1Z6SbjLuAEXfhX1UJvXT6BB5osWYxVg3F3",
               "n1Zn6iyyQRhqthmCfqGBzWfip1Wx8wEvtrJ"
           ]
       }
   }
```

##### Unlock Account

```js
> admin.unlockAccount("n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5")
Unlock account n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5
Passphrase:
{
    "result": {
        "result": true
    }
}
```

##### Send Transaction

```js
> admin.sendTransaction("n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "your_address","10",2, "1000000", "200000")
{
    "result": {
        "contract_address": "",
        "txhash": "84d1ed79830566013df68809eb65e3948551ba0c2758e048ff2101aa5665703d"
    }
}
```

##### Check Transactions

```js
> api.getTransactionReceipt("84d1ed79830566013df68809eb65e3948551ba0c2758e048ff2101aa5665703d")
{
    "result": {
        "chainId": 100,
        "contract_address": "",
        "data": null,
        "from": "n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5",
        "gas_limit": "200000",
        "gas_price": "1000000",
        "gas_used": "20000",
        "hash": "84d1ed79830566013df68809eb65e3948551ba0c2758e048ff2101aa5665703d",
        "nonce": "2",
        "status": 1,
        "timestamp": "1522341302",
        "to": "your_address",
        "type": "binary",
        "value": "10"
    }
}
```
#### check account balance
```js
> api.getAccountState("your_address")
{
    "result": {
        "balance": "20",
        "nonce": "0",
        "type": 87
    }
}
```

### Using Nebtestkit
[nebtestkit](https://github.com/nebulasio/go-nebulas/tree/develop/nebtestkit) is an integrated testing framework based on [mocha](https://github.com/mochajs/mocha). With `nebtestkit`, you can launch one or more Nebulas nodes, assemble a complete private chain, or join an existing network, then make transfer transactions, deploy and invoke smart contracts, etc.
The use of `nebtestkit` instructions can be referred to [nebtestkit instructions](https://github.com/nebulasio/go-nebulas/blob/develop/nebtestkit/README.md).



### Next step: Tutorial 3:

 [Write and run a smart contract with JavaScript](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md)
