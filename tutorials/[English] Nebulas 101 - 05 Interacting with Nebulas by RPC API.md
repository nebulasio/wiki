
# Nebulas 101 - 05 Interacting with Nebulas by RPC API

Nebulas node, after starting, can be accessed remotely through RPC. Nebulas provides APIs to get the node information, account balances, send transactions and deploy and call smart contracts.

The remote access to Nebulas is done by [gRPC](https://grpc.io), via proxy ([grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway)) or HTTP. HTTP access is a RESTful API, with the same parameters as the gRPC API.

## gRPC Access
gRPC is a high-performance, general, and open-source RPC framework developed by Google primarily for mobile applications and designed based on the HTTP/2 protocol standard, based on [ProtoBuf](https://github.com/google/protobuf) serialization protocol, and it supports many development languages.

The gRPC API for Nebulas is developed using Golang and provides a client written in Golang [demo](https://github.com/nebulasio/go-nebulas/blob/develop/rpc/testing/client/main.go). Here we mainly introduce the use of Golang to implement gRPC API. (gRPC supports multiple languages, and gRPC can be accessed in other languages.)

gRPC defines services using ProtoBuf, which is defined in the official code in [/rpc/pb](https://github.com/nebulasio/go-nebulas/tree/master/rpc/pb):

```
// Defines APIs for node and account address information access, sending transaction, etc. for external users
api_rpc.proto

// Management API, which defines node management functions such as creating an account address, unlocking addresses, etc.
management_rpc.proto
```
Use `make` in `rpc/pb` folder:

```
cd rpc/pb
make
```

Generate the corresponding Golang version of the gRPC API code. The official Golang code has been generated, re-generation is not needed. The gRPC port can be modified in the configuration file (eg: `config-seed.pb.txt`). Configuration items in the port configuration items:

```
# Service configuration of interactions between the user and nodes. When multiple ports are started on the same machine, make sure to modify the port to prevent override
rpc {
  # gRPC API for users
  api_port: 51510
  # gRPC Management API, for Admins
  management_port: 52520
  # HTTP API, for users
  api_http_port: 8090
  # HTTP Management API, for Admins
  management_http_port: 8191
}
```
The default configuration ports are the above `API: 51510` and `management: 52520`. The API port provides the API interface service, the Management port provides the management interface service, and managmenr can also access the API interface service.

Golang gRPC access code:

```go
// gRPC server connection address configuration
addr := fmt.Sprintf("127.0.0.1:%d", uint32(52520))
conn, err := grpc.Dial(addr, grpc.WithInsecure())
if err != nil {
	log.Warn("rpc.Dial() failed: ", err)
}
defer conn.Close()

// API interface access, or node status information
api := rpcpb.NewAPIServiceClient(conn)
resp, err := ac.GetNebState(context.Background(), &rpcpb.GetNebStateRequest{})
if err != nil {
	log.Println("GetNebState", "failed", err)
} else {
	//tail := r.GetTail()
	log.Println("GetNebState tail", resp)
}

//Management interface access, locking account address
management := rpcpb.NewManagementServiceClient(conn)
from := "8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf"
resp, err = management.LockAccount(context.Background(), &rpcpb.LockAccountRequest{Address: from})
if err != nil {
	log.Println("LockAccount", from, "failed", err)
} else {
	log.Println("LockAccount", from, "result", resp)
}
```
The API and management interfaces are defined in the Golang interface files generated from the proto file:
`api_rpc.pb.go` and `management_rpc.pb.go`.

## HTTP Access
Nebulas' HTTP access uses RESTful-like API. Using the HTTP interface, you can easily access node information, account address balance, send transactions, and deploy and call smart contracts. Currently Nebulas also provides two ports, respectively, for ordinary users and administrators to access. The default port setting is at the port where gRPC was previously set.

Official default ports：

* 8090: The default API port, to access the [RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md) interface for getting node information, sending transactions, etc.; can be accessed by external users.
* 8191: The default management port, to access [Management RPC](https://github.com/nebulasio/wiki/blob/master/management_rpc.md) interface, for creating accounts, transaction signatures, etc.; generally not accessible to external users.

Examples of using the HTTP access interface:

##### Get Node Information
Return node information.

| Protocol | Method | API |
|----------|--------|-----|
| HTTP | GET |  /v1/node/info |

###### Parameters
none

###### Returns
`id` node ID.

`chain_id` Blockchain ID.

`version` node version.

`peer_count` current node count.

`synchronized` node sychronization status.

`bucket_size` number of node saved in node route table.

`relay_cache_size` relay cache size.

`stream_store_size` node stream store size.

`stream_store_extend_size` node stream store extend size.

`protocol_version` network protocol version.

`RouteTable route_table` route table.

```
message RouteTable {
	string id = 1;
	repeated string address = 2;
}
```

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X GET http://localhost:8090/v1/node/info

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
##### Account List
Returns the list of accounts that the nodes exist on.

| Protocol | Method | API |
|----------|--------|-----|
| HTTP | GET |  /v1/accounts |

##### Parameters
None

##### Returns
`addresses` account list

##### HTTP Example
```
// Request
curl -i -H Accept:application/json -X GET http://localhost:8090/v1/accounts

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
#### Ger Account Information
Returns account information, including account address balance and current transaction nonce.

| Protocol | Method | API |
|----------|--------|-----|
| HTTP | POST |  /v1/account/state |

###### Parameters
`address` address hash.

###### Returns
`balance` current balance. Unit:： 1/(10^18) NAS.

`nonce` current transaction nonce.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/account/state -d '{"address":"22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09"}'

// Result
{
    "balance":"5",
    "nonce": "0"
}
```
#### Unlock Account
Use passphrase to unlock account.

| Protocol | Method | API |
|----------|--------|-----|
| HTTP | POST |  /v1/account/unlock |


###### Parameters
`address` account address hash.

`passphrase` account passphrase.

###### Returns
`result` unlock results.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8191/v1/account/unlock -d '{"address":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf", "passphrase":"passphrase"}'

// Result
{
    "result":true
}

```
#### Send Transactions
Send Transactions, contract submission port

| Protocol | Method | API |
|----------|--------|-----|
| HTTP | POST |  /v1/transaction |

###### Parameters
`from` Sender account address hash.

`to` Receiver account address hash.

`value` Value. Unit: 1/(10^18) NAS.

`nonce` Transaction nonce.

`source` Contract code.

`args` Contract arguments.

`gas_price` Transaction gas price.

`gas_limit` Transaction gas limit.

###### Returns
`txhash` Transaction hash;

If deploying contract, would also return contract address information:

`contract_address` contract address information；

###### Example
```
// Request
curl -i -H 'Accept: application/json' -X POST http://localhost:8090/v1/transaction -H 'Content-Type: application/json' -d '{"from":"83a78219edbdeee19eefc48b8d9a4a7cfa02704518b54511","to":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","nonce":1,"source":"\"use strict\";var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\")};BankVaultContract.prototype={init:function(){},save:function(height){var deposit=this.bankVault.get(Blockchain.transaction.from);var value=new BigNumber(Blockchain.transaction.value);if(deposit!=null&&deposit.balance.length>0){var balance=new BigNumber(deposit.balance);value=value.plus(balance)}var content={balance:value.toString(),height:Blockchain.block.height+height};this.bankVault.put(Blockchain.transaction.from,content)},takeout:function(amount){var deposit=this.bankVault.get(Blockchain.transaction.from);if(deposit==null){return 0}if(Blockchain.block.height<deposit.height){return 0}var balance=new BigNumber(deposit.balance);var value=new BigNumber(amount);if(balance.lessThan(value)){return 0}var result=Blockchain.transfer(Blockchain.transaction.from,value);if(result>0){deposit.balance=balance.dividedBy(value).toString();this.bankVault.put(Blockchain.transaction.from,deposit)}return result}};module.exports=BankVaultContract;", "args":""}'

// Result
{
    "txhash":"896f813a4560732793cdceb9133682277de20df502a35f7af2a845e6606c450b",
    "contract_address":"695392515cfb8fbfc1f50b7eec02e79a1fbb31e710178b0f"
}
```
#### Get Transaction Information
Return transaction information using hash.

| Protocol | Method | API |
|----------|--------|-----|
| HTTP | POST |  /v1/getTransactionReceipt |

###### Parameters
`hash` 交易哈希. Transaction hash.

###### Returns

`hash` 交易哈希. Transaction hash.

`from` Sender account address hash.

`to` Receiver account address hash.

`nonce` Transaction nonce.

`timestamp` Transaction timestamp.

`data` Transaction data.

`chainId` Transaction ID.

`contract_address` Contract address.

###### HTTP Example
```
// Request
curl -i -H Accept:application/json -X POST http://localhost:8090/v1/getTransactionReceipt -d '{"hash":"f37acdf93004f7a3d72f1b7f6e56e70a066182d85c186777a2ad3746b01c3b52"}'

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


For detailed API and parameter documentation, please refer to the official documentations of [RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md) and [Management RPC](https://github.com/nebulasio/wiki/blob/master/management_rpc.md).
