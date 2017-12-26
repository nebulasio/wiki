# Overview
Remote Procedure Calls (RPCs) provide a useful abstraction for building distributed applications and services.

Nebulas provide both [gRPC](https://grpc.io) and RESTful API, let users interact with Nebulas.

[grpc](https://github.com/grpc/grpc-go) provides a concrete implementation of the gRPC protocol, layered over HTTP/2. These libraries enable communication between clients and servers using any combination of the supported languages.

[grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) is a plugin of protoc. It reads gRPC service definition, and generates a reverse-proxy server which translates a RESTful JSON API into gRPC. we use it to map gRPC to HTTP.

## Endpoint
Default endpoints:

| API | URL | Protocol |
|-------|:------------:|:------------:|
| gRPC |  http://localhost:51521 | Protobuf|
| RESTful |http://localhost:8090 | HTTP |

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
curl -i -H 'Accept: application/json' -X POST http://localhost:8090/v1/user/blockdump -H 'Content-Type: application/json' -d '{"count":1}'
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
curl -i -H Accept:application/json -X GET http://localhost:8090/v1/user/nebstate

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
curl -i -H Accept:application/json -X GET http://localhost:8090/v1/user/nodeinfo

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
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/user/blockdump -d '{"count":2}'
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
curl -i -H Accept:application/json -X GET http://localhost:8090/v1/user/accounts

// Result
{
    "addresses":[
        "16464b93292d7c99099d4d982a05140f12779f5e299d6eb4",
        "22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09",
        "5cdadc1cfe3da0a3d067e9f1b195b90c5aebfb5afc8d43b4",
        "8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf"
    ]
}
```
***

#### GetAccountState
Return the state of the account.

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
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/user/accountstate -d '{"address":"22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09"}'

// Result
{
    "balance":"5"
}
```
***

#### SendTransaction
Verify, sign, and send the transaction.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  SendTransaction |
| HTTP | POST |  /v1/user/transaction |

###### Parameters
`from` Hex string of the sender account addresss.

`to` Hex string of the receiver account addresss.

`value` Amount of value sending with this transaction.

`nonce` Transaction nonce.

`source` contract source code.

`args` the params of contract.

`gas_price` gasPrice sending with this transaction.

`gas_limit` gasLimit sending with this transaction.

###### Returns
if general transaction:
`hash` Hex string of transaction hash.
if deploy & init smart contract:
`hash` transaction hash + '$' + address of contract

###### Example
```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8191/v1/user/transaction -H 'Content-Type: application/json' -d '{"from":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","to":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","nonce":1,"source":"\"use strict\";var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\")};BankVaultContract.prototype={init:function(){},save:function(height){var deposit=this.bankVault.get(Blockchain.transaction.from);var value=new BigNumber(Blockchain.transaction.value);if(deposit!=null&&deposit.balance.length>0){var balance=new BigNumber(deposit.balance);value=value.plus(balance)}var content={balance:value.toString(),height:Blockchain.block.height+height};this.bankVault.put(Blockchain.transaction.from,content)},takeout:function(amount){var deposit=this.bankVault.get(Blockchain.transaction.from);if(deposit==null){return 0}if(Blockchain.block.height<deposit.height){return 0}var balance=new BigNumber(deposit.balance);var value=new BigNumber(amount);if(balance.lessThan(value)){return 0}var result=Blockchain.transfer(Blockchain.transaction.from,value);if(result>0){deposit.balance=balance.dividedBy(value).toString();this.bankVault.put(Blockchain.transaction.from,deposit)}return result}};module.exports=BankVaultContract;", "args":""}'

// Result
{
   "hash": "8f5aad7e7ad59c9d9eaa351b3f41f887e49d13f37974a02c$854f488409cfcc8126d68b828c254e8926644a6efbd2f25e9439945834229e79"
}
```
***

#### Call
call a smart contract.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  Call |
| HTTP | POST |  /v1/user/call |

###### Parameters

`from` Hex string of the sender account addresss.

`to` Hex string of the receiver account addresss.

`nonce` Transaction nonce.

`function` call contract function name.

`args` the params of contract.

`gas_price` gasPrice sending with this transaction.

`gas_limit` gasLimit sending with this transaction.

###### Returns
`hash` Hex string of transaction hash.

###### HTTP Example
```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8191/v1/user/call -H 'Content-Type: application/json' -d '{"from":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","to":"4df690cad7727510f386cddb9416f601de69e48ac662c44c","nonce":2,"function":"save","args":"[0]"}'

// Result
{
   "hash": "03bd2bbeafa03e2432d774a2b52e570b0f2e615b8a6c457b0e1ae4668faf1a15"
}
```
***

#### SendRawTransaction
Submit the signed transaction.

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
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/user/rawtransaction -d '{"data":"CiDzes35MAT3o9cvG39uVucKBmGC2FwYZ3eirTdGsBw7UhIYiiCc7ALL6rfi90rZadLf6N0kQWqmVYm/GhgirDqaKxwxt6kITkbq4W52H4PwIyQJKwkiEAAAAAAAAAAAAAAAAAAAAAUoDDDz5t/QBToICgZiaW5hcnlAAUoQAAAAAAAAAAAAAAAAAAAAAFIQAAAAAAAAAAAAAAAAAAAAAFgBYkHuYSntYejaZws/TqJRjP0NDr8cAnzf28FnOzBaI+nEpwfWQYA+AZ1cM3Djkp6UckShCRfJP4u+da5r6XkiDevRAQ=="}'

// Result
{
    "hash": "f37acdf93004f7a3d72f1b7f6e56e70a066182d85c186777a2ad3746b01c3b52"
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
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/user/getBlockByHash -d '{"hash":"00000658397a90df6459b8e7e63ad3f4ce8f0a40b8803ff2f29c611b2e0190b8"}'

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
Get transactionReceipt info by tansaction hash.

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

`nonce` Transaction nonce.

`timestamp` Transaction timestamp.

`data` Transaction data.

`chainId` Transaction chain id.

`contract_address` Transaction contract address.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/user/getTransactionReceipt -d '{"hash":"f37acdf93004f7a3d72f1b7f6e56e70a066182d85c186777a2ad3746b01c3b52"}'

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
