# Management RPC

Beside the [NEB API RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md) interface nebulas provides additional management APIs. Neb console supports both API and management interfaces. Management RPC has separate gRPC and HTTP port, which also binds [NEB API RPC](https://github.com/nebulasio/wiki/blob/master/rpc.md) interfaces.

Nebulas provide both [gRPC](https://grpc.io) and RESTful management APIs, let users interact with Nebulas. Our admin [proto](https://github.com/nebulasio/go-nebulas/blob/develop/rpc/pb/rpc.proto) file defines all admin APIs. **We recommend using the console access admin interfaces, or restricting the port of the admin RPC to local access.**

Default management RPC Endpoint:

| API | URL | Protocol |
|-------|:------------:|:------------:|
| gRPC |  http://localhost:8684 | Protobuf
| RESTful |http://localhost:8685 | HTTP |


## Management RPC methods

* [NewAccount](#newaccount)
* [UnlockAccount](#unlockaccount)
* [LockAccount](#lockaccount)
* [SignTransaction](#signtransaction)
* [SendTransactionWithPassphrase](#sendtransactionwithpassphrase)
* [StartMining](#startmining)
* [StopMining](#stopmining)

## Management RPC API Reference

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
    "address":"71e616cb8bcd1270eee26f54247bd45dd0e5e0032bc04279"
}

```
***

#### UnlockAccount
UnlockAccount unlock account with passphrase. After the default unlock time, the account will be locked.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  UnlockAccount |
| HTTP | POST |  /v1/admin/account/unlock |


###### Parameters
`address` Unlock account address.

`passphrase` Unlock account passphrase.

###### Returns
`result` Unlock account result.

###### HTTP Example
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/unlock -d '{"address":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf", "passphrase":"passphrase"}'

// Result
{
    "result":true
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
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/account/lock -d '{"address":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf"}'

// Result
{
    "result":true
}

```
***

#### SignTransaction
SignTransaction sign transaction. The transaction's from addrees must be unlock before sign call.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  SignTransaction |
| HTTP | POST |  /v1/admin/sign |


###### Parameters
The parameters of the `SignTransaction` method is the same as the [SendTransaction](#sendtransaction) parameters.

###### Returns
`data` Signed transaction data. 

###### sign normal transaction Example
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"333cb3ed8c417971845382ede3cf67a0a96270c05fe2f700", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"}'

// Result
{
    "data":"CiCBzwh67ctZ8bsS/C93EYTtWhkYrCfKuUlg2fRtewfcYxIYGiY1R9Fnx0z0uPkWbPokTeBIHFFKRaosGhgzPLPtjEF5cYRTgu3jz2egqWJwwF/i9wAiEAAAAAAAAAAADeC2s6dkAAAoATDUhrHTBToICgZiaW5hcnlAZEoQAAAAAAAAAAAAAAAAAA9CQFIQAAAAAAAAAAAAAAAAAB6EgFgBYkH4U+V9Qp9FNdSHfUd/NAwWEhjTu2bevaEaNyqRx1p1QFS322nnJRFDRePHDx56pUADk2H4l8ZRJyhF31woH3V2AQ=="
}
```
***

###### sign deploy contract Example
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/sign -d '{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c", "value":"0","nonce":2,"gasPrice":"1000000","gasLimit":"2000000","contract":{
"source":"\"use strict\";var BankVaultContract=function(){LocalContractStorage.defineMapProperty(this,\"bankVault\")};BankVaultContract.prototype={init:function(){},save:function(height){var deposit=this.bankVault.get(Blockchain.transaction.from);var value=new BigNumber(Blockchain.transaction.value);if(deposit!=null&&deposit.balance.length>0){var balance=new BigNumber(deposit.balance);value=value.plus(balance)}var content={balance:value.toString(),height:Blockchain.block.height+height};this.bankVault.put(Blockchain.transaction.from,content)},takeout:function(amount){var deposit=this.bankVault.get(Blockchain.transaction.from);if(deposit==null){return 0}if(Blockchain.block.height<deposit.height){return 0}var balance=new BigNumber(deposit.balance);var value=new BigNumber(amount);if(balance.lessThan(value)){return 0}var result=Blockchain.transfer(Blockchain.transaction.from,value);if(result>0){deposit.balance=balance.dividedBy(value).toString();this.bankVault.put(Blockchain.transaction.from,deposit)}return result}};module.exports=BankVaultContract;","sourceType":"js", "args":""}}'

// Result
{
    "data":"CiD6OXQajree4K7zTPN+Vo1+QhaYKG3nECOBtlJRqT4IqRIYGiY1R9Fnx0z0uPkWbPokTeBIHFFKRaosGhgaJjVH0WfHTPS4+RZs+iRN4EgcUUpFqiwiEAAAAAAAAAAAAAAAAAAAAAAoAjCdh7HTBTrnCAoGZGVwbG95EtwIeyJTb3VyY2VUeXBlIjoianMiLCJTb3VyY2UiOiJcInVzZSBzdHJpY3RcIjt2YXIgQmFua1ZhdWx0Q29udHJhY3Q9ZnVuY3Rpb24oKXtMb2NhbENvbnRyYWN0U3RvcmFnZS5kZWZpbmVNYXBQcm9wZXJ0eSh0aGlzLFwiYmFua1ZhdWx0XCIpfTtCYW5rVmF1bHRDb250cmFjdC5wcm90b3R5cGU9e2luaXQ6ZnVuY3Rpb24oKXt9LHNhdmU6ZnVuY3Rpb24oaGVpZ2h0KXt2YXIgZGVwb3NpdD10aGlzLmJhbmtWYXVsdC5nZXQoQmxvY2tjaGFpbi50cmFuc2FjdGlvbi5mcm9tKTt2YXIgdmFsdWU9bmV3IEJpZ051bWJlcihCbG9ja2NoYWluLnRyYW5zYWN0aW9uLnZhbHVlKTtpZihkZXBvc2l0IT1udWxsXHUwMDI2XHUwMDI2ZGVwb3NpdC5iYWxhbmNlLmxlbmd0aFx1MDAzZTApe3ZhciBiYWxhbmNlPW5ldyBCaWdOdW1iZXIoZGVwb3NpdC5iYWxhbmNlKTt2YWx1ZT12YWx1ZS5wbHVzKGJhbGFuY2UpfXZhciBjb250ZW50PXtiYWxhbmNlOnZhbHVlLnRvU3RyaW5nKCksaGVpZ2h0OkJsb2NrY2hhaW4uYmxvY2suaGVpZ2h0K2hlaWdodH07dGhpcy5iYW5rVmF1bHQucHV0KEJsb2NrY2hhaW4udHJhbnNhY3Rpb24uZnJvbSxjb250ZW50KX0sdGFrZW91dDpmdW5jdGlvbihhbW91bnQpe3ZhciBkZXBvc2l0PXRoaXMuYmFua1ZhdWx0LmdldChCbG9ja2NoYWluLnRyYW5zYWN0aW9uLmZyb20pO2lmKGRlcG9zaXQ9PW51bGwpe3JldHVybiAwfWlmKEJsb2NrY2hhaW4uYmxvY2suaGVpZ2h0XHUwMDNjZGVwb3NpdC5oZWlnaHQpe3JldHVybiAwfXZhciBiYWxhbmNlPW5ldyBCaWdOdW1iZXIoZGVwb3NpdC5iYWxhbmNlKTt2YXIgdmFsdWU9bmV3IEJpZ051bWJlcihhbW91bnQpO2lmKGJhbGFuY2UubGVzc1RoYW4odmFsdWUpKXtyZXR1cm4gMH12YXIgcmVzdWx0PUJsb2NrY2hhaW4udHJhbnNmZXIoQmxvY2tjaGFpbi50cmFuc2FjdGlvbi5mcm9tLHZhbHVlKTtpZihyZXN1bHRcdTAwM2UwKXtkZXBvc2l0LmJhbGFuY2U9YmFsYW5jZS5kaXZpZGVkQnkodmFsdWUpLnRvU3RyaW5nKCk7dGhpcy5iYW5rVmF1bHQucHV0KEJsb2NrY2hhaW4udHJhbnNhY3Rpb24uZnJvbSxkZXBvc2l0KX1yZXR1cm4gcmVzdWx0fX07bW9kdWxlLmV4cG9ydHM9QmFua1ZhdWx0Q29udHJhY3Q7IiwiQXJncyI6IiJ9QGRKEAAAAAAAAAAAAAAAAAAPQkBSEAAAAAAAAAAAAAAAAAAehIBYAWJBmzfKx7Z3bpXEqWQ3G5rveVUHnglr3KVFRqemqfPdqpwtVimEylObGiCvh6aZdCtih+h2EbuBr2c6u7sYQwKdagA="
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
`transaction` transaction parameters, which is the same as the [SendTransaction](#sendtransaction) parameters.

`passphrase` From address passphrase.

###### Returns
`txhash` transaction hash.

`contract_address ` returns only for deploy contract transaction.

###### Example
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/transactionWithPassphrase -d '{"transaction":{"from":"1a263547d167c74cf4b8f9166cfa244de0481c514a45aa2c","to":"333cb3ed8c417971845382ede3cf67a0a96270c05fe2f700", "value":"1000000000000000000","nonce":1,"gasPrice":"1000000","gasLimit":"2000000"},"passphrase":"passphrase"}'

// Result
{
    "hash":"00a631ebf5e1b02e9d8ad3f714d8b4ae64aca211e65323d41469d233270c9dc5"
}
```
***


#### StartMining
StartMining start consensus mining.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  StartMining |
| HTTP | POST |  /v1/admin/startMining |


###### Parameters
`passphrase` miner's passphrase. miner address configed in config file.

###### Returns
`result` result value true / throw error.

###### Example
```
// Request
curl -i -H 'Content-Type: application/json' -X POST http://localhost:8685/v1/admin/startMining -d '{"passphrase": "passphrase"}'

// Result
{
    "result": true
}
```
***

#### StopMining
StopMining stop consensus mining.

| Protocol | Method | API |
|----------|--------|-----|
| gRpc |  |  StopMining |
| HTTP | GET |  /v1/admin/stopMining |


###### Parameters
none.

###### Returns
`result` result value true / throw error.

###### Example
```
// Request
curl -i -H 'Content-Type: application/json' -X GET http://localhost:8685/v1/admin/stopMining

// Result
{
    "result": true
}
```
***