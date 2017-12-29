# Nebulas 101 - 02 Sending Transactions on Nebulas

Nebulas provides three ways to send transactions：

1. Through http port
2. Through console
3. Through Nebulas test framework

Here is an introduction to sending a transaction in Nebulas through the three methods above and verifying whether the transaction is successful.

### Preparations
Before starting neb app, some preparations are needed:

1. Create two wallet addresses: 
    * One wallet address as the mining coin wallet address on Coinbase to receive mining awards and also as the sender address of the transfer transactions. 
    * Another wallet address as the receiving address of transfer transactions.

2. Modify the node's configuration file, and configure the coinbase address.

#### Preparations Detail
1. Create a mining wallet address on Coinbase.
Coinbase corresponds to the reward address of the miner's mining. The reward that the miner gets will go to this address. So before starting the node, you need to configure the Coinbase address first. We can select one address as the address of coinbase from the six addresses in the `conf/default/genesis.conf`[`consensus-> dpos-> dynasty`]. Currently, our version is six addresses as a dynasty, rotating generate blocks, so only the first six addresses can be configured as coinbase addresses. For this tutorial were be going to be using the first address.
![key](resources/101-02-genesis.png)

For Example: The generated coinbase address `1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c` needs to replace the coinbase address and miner address in the` chain` attribute in the `conf/default/seed.conf` configuration file. The example below will show the coinbase and miner having the same address - (shown below). 

![key](resources/101-02-coinbase.png)

2. Create a receiving address for a transfer.
Now let us create a transfer receiving address in the same fashion.
```sh
$ ./neb account new
Your new account is locked with a passphrase. Please give a passphrase. Do not forget this passphrase.
Passphrase:
Repeat passphrase:
Address: e6dea0d0769fbf71ab01f8e0d78cd59e78361a450e1f4f88
```

The command above will generate a json file at this location: go-nebulas/keydir/  

## Important Notice
The next step is to go to go-nebulas/conf/default/seed.conf and open up seed.conf and add the location of the json file which is: /location/of/Nebulas/go-nebulas/keydir/

Here is an example on how you seed.conf chain attribute should look: 

```
chain {
  chain_id: 100
  datadir: "seed.db"
  keydir: "/Users/macbook/Documents/go/src/github.com/go-nebulas/keydir/"
  genesis: "conf/default/genesis.conf"
  coinbase: "1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c"
  signature_ciphers: ["ECC_SECP256K1"]
  miner: "1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c"
  passphrase: "passphrase"
}

```	
Note: for this tutorial make sure your seed.conf looks similar to the example above.

### Start the Neb App
After completing all the preparations, you can start the neb app:
```
$ ./neb -c conf/default/seed.conf
```
The neb app loads the `conf/default/seed.conf` configuration file we set earlier. After starting, neb app will enter mining mode by default. After a period of time (1 to 2 minutes), mining reward will be sent to the coinbase account address we set before. The current development code mining reward is 16 NAS (which will be adjusted according to the requirements of the white paper). The average block time is about 10 seconds.

### Check Balance
Nebulas provides the RPC port, allowing developers to interact with the Nebulas via HTTP or gPRC protocols for more complex operations. Here, we introduce how to check the balance of each account through the port of HTTP protocol. The Nebulas HTTP port's address and port is configured via the `api_http_port` attribute in the configuration file. The default port is 8090.

Next, we use curl to show the RPC calls.

We can check this Coinbase address account balance's port to see this user's mining reward. When the Coinbase account address has a balance, transfer transactions can be made.
After the system starts, we can query the balance information of the account by sending an http request by curl. The following return value indicates that the balance of this address is 64:


# Step 1: In the terminal type:
Note: This command is just a precaution but I recommend using it.

```
unset https_proxy 
```

# Step 2: In the terminal type:
Note: the address below is an example address from above and will work for the rest of this tutorial.
                                  
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/user/accountstate -d '{"address":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c"}'


// Result
{
   "Balance":" 10080640000000000000000"，
   "nonce":"0"
}

```
Great! The above shows us the balance.



### Ok Send and Verify Transfers

Now let’s make a transfer

To send a transfer, follow these steps:

1. Get the account information;

```
// Request
curl -i -H Accept:application/json -X GET http://localhost:8090/v1/user/accounts

// Result

{
   "addresses":[
       "1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c",
       "e6dea0d0769fbf71ab01f8e0d78cd59e78361a450e1f4f88"
   ]
}
```
You should see addresses like the above example.
Basically this will return address or addresses that you created earlier. 
Note: when you used the “./neb account new” it created the address or addresses you see above. If you want to create more addresses use “./neb account new”

2. The terminal code below will unlock the coinbase account. Just copy the code below and paste into your terminal.
 
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/admin/account/unlock -d '{"address":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c", "passphrase":"passphrase"}'

// Result
{
   "result":true
}
```
This will unlock the sender's address which is needed for the transfers mentioned in the Preparations. 

3. Make a fund transfer to another account using an unlocked account;
Note: you see two addresses here; “from” and “to” we need to edit the “to” address with your generated address from earlier. When you “./neb account new” it generated an address we need that address now and copy and paste it into “to” Here are some examples below. 


Example 1: 

```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8090/v1/user/transaction -H 'Content-Type: application/json' -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"bf9f8c6d12797d44ea5213988aaad87e964eef32e4fab0dd","nonce": 1,"value": “10”}'

//Result

{
"txhash":"17657a7a574ea767bd1618f2392d7a212b71c8ca5bd688623085c257022d07aa"
}

```
Note: After the “from” and “to” addresses there is “nonce” and “value” the “value” number should always be inside “ ” quotes. For example “10” or “12” or “100” ect.. so if you get an error: 
{"error":"json: cannot unmarshal number into Go value of type string","code":3} – this means you forgot to add “10” quotes around the numbers


Example 2:
```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8090/v1/user/transaction -H 'Content-Type: application/json' -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"e6dea0d0769fbf71ab01f8e0d78cd59e78361a450e1f4f88","nonce": 1,"value": “10”}'

// Result
{
  "txhash": "93930906f21282b4cd72de8292d122806f65e6803cddd9e9e203561996237ace"
}
```
Transfer port: account coinbase address (`0fba`) to account user generated address using ./neb account new (`6c05`), transfer amount of “10”. 

Note: The nonce here must be the user's last nonce + 1, This means for every transaction you make the nonce needs to be increased by +1.


Let’s get the nonce value now

the nonce value of the user can be obtained by querying the account balance information with:
```
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/user/accountstate -d '{"address":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c"}'

{
"balance":"10234719999999999999990",
"nonce":"1"
}

```
This code was used earlier in this tutorial to get the balance but here were checking the nonce which should now be 1.


# Now back to the Hash value or the txHash we generated above

The port return value is the hash value of the transaction. This hash value can be used to query the transaction.

4. Wait for about 30s, and then check the transfer information (because the transfer requires the miner to confirm, so there will be a delay);

Note: Use your txHash that you generated above. The txHash you see below is mine and only used for an example. 

```
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/user/getTransactionReceipt -d '{"hash":"your TxHash Goes Here"}'

```

```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/user/getTransactionReceipt -d '{"hash":"93930906f21282b4cd72de8292d122806f65e6803cddd9e9e203561996237ace"}'

// Result
{
   "hash":"93930906f21282b4cd72de8292d122806f65e6803cddd9e9e203561996237ace",
   "from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c",
   "to":"e6dea0d0769fbf71ab01f8e0d78cd59e78361a450e1f4f88",
   "nonce":"1",
   "timestamp":"1511519091",
   "chainId":1
}
```
This code can get information from the previous transfer we did earlier. The request parameter is the hash value of the previous transfer. If you can check the transaction information, that means the transaction was successfully executed. So basically if you see “hash” “from” “to” “nonce” “timestamp” and “chainId” your doing good.


5. Now were going to check the balance of the transfer receiving account to verify whether the transfer was successful. So using the code we used earlier to check the balance we will use the same code but we will change the address to the address you generated with ./neb account new. Remember the “from” and “to” addresses? We need to plug in the “to” address now.
Hopefully you didn’t forget it. So now let’s plug it in to our code below.


Example 1
```
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/user/accountstate -d '{"address":"Your ./neb account new generated address from earlier goes here"}'

```

Example 2
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/user/accountstate -d '{"address":"e6dea0d0769fbf71ab01f8e0d78cd59e78361a450e1f4f88"}'

// Result
{
	"balance":"10",
	"nonce":"0"
}
```

### Console
Nebulas provides JavaScript console. The console implements [API](https://github.com/nebulasio/wiki/blob/master/rpc.md) and [Admin](https://github.com/nebulasio/wiki/blob/master/management_rpc.md)

The console provides functions such as view account, create account address, unlock account, transaction signature, send transactions, etc. 
The console needs to start the node locally, or connect to the remote node with `admin.setHost ()` after starting the console.
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
> api.accounts()
{
   "addresses": [
       "22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09",
       "5cdadc1cfe3da0a3d067e9f1b195b90c5aebfb5afc8d43b4",
       "83a78219edbdeee19eefc48b8d9a4a7cfa02704518b54511",
       "8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf"
   ]
}
```

##### Unlock Account

```js
> admin.unlockAccount("8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf")
Unlock account 8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf
Passphrase:
{
   "result": true
}
```

##### Send Transaction

```js
> api.sendTransaction("8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf", "22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09","5",13)
{
   "txhash": "93930906f21282b4cd72de8292d122806f65e6803cddd9e9e203561996237ace"
}
```

##### Check Transactions

```js
> api.getTransactionReceipt("93930906f21282b4cd72de8292d122806f65e6803cddd9e9e203561996237ace")
{
   "chainId": 1,
   "from": "8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf",
   "hash": "93930906f21282b4cd72de8292d122806f65e6803cddd9e9e203561996237ace",
   "nonce": "13",
   "timestamp": "1511754470",
   "to": "22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09"
}
```

### Through Nebtestkit
[nebtestkit](https://github.com/nebulasio/go-nebulas/tree/develop/nebtestkit) is an integrated testing framework based on [mocha](https://github.com/mochajs/mocha). With `nebtestkit`, you can launch one or more Nebulas nodes, assemble a complete private chain, or join an existing network, then make transfer transactions, deploy and invoke smart contracts, etc.
The use of `nebtestkit` instructions can be referred to [nebtestkit instructions](https://github.com/nebulasio/go-nebulas/blob/develop/nebtestkit/README.md), not going into detail here


