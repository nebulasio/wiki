# Overview
Remote Procedure Calls (RPCs) provide a useful abstraction for building distributed applications and services.

Nebulas provide both [gRPC](https://grpc.io) and RESTful API, let users interact with Nebulas.

[grpc](https://github.com/grpc/grpc-go) provides a concrete implementation of the gRPC protocol, layered over HTTP/2. These libraries enable communication between clients and servers using any combination of the supported languages.

[grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) is a plugin of protoc. It reads gRPC service definition, and generates a reverse-proxy server which translates a RESTful JSON API into gRPC. we use it to map gRPC to HTTP.

## Endpoint
Default endpoints:

| API | URL | Protocol |
|-------|:------------:|:------------:|
| gRPC |  http://localhost:8684 | Protobuf|
| RESTful |http://localhost:8685 | HTTP |

##### gRPC API
We can run the gRPC example [testing client code](https://github.com/nebulasio/go-nebulas/blob/develop/rpc/testing/client/main.go):

```
go run main.go
```

The testing client gets account state from sender address, makes a transaction from sender to receiver, and also checks the account state of receiver address.

We can see client log output like:

```
GetAccountState 8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf nonce 1 value 78
SendTransaction 8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf -> 22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09 value 2 hash:"d9258c06899412169f969807629e1c152b54a3c4033e43727f3a74855849ffa6"
GetAccountState 22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09 nonce 0 value 2
```
##### HTTP
Now we also provided HTTP to access the RPC API. The file that ends with **gw.go** is the mapping file.
Now we can access the rpc API directly from our browser, you can update the **api-http-port** and **management-http-port** in **config-seed/normal.pb.txt** to change HTTP port.

###### Example:
```
curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/blockdump -H 'Content-Type: application/json' -d '{"count":1}'
```


## RPC methods

* [GetNebState](#getnebstate)
* [NodeInfo](#nodeinfo)
* [BlockDump](#blockdump)
* [Accounts](#accounts)
* [GetAccountState](#getaccountstate)
* [SendTransaction](#sendtransaction)
* [Call](#call)
* [SendRawTransaction](#sendrawtransaction)
* [GetBlockByHash](#getblockbyhash)
* [GetTransactionReceipt](#gettransactionreceipt)
* [GetGasPrice](#getGasPrice)
* [EstimateGas](#estimateGas)
* [LatestIrreversibleBlock](latestirreversibleblock)

## RPC API Reference



#### GetNebState
Return the state of the neb.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  GetNebState |
| HTTP | GET |  /v1/user/nebstate |


###### Parameters
none

###### Returns
`chain_id` Block chain id

`tail` Current neb tail hash

`coinbase` Neb coinbase

`peer_count` Number of peers currenly connected

`is_mining` Neb mine status, minging is true ,otherwise false

`protocol_version` The current neb protocol version.

`synchronized` The peer sync status.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X GET http://localhost:8685/v1/user/nebstate

// Result
{
    "chain_id":1,
    "tail":"00001555adb8c03238169c89a90a7d3776cc0500a4d01e0187c716faaab4a21b",
    "coinbase":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf",
    "protocol_version":"/neb/1.0.0",
    "synchronized":true
}
```

***

#### NodeInfo
Return the p2p node info.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  NodeInfo |
| HTTP | GET |  /v1/user/nodeinfo |

###### Parameters
none

###### Returns
`id` the node ID.

`chain_id` the block chainID.

`version` the node version.

`peer_count` Number of peers currenly connected.

`synchronized` the node synchronized status.

`bucket_size` the node route table bucket size.

`relay_cache_size` the node relay cache size.

`stream_store_size` the node stream store size.

`stream_store_extend_size` the node stream store extend size.

`protocol_version` the network protocol version.

`RouteTable route_table` the network routeTable

```
message RouteTable {
	string id = 1;
	repeated string address = 2;
}
```

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X GET http://localhost:8685/v1/user/nodeinfo

// Result
{
    "id":"QmPyr4ZbDmwF1nWxymTktdzspcBFPL6X1v3Q5nT7PGNtUN",
    "chain_id":100,
    "version":1,
    "bucket_size":16,
    "relay_cache_size":65536,
    "stream_store_size":128,
    "stream_store_extend_size":32,
    "protocol_version":"/neb/1.0.0"
}
```
***

#### BlockDump
Return the dump info of blockchain.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  BlockDump |
| HTTP | POST |  /v1/user/blockdump |

##### Parameters
`count` the count of blocks to dump before current tail.

##### Returns
`data` block dump info.

##### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/blockdump -d '{"count":2}'

// Result
{
    "data":" {38117, hash: 000013928d115549e4e6bf058d8e84e8037d2d59fbd9f40342090ebb12612b3c, parent: 00000b4cae32da4430b05adac24146e00c3440b1ae91f66771f5c844afb316fe, stateRoot: cef1b6bc9fb3e5f3768e66d98b671e032561fb24fcfa0e858b6eb05d5f1d8f63, coinbase: 8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf} {38116, hash: 00000b4cae32da4430b05adac24146e00c3440b1ae91f66771f5c844afb316fe, parent: 0000158cf35cf2654d04b88a10a9b219bb447d9b1ed1e26a3e95078cea1ad824, stateRoot: 992bfa2b2d5ae17a374e84ce38f6e2dee51748eaa68ddd000d8fe6dd1a184501, coinbase: 8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf}"
}
```
***

#### Accounts
Return account list.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  Accounts |
| HTTP | GET |  /v1/user/accounts |

##### Parameters
none

##### Returns
`addresses` account list

##### HTTP Example
```
// Request
curl -i -H Accept:application/json -X GET http://localhost:8685/v1/user/accounts

// Result
{
    "addresses": [
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
***

#### GetAccountState
Return the state of the account. Balance and nonce of the given address will be returned.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  GetAccountState |
| HTTP | POST |  /v1/user/accountstate |

###### Parameters
`address` Hex string of the account addresss.

`block` Hex string block number, or one of "latest", "earliest" or "pending". If not specified, use "latest".

###### Returns
`balance` Current balance in unit of 1/(10^18) nas.

`nonce` Current transaction count.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/accountstate -d '{"address":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c"}'

// Result
{
    "balance":"5"
}
```
***

#### SendTransaction
Send the transaction. Parameters `from`, `to`, `value`, `nonce` are required. If `gasprice` and `gaslimit` are not provided, the transaction will use the default parameters, with only normal transaction support it. If the transaction is to send contract, delegate, or candidate, you must specify the `gaslimit`.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  SendTransaction |
| HTTP | POST |  /v1/user/transaction |

###### Parameters
`from` Hex string of the sender account addresss.

`to` Hex string of the receiver account addresss.

`value` Amount of value sending with this transaction.

`nonce` Transaction nonce.

`gas_price` gasPrice sending with this transaction.

`gas_limit` gasLimit sending with this transaction.

`contract` transaction contract object for deploy/call smart contract.

* Sub properties:
	* `source` contract source code for deploy contract.
	* `sourceType` contract source type for deploy contract. Currently support `js` and `ts`
		* `js` the contract source write with javascript.
		* `ts` the contract source write with typescript. 
	* `function` the contract call function for call contarct function.
	* `args` the params of contract. The args content is JSON string of parameters array.

Notice:

* `nonce` the value is plus 1 from the nonce value of the current from address. Current nonce can get from [GetAccountState](#getaccountstate).
* `gasPrice` and `gasLimit` default values only support normal transaction. We recommend taking them when sending a transaction.
* `contract` parameter only need for smart contract deploy and call. `source` and `sourceType` are used only when deploy, and when the contract method is invoked, no passing is required. 

###### Returns

`txhash` transaction hash.

`contract_address ` returns only for deploy contract transaction.

###### Normal Transaction Example
```js
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/transaction -H 'Content-Type: application/json' -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"333cb3ed8c417971845382ede3cf67a0a96270c05fe2f700", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}'

// Result
{
    "txhash":"cc7133643a9ae90ec9fa222871b85349ccb6f04452b835851280285ed72b008c"
}
```

###### Deploy Smart Contract Example
```js
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/transaction -H 'Content-Type: application/json' -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c", "value":"0","nonce":2,"gasPrice":"1000000","gasLimit":"2000000","contract":{
"source":"\"use strict\";var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\")};BankVaultContract.prototype={init:function(){},save:function(height){var deposit=this.bankVault.get(Blockchain.transaction.from);var value=new BigNumber(Blockchain.transaction.value);if(deposit!=null&&deposit.balance.length>0){var balance=new BigNumber(deposit.balance);value=value.plus(balance)}var content={balance:value.toString(),height:Blockchain.block.height+height};this.bankVault.put(Blockchain.transaction.from,content)},takeout:function(amount){var deposit=this.bankVault.get(Blockchain.transaction.from);if(deposit==null){return 0}if(Blockchain.block.height<deposit.height){return 0}var balance=new BigNumber(deposit.balance);var value=new BigNumber(amount);if(balance.lessThan(value)){return 0}var result=Blockchain.transfer(Blockchain.transaction.from,value);if(result>0){deposit.balance=balance.dividedBy(value).toString();this.bankVault.put(Blockchain.transaction.from,deposit)}return result}};module.exports=BankVaultContract;","sourceType":"js", "args":""}}'

// Result
{
    "txhash":"3a69e23903a74a3a56dfc2bfbae1ed51f69debd487e2a8dea58ae9506f572f73",
    "contract_address":"4702b597eebb7a368ac4adbb388e5084b508af582dadde47"
}
```
***

#### Call
Call a smart contract function. The smart contract must have been submited.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  Call |
| HTTP | POST |  /v1/user/call |

###### Parameters

The parameters of the `call` method is the same as the [SendTransaction](#sendtransaction) parameters. Special attention:

`to` Hex string of the receiver account addresss. **The value of `to` is a contract address.**

`contract` transaction contract object for call smart contract.

* Sub properties(**`source` and `sourceType` are not need**):
	* `function` the contract call function for call contarct function.
	* `args` the params of contract. The args content is JSON string of parameters array.

###### Returns
`hash` Hex string of transaction hash.

###### HTTP Example
```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/call -H 'Content-Type: application/json' -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"333cb3ed8c417971845382ede3cf67a0a96270c05fe2f700","value":"0","nonce":3,"gasPrice":"1000000","gasLimit":"2000000","contract":{"function":"save","args":"[0]"}}'

// Result
{
   "txhash": "cab27f9653cd8f3232d68fc8123d85ea508181a545b22d6eefd1f394dee7d053"
}
```
***

#### SendRawTransaction
Submit the signed transaction. The transaction signed value should be return by [SignTransaction](https://github.com/nebulasio/wiki/blob/master/management_rpc.md#signtransaction).

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  | SendRawTransaction |
| HTTP | POST |  /v1/user/rawtransaction |

###### Parameters
`data` Signed data of transaction

###### Returns
`hash` Hex string of transaction hash.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/rawtransaction -d '{"data":"CiCrHtxyyIJks2/RErvBBA862D6iwAaGQ9OK1NisSGAuTBIYGiY1R9Fnx0z0uPkWbPokTeBIHFFKRaosGhgzPLPtjEF5cYRTgu3jz2egqWJwwF/i9wAiEAAAAAAAAAAADeC2s6dkAAAoAjDd/5jSBToICgZiaW5hcnlAZEoQAAAAAAAAAAAAAAAAAA9CQFIQAAAAAAAAAAAAAAAAAABOIFgBYkGLnnvGZEDSlocc202ZRWtUlbl2RHfGNdBY5eajFiHKThfgXIwGixh17LpnZGnYHlmfiGe2zqnFHdj7G8b2XIP2AQ=="}'

// Result
{
    "txhash": "f37acdf93004f7a3d72f1b7f6e56e70a066182d85c186777a2ad3746b01c3b52"
}
```
***

#### GetBlockByHash
Get block header info by the block hash.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  | GetBlockByHash |
| HTTP | POST |  /v1/user/getBlockByHash |

###### Parameters
`hash` Hex string of transaction hash.

###### Returns
`block` block info 

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getBlockByHash -d '{"hash":"00000658397a90df6459b8e7e63ad3f4ce8f0a40b8803ff2f29c611b2e0190b8"}'

// Result

{
    "header":{
        "hash":"AAAGWDl6kN9kWbjn5jrT9M6PCkC4gD/y8pxhGy4BkLg=",
        "parent_hash":"AAAP2g0bIVupgD0D8gt5pB9SdGw7scSPll83c0XNwDM=",
        "nonce":"412557",
        "coinbase":"iiCc7ALL6rfi90rZadLf6N0kQWqmVYm/",
        "timestamp":"1511520895",
        "chain_id":1,
        "state_root":"u8+xfOhG21cyz2MM1x3M0ZNFZSfgkJYkZDuGmLRDPhA=",
        "txs_root":"EqiEdCEeJYtr4DpCtxFo9DzxEIhLJ7YLUxQ3caCq7YA="
    },
    "height":"42073"
}
```
***

#### GetTransactionReceipt
Get transactionReceipt info by tansaction hash. If the transaction     not submit or only submit and not packaged on chain, it will reurn not found error.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  | GetTransactionReceipt |
| HTTP | POST |  /v1/user/getTransactionReceipt |

###### Parameters
`hash` Hex string of transaction hash.

###### Returns
`hash` Hex string of tx hash.

`from` Hex string of the sender account addresss.

`to` Hex string of the receiver account addresss.

`value` Value of transaction.

`nonce` Transaction nonce.

`timestamp` Transaction timestamp.

`type` Transaction type.

`data` Transaction data.

`chainId` Transaction chain id.

`contract_address` Transaction contract address.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8685/v1/user/getTransactionReceipt -d '{"hash":"cc7133643a9ae90ec9fa222871b85349ccb6f04452b835851280285ed72b008c"}'

// Result
{
    "hash":"f37acdf93004f7a3d72f1b7f6e56e70a066182d85c186777a2ad3746b01c3b52",
    "from":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf",
    "to":"22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09",
    "nonce":"12",
    "timestamp":"1511519091",
    "chainId":1
}
```
***

#### GetGasPrice
Return current gasPrice.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  GetGasPrice |
| HTTP | GET |  /v1/user/getGasPrice |

##### Parameters
none

##### Returns
`gas_price` gas price. The unit is 10^-18 NAS.

##### HTTP Example
```js
// Request
curl -i -H Accept:application/json -X GET http://localhost:8685/v1/user/getGasPrice

// Result
{
    "gas_price":"1000000"
}
```
***

#### EstimateGas
Return the estimate gas of transaction.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  EstimateGas |
| HTTP | GET |  /v1/user/estimateGas |

##### Parameters
The parameters of the `EstimateGas` method is the same as the [SendTransaction](#sendtransaction) parameters.

##### Returns
`estimate_gas` the estimate gas.

##### HTTP Example
```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/estimateGas -H 'Content-Type: application/json' -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"333cb3ed8c417971845382ede3cf67a0a96270c05fe2f700", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}'

// Result
{
    "estimate_gas":"20000"
}
```
***

#### LatestIrreversibleBlock
Return the latest irreversible block.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  LatestIrreversibleBlock |
| HTTP | GET |  /v1/user/lib |

##### Parameters
none

###### Returns
`data` block info.

##### HTTP Example
```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8685/v1/user/lib -H 'Content-Type: application/json'

// Result
{
    "data":"{\"height\":305742, \"hash\":\"ec239d532249f84f158ef8ec9262e1d3d439709ebf4dd5f7c1036b26c6fe8073\", \"parentHash\":\"4d7ab29506ffb353240b6279ffaaf2b9b1681679628c78ac2033ea6bcfe77e46\", \"nonce\":0, \"timestamp\": 1516389660, \"coinbase\": \"0b9cd051a6d7129ab44b17833c63fe4abead40c3714cde6d\", \"tx\": 0}"
}
```
