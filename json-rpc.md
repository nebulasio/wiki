# JSON RPC
[JSON](http://www.json.org/)(JavaScript Object Notation) is a lightweight data-interchange format. It is easy for humans to read and write. It is easy for machines to parse and generate. JSON is a text format that is completely language independent but uses conventions that are familiar to programmers of the C-family of languages, including C, C++, C#, Java, JavaScript, Perl, Python, and many others. These properties make JSON an ideal data-interchange language.

[JSON-RPC](http://www.jsonrpc.org/specification) is a stateless, light-weight remote procedure call (RPC) protocol. Primarily this specification defines several data structures and the rules around their processing. It is transport agnostic in that the concepts can be used within the same process, over sockets, over http, or in many various message passing environments.

[grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway) is a plugin of protoc. It reads gRPC service definition, and generates a reverse-proxy server which translates a RESTful JSON API into gRPC. we use it to map gRPC to HTTP.

## JSON-RPC Endpoint
Default JSON-RPC endpoints:
```
http://localhost:8080
```
you can update the `gateway_port` in config.pb.txt to change HTTP default port.

## JSON-RPC methods

* [get_neb_state](#get_neb_state)
* [accounts](#accounts)
* [get_account_state](#get_account_state)
* [block_dump](#block_dump)
* [send_transaction](#send_transaction)
* [call](#call)

## JSON RPC API Reference

***

#### get_neb_state
get the state of the neb

##### Parameters
none

##### Returns
`chainID` block chain id
`tail` current neb tail hash
`coinbase` neb coinbase

##### Example
```js
// Request
curl -i -H Accept:application/json -X GET http://localhost:8080/v1/neb/state

// Result
{
    "chainID":1,
    "tail":"000000e8abc390e8b8288bc9f91105e38e55d41c4dc6b6e53e2f1f06e9a7aedf",
    "coinbase":"8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf"
}
```

***

***

#### accounts
get the accounts info

##### Parameters
none

##### Returns
`addresses` account list

##### Example
```js
// Request
curl -i -H Accept:application/json -X GET http://localhost:8080/v1/accounts

// Result
{
   "addresses": [
      "22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09",
      "5cdadc1cfe3da0a3d067e9f1b195b90c5aebfb5afc8d43b4",
      "8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf"
   ]
}
```
***

***

#### get_account_state
get the account state

##### Parameters
`address` Hex string of the account address
`block` Hex string block number, or one of "latest", "earliest" or "pending". If not specified, use "latest".

##### Returns
`balance` Current balance in unit of 1/(10^18) nas.
`nonce` Current transaction count.

##### Example
```js
// Request
curl -i -H Accept:application/json -X POST http://localhost:8080/v1/account/state -H Content-Type: application/json -d '{"address":"22ac3a9a2b1c31b7a9084e46eae16e761f83f02324092b09"}'

// Result
{
   "balance": "AAAAAAAAAAAAAAAAAAAACA=="
}
```

***

***
#### block_dump
get the block dump info.

##### Parameters
`count` the count of blocks to dump before current tail.

##### Returns
`data` block dump info.

##### Example
```js
// Request
curl -i -H Accept:application/json -X POST http://localhost:8080/v1/block/dump -H Content-Type: application/json -d '{"count":2}'
// Result
{
   "data": " {1343, hash: 0000005eedcaa68ac5f307651c49e87b4f80f26d5cefdf1aded07f89908843d4, parent: 000000ad4a11f326309619823463eb397dc51bb54991f131042855034824b92b, stateRoot: c4e4952fe8afde076d38e3de5c0e5ce395849425a4ebff87344e4f01427617dc, coinbase: 8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf} {1342, hash: 000000ad4a11f326309619823463eb397dc51bb54991f131042855034824b92b, parent: 0000002bfc9cdb64745d413375f521fdc5a59cf91fd888e983852859b0517114, stateRoot: 68bb2ca8f6fea95ab2dde2bd79590d051cf622543fe30c235595ef221a75bf3a, coinbase: 8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf}"
}
```
***

***
#### send_transaction
send transaction or deploy a smart contract.

##### Parameters
`from` Hex string of the sender account addresss.
`to` Hex string of the receiver account addresss.
`value` Amount of value sending with this transaction.
`nonce` Transaction nonce.
`source` contract source code.
`args` the params of contract.

##### Returns
if general transaction:
`hash` Hex string of transaction hash.
if deploy & init smart contract:
`hash` transaction hash + '$' + address of contract

##### Example
```js
// Request
curl -i -H Accept:application/json -X POST http://localhost:8080/v1/transaction -H Content-Type: application/json -d '{"from":"0x8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","to":"0x8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","nonce":1,"source":"'use strict';var SampleContract = function () {LocalContractStorage.defineProperties(this, {name: null,count: null});LocalContractStorage.defineMapProperty(this, \"allocation\");};SampleContract.prototype = {init: function (name, count, allocation) {this.name = name;this.count = count;allocation.forEach(function (item) {this.allocation.put(item.name, item.count);}, this);},dump: function () {console.log('dump: this.name = ' + this.name);console.log('dump: this.count = ' + this.count);},verify: function (expectedName, expectedCount, expectedAllocation) {if (!Object.is(this.name, expectedName)) {throw new Error(\"name is not the same, expecting \" + expectedName + \", actual is \" + this.name + \".\");}if (!Object.is(this.count, expectedCount)) {throw new Error(\"count is not the same, expecting \" + expectedCount + \", actual is \" + this.count + \".\");}expectedAllocation.forEach(function (expectedItem) {var count = this.allocation.get(expectedItem.name);if (!Object.is(count, expectedItem.count)) {throw new Error(\"count of \" + expectedItem.name + \" is not the same, expecting \" + expectedItem.count + \", actual is \" + count + \".\");}}, this);}};module.exports = SampleContract;", "args":"[\"TEST001\", 123,[{\"name\":\"robin\",\"count\":2},{\"name\":\"roy\",\"count\":3},{\"name\":\"leon\",\"count\":4}]]"}'

// Result
{
   "hash": "8f5aad7e7ad59c9d9eaa351b3f41f887e49d13f37974a02c$854f488409cfcc8126d68b828c254e8926644a6efbd2f25e9439945834229e79"
}
```
***

***
#### call
call a smart contract.

##### Parameters
`from` Hex string of the sender account addresss.
`to` Hex string of the receiver account addresss.
`nonce` Transaction nonce.
`function` call contract function name.
`args` the params of contract.

##### Returns
`hash` Hex string of transaction hash.

##### Example
```js
// Request
curl -i -H Accept:application/json -X POST http://localhost:8080/v1/call -H Content-Type: application/json -d '{"from":"0x8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf","to":"8f5aad7e7ad59c9d9eaa351b3f41f887e49d13f37974a02c", "nonce":2,"function":"dump"}'
// Result

{
   "hash": "03bd2bbeafa03e2432d774a2b52e570b0f2e615b8a6c457b0e1ae4668faf1a15"
}
```
***