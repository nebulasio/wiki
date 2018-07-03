# Management RPC

Beside the [NEB API RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md) interface nebulas provides additional management APIs. Neb console supports both API and management interfaces. Management RPC uses the same gRPC and HTTP port, which also binds [NEB API RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md) interfaces.

Nebulas provide both [gRPC](https://grpc.io) and RESTful management APIs for users to interact with Nebulas. Our admin [proto](https://github.com/nebulasio/go-nebulas/blob/develop/rpc/pb/rpc.proto) file defines all admin APIs. **We recommend using the console access admin interfaces, or restricting the admin RPC to local access.**

Default management RPC Endpoint:

| API | URL | Protocol |
|-------|:------------:|:------------:|
| gRPC |  http://localhost:8684 | Protobuf
| RESTful |http://localhost:8685 | HTTP |


## Management RPC methods

* [NodeInfo](#nodeinfo)
* [Accounts](#accounts)
* [NewAccount](#newaccount)
* [UnLockAccount](#unlockaccount)
* [LockAccount](#lockaccount)
* [SignTransactionWithPassphrase](#signtransactionwithpassphrase)
* [SendTransactionWithPassphrase](#sendtransactionwithpassphrase)
* [SendTransaction](#sendtransaction)
* [SignHash](#signhash)
* [StartPprof](#startpprof)
* [GetConfig](#getconfig)

## Management RPC API Reference

#### NodeInfo
Return the p2p node info.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  NodeInfo |
| HTTP | GET |  /v1/admin/nodeinfo |

###### Parameters
none

###### Returns
`id` the node ID.

`chain_id` the block chainID.

`coninbase` coinbase 

`peer_count` Number of peers currenly connected.

`synchronized` the node synchronized status.

`bucket_size` the node route table bucket size.

`protocol_version` the network protocol version.

`RouteTable*[] route_table` the network routeTable

```
message RouteTable {
	string id = 1;
	repeated string address = 2;
}
```

###### HTTP Example
```
// Request
curl -i -H 'Content-Type: application/json' -X GET http://localhost:8685/v1/admin/nodeinfo

// Result
{
    "result":{
        "id":"QmP7HDFcYmJL12Ez4ZNVCKjKedfE7f48f1LAkUc3Whz4jP",
        "chain_id":100,
        "coinbase":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5",
        "peer_count":4,
        "synchronized":false,
        "bucket_size":64,
        "protocol_version":"/neb/1.0.0",
        "route_table":[
            {
                "id":"QmP7HDFcYmJL12Ez4ZNVCKjKedfE7f48f1LAkUc3Whz4jP",
                "address":[
                    "/ip4/127.0.0.1/tcp/8680",
                    "/ip4/192.168.1.206/tcp/8680"
                ]
            },
            {
                "id":"QmUxw4PZ8kMEnHD8WaSVE92dtvdnwgufM6m5DrWemdk2M7",
                "address":[
                    "/ip4/192.168.1.206/tcp/10003","/ip4/127.0.0.1/tcp/10003"
                ]
            }
        ]
    }
}
```
***

#### Accounts
Return account list.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  Accounts |
| HTTP | GET |  /v1/admin/accounts |

##### Parameters
none

##### Returns
`addresses` account list

##### HTTP Example
```
// Request
curl -i -H 'Content-Type: application/json' -X GET http://localhost:8685/v1/admin/accounts

// Result
{
    "result":{
        "addresses":[
            "n1FkntVUMPAsESuCAAPK711omQk19JotBjM",
            "n1JNHZJEUvfBYfjDRD14Q73FX62nJAzXkMR",
            "n1Kjom3J4KPsHKKzZ2xtt8Lc9W5pRDjeLcW",
            "n1NHcbEus81PJxybnyg4aJgHAaSLDx9Vtf8",
            "n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5",
            "n1TV3sU6jyzR4rJ1D7jCAmtVGSntJagXZHC",
            "n1WwqBXVMuYC3mFCEEuFFtAXad6yxqj4as4",
            "n1Z6SbjLuAEXfhX1UJvXT6BB5osWYxVg3F3",
            "n1Zn6iyyQRhqthmCfqGBzWfip1Wx8wEvtrJ"
        ]
    }
}
```
***
#### NewAccount
NewAccount create a new account with passphrase.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  NewAccount |
| HTTP | POST |  /v1/admin/account/new |


###### Parameters
`passphrase` New account passphrase.

###### Returns
`address` New Account address.

###### HTTP Example
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/new -d '{"passphrase":"passphrase"}'

// Result

{
    "result":{
        "address":"n1czGUvbQQton6KUWga4wKDLLKYDEn39mEk"
    }
}

```
***

#### UnLockAccount
UnlockAccount unlock account with passphrase. After the default unlock time, the account will be locked.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  UnLockAccount |
| HTTP | POST |  /v1/admin/account/unlock |

###### Parameters
`address` UnLock account address.

`passphrase` UnLock account passphrase.

`duration` Unlock accout duration. The unit is ns (10e-9 s).

###### Returns
`result` UnLock account result, unit is ns.

###### HTTP Example
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"n1czGUvbQQton6KUWga4wKDLLKYDEn39mEk","passphrase":"passphrase","duration":"1000000000"}'

// Result
{
    "result":{
        "result":true
    }
}

```
***

#### LockAccount
LockAccount lock account.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  LockAccount |
| HTTP | POST |  /v1/admin/account/lock |


###### Parameters
`address` Lock account address.

###### Returns
`result` Lock account result.

###### HTTP Example
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/lock -d '{"address":"n1czGUvbQQton6KUWga4wKDLLKYDEn39mEk"}'

// Result
{
    "result":{
        "result":true
    }
}

```
***

#### SignTransactionWithPassphrase
SignTransactionWithPassphrase sign transaction. The transaction's from addrees must be unlocked before sign call.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  SignTransactionWithPassphrase |
| HTTP | POST |  /v1/admin/sign |


###### Parameters
`transaction` this is the same as the [SendTransaction](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#sendtransaction) parameters.

`passphrase` from account passphrase

###### Returns
`data` Signed transaction data. 

###### sign normal transaction Example
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"transaction":{"from":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}, "passphrase":"passphrase"}'

// Result
{
    "result":{
        "data":"CiBOW15yoZ+XqQbMNr4bQdJCXrBTehJKukwjcfW5eASgtBIaGVduKnw+6lM3HBXhJEzzuvv3yNdYANelaeAaGhlXbip8PupTNxwV4SRM87r798jXWADXpWngIhAAAAAAAAAAAA3gtrOnZAAAKAEwucHt1QU6CAoGYmluYXJ5QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJB/BwhwhqUkp/gEJtE4kndoc7NdSgqD26IQqa0Hjbtg1JaszAvHZiW+XH7C+Ky9XTKRJKuTOc446646d/Sbz/nxQE="
    }
}
```
***

#### SendTransactionWithPassphrase
SendTransactionWithPassphrase send transaction with passphrase.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  SendTransactionWithPassphrase |
| HTTP | POST |  /v1/admin/transactionWithPassphrase |


###### Parameters
`transaction` transaction parameters, which is the same as the [SendTransaction](https://github.com/nebulasio/wiki/blob/master/rpc_admin.md#sendtransaction) parameters.

`passphrase` From address passphrase.

###### Returns
`txhash` transaction hash.

`contract_address ` returns only for deploy contract transaction.

###### Example
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -d '{"transaction":{"from":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"},"passphrase":"passphrase"}'

// Result
{
    "result":{
        "hash":"143eac221da8079f017bd6fd6b6a08ea0623114c93c638b94334d16aae109666",
        "contract_address":""
    }
}
```
***


#### SendTransaction
Send the transaction. Parameters `from`, `to`, `value`, `nonce`, `gasPrice` and `gasLimit` are required. If the transaction is to send contract, you must specify the `contract`.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  SendTransaction |
| HTTP | POST |  /v1/admin/transaction |

###### Parameters
`from` Hex string of the sender account addresss.

`to` Hex string of the receiver account addresss.

`value` Amount of value sending with this transaction. The unit is Wei (10^-18 NAS).

`nonce` Transaction nonce.

`gas_price` gasPrice sending with this transaction.

`gas_limit` gasLimit sending with this transaction.

`type` transaction payload type. If the type is specified, the transaction type is determined and the corresponding parameter needs to be passed in, otherwise the transaction type is determined according to the contract and binary data.  [optional]

- type enum:
	- `binary`: normal transaction with binary 
	- `deploy`: deploy smart contract 
	- `call`: call smart contract function

`contract` transaction contract object for deploy/call smart contract. [optional]

* Sub properties:
	* `source` contract source code for deploy contract.
	* `sourceType` contract source type for deploy contract. Currently support `js` and `ts`
		* `js` the contract source write with javascript.
		* `ts` the contract source write with typescript. 
	* `function` the contract call function for call contarct function.
	* `args` the params of contract. The args content is JSON string of parameters array.
	
`binary` any binary data with a length limit = 64bytes. [optional] 

Notice:

* `from = to` when deploy a contract, the `to` address must be equal to `from` address.

* `nonce` the value is **plus one**(+1) on the nonce value of the current from address. Current nonce can get from [GetAccountState](https://github.com/nebulasio/wiki/blob/master/rpc.md/#getaccountstate).
* `gasPrice` and `gasLimit` need for every transaction. We recommend taking them use [GetGasPrice](https://github.com/nebulasio/wiki/blob/master/rpc.md/#getgasprice) and [EstimateGas](https://github.com/nebulasio/wiki/blob/master/rpc.md/#estimategas).
* `contract` parameter only need for smart contract deploy and call. When a smart contract is deployed, the `source` and `sourceType` must be specified, the `args` is optional and passed in when the initialization function takes a parameter. The `function` field is used to call the contract method.

###### Returns

`txhash` transaction hash.

`contract_address ` returns only for deploying contract transaction.

###### Normal Transaction Example
```js
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transaction -d '{"from":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","to":"n1SAeQRVn33bamxN4ehWUT7JGdxipwn8b17", "value":"1000000000000000000","nonce":1000,"gasPrice":"1000000","gasLimit":"2000000"}'

// Result
{
    "result":{
      "txhash":"fb5204e106168549465ea38c040df0eacaa7cbd461454621867eb5abba92b4a5",
      "contract_address":""
    }
}
```

###### Deploy Smart Contract Example
```js
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transaction -d '{"from":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","to":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5", "value":"0","nonce":2,"gasPrice":"1000000","gasLimit":"2000000","contract":{
"source":"\"use strict\";var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\")};BankVaultContract.prototype={init:function(){},save:function(height){var deposit=this.bankVault.get(Blockchain.transaction.from);var value=new BigNumber(Blockchain.transaction.value);if(deposit!=null&&deposit.balance.length>0){var balance=new BigNumber(deposit.balance);value=value.plus(balance)}var content={balance:value.toString(),height:Blockchain.block.height+height};this.bankVault.put(Blockchain.transaction.from,content)},takeout:function(amount){var deposit=this.bankVault.get(Blockchain.transaction.from);if(deposit==null){return 0}if(Blockchain.block.height<deposit.height){return 0}var balance=new BigNumber(deposit.balance);var value=new BigNumber(amount);if(balance.lessThan(value)){return 0}var result=Blockchain.transfer(Blockchain.transaction.from,value);if(result>0){deposit.balance=balance.dividedBy(value).toString();this.bankVault.put(Blockchain.transaction.from,deposit)}return result}};module.exports=BankVaultContract;","sourceType":"js", "args":""}}'

// Result
{
    "result":{
        "txhash":"3a69e23903a74a3a56dfc2bfbae1ed51f69debd487e2a8dea58ae9506f572f73",
        "contract_address":"n21Y7arNbUfLGL59xgnA4ouinNxyvz773NW"
    }
}
```
***

#### SignHash
SignHash sign the hash of a message.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  SignHash |
| HTTP | POST |  /v1/admin/sign/hash |


###### Parameters
`address` Sign address

`hash`  A sha3256 hash of the message, base64 encoded.

`alg`  Sign algorithm
- `1` SECP256K1


###### Returns
`data` Signed transaction data. 

###### sign normal transaction Example
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign/hash -d '{"address":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5","hash":"W+rOKNqs/tlvz02ez77yIYMCOr2EubpuNh5LvmwceI0=","alg":1}'

// Result
{
    "result":{
        "data":"a7HHsLRvKTNazD1QEogY+Fre8KmBIyK+lNa4zv0Z72puFVkY9uZD6nGixGx/6s1x6Baq7etGwlDNxVvHsoGWbAA="
    }
}
```
***

#### StartPprof
StartPprof starts pprof

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  Pprof |
| HTTP | POST |  /v1/admin/pprof |


###### Parameters
`listen` the address to listen

###### Returns
`result` start pprof result 

###### Example
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/pprof -d '{"listen":"0.0.0.0:1234"}'

// Result
{
    "result":{
        "result":true
    }
}
```
***

#### GetConfig
GetConfig return the config current neb is using 

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  GetConfig |
| HTTP | GET |  /v1/admin/getConfig |


###### Parameters
none

###### Returns
`config` neb config 

###### Example
```
// Request
curl -i -H 'Content-Type: application/json' -X GET http://localhost:8685/v1/admin/getConfig

// Result
{
    "result":{
        "config":{
            "network":{
                "seed":[],
                "listen":["0.0.0.0:8680"],
                "private_key":"conf/network/ed25519key",
                "network_id":1
            },
            "chain":{
                "chain_id":100,
                "genesis":"conf/default/genesis.conf",
                "datadir":"data.db",
                "keydir":"keydir",
                "start_mine":true,
                "coinbase":"n1QZMXSZtW7BUerroSms4axNfyBGyFGkrh5",
                "miner":"n1Zn6iyyQRhqthmCfqGBzWfip1Wx8wEvtrJ",
                "passphrase":"",
                "enable_remote_sign_server":false,
                "remote_sign_server":"",
                "gas_price":"",
                "gas_limit":"",
                "signature_ciphers":["ECC_SECP256K1"]
            },
            "rpc":{
                "rpc_listen":["127.0.0.1:8684"],
                "http_listen":["127.0.0.1:8685"],
                "http_module":["api","admin"],
                "connection_limits":0,
                "http_limits":0,
                "http_cors":[]
            },
            "stats":{
                "enable_metrics":false,
                "reporting_module":[],
                "influxdb":{
                    "host":"http://localhost:8086",
                    "port":0,
                    "db":"nebulas",
                    "user":"admin",
                    "password":"admin"
                },
                "metrics_tags":[]
            },
            "misc":null,
            "app":{
                "log_level":"debug",
                "log_file":"logs",
                "log_age":0,
                "enable_crash_report":true,
                "crash_report_url":"https://crashreport.nebulas.io",
                "pprof":{
                    "http_listen":"0.0.0.0:8888",
                    "cpuprofile":"",
                    "memprofile":""
                },
                "version":"0.7.0"
            }
        }
    }
}
```
***
