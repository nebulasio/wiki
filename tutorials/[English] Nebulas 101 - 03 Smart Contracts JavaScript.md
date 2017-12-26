
# Nebulas 101 - 03 Write and run a smart contract with JavaScript


Today we will learn how to write, deploy, and execute smart contracts in Nebulas using JavaScript

## First things first

Before making a smart contract, first we need to review the following:

1. Install, compile and start neb application
2. Create wallet address, set coinbase, and start mining
3. Query neb node information, wallet address balance
4. Make a transfer and verify the transaction was successful

If you have doubts about the above content, they can go back to the previous chapters to review. 

Now we will learn and use the smart contracts through the following steps:

1. Write a smart contract
2. Deploy the smart contract
3. Call the smart contract to verify the contract execution works


## Write a smart contract

Like Ethereum, Nebulas implements NVM virtual machines to run smart contracts and NVM implementations use the JavaScript V8 engine, so for the current development we can write smart contracts using JavaScript and even use TypeScript.

Write a brief description of a smart contract:

1. The smart contract code must be a Prototype object;

2. Smart contract code must have a init () method, this method will only be executed once deployed;

Devlopers note: smart contract inside the private method is _ at the beginning of the method, the private method can not be externally called directly;

Below we use JavaScript to write the first smart contract: bank safes.

This smart contract needs to fulfill the following functions:

1. The user can save money to his bank vault.
2. Users can withdraw money from his bank vault.

Smart contract example:

```js
"use strict";

var BankVaultContract = function () {
   // LocalContractStorage is a built in storage to store JavaScript objects. With LocalContractStorage, you can store JavaScript objects.
   LocalContractStorage.defineMapProperty (this, "bankVault");
};

BankVaultContract.prototype = {
   init: function () {},
   save: function () {
       var deposit = this.bankVault.get (Blockchain.transaction.from);
       var value = new BigNumber (Blockchain.transaction.value);
       if (deposit! = null && deposit.balance.length> 0) {
           var balance = new BigNumber (deposit.balance);
           value = value.plus (balance);
       }
       var content = {
           balance: value.toString ()
       };
       this.bankVault.put (Blockchain.transaction.from, content);
   },
   takeout: function (amount) {
       var deposit = this.bankVault.get (Blockchain.transaction.from);
       if (deposit == null) {
           return 0;
       }
       var balance = new BigNumber (deposit.balance);
       var value = new BigNumber (amount);
       if (balance.lessThan (value)) {
           return 0;
       }
       var result = Blockchain.transfer (Blockchain.transaction.from, value);
       if (result> 0) {
           deposit.balance = balance.dividedBy (value) .toString ();
           this.bankVault.put (Blockchain.transaction.from, deposit);
       }
       return result;
   }
};

module.exports = BankVaultContract;

```

As you can see from the smart contract example above, `BankVaultContract` is a prototype object that has an init () method that satisfies what we call the most basic description for writing smart contracts.

BankVaultContract implements two other methods:
- save (): The user can save money to the bank vault by calling the save () function;
- takeout (): Users can withdraw money from bank vault by calling takeout () function;

The contract code above uses the built-in `Blockchain` object and the built-in` BigNumber () `function.

## So now let's break down the smart contract code line by line:


```js

// Check the contract balance information from the bank vault 
save ():

// The amount of money this user will deposit into the bank vault
var deposit = this.bankVault.get (Blockchain.transaction.from);

// The amount of money this user saves
var value = new BigNumber (Blockchain.transaction.value);

// Update the Vault balance information (new balance = previous balance + amount of money deposited)
if (deposit! = null && deposit.balance.length> 0) {
   var balance = new BigNumber (deposit.balance);
  value = value.plus (balance);
}
var content = {
   balance: value.toString ()
};
// Store the new balance information on the chain
this.bankVault.put (Blockchain.transaction.from, content);
```

takeout ():

```js
// balance from the safe
var deposit = this.bankVault.get (Blockchain.transaction.from);

// If the balance does not exist or has no value, return no value or 0.
if (deposit == null) {
   return 0;
}
var balance = new BigNumber (deposit.balance);
var value = new BigNumber (amount);

// If the vault balance is less than the amount fetched, return no value or 0
if (balance.lessThan (value)) {
   return 0;
}

// The value amount transferred to the user’s wallet address
var result = Blockchain.transfer (Blockchain.transaction.from, value);
if (result> 0) {

   	// Update Vault Balance (New Balance = Previous Balance - pending Amount)
   deposit.balance = balance.dividedBy (value).toString ();
   	// Store the new balance information on the chain
   this.bankVault.put (Blockchain.transaction.from, deposit);
}
```

## Deploy smart contracts
Here's how to write a smart contract in Nebulas, and now we need to deploy the smart contract to the chain.

Earlier, I introduced how users made an account transaction in Nebulas, and we used the sendTransation () interface to initiate an account transaction. Deploying a smart contract in Nebulas is actually just sending a transaction, just with different parameters.

```js
sendTransation (from, to, nonce, source, args)
```
We agree that if from and to are the same address, we assume that we are deploying a smart contract.

source: Source code for the smart contract to be deployed
args: Parameters used to deploy smart contracts

## Example of deploying a smart contract using curl:

```
// Request
curl -i -H 'Accept: application / json' -X POST http: // localhost: 8090 / v1 / transaction -H 'Content-Type: application / json' -d '{"from": "8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf", " to ":" 8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf "," nonce ": 1," source ":" \ "use strict \"; var BankVaultContract = function () {LocalContractStorage.defineMapProperty (this, \ "bankVault \")}; BankVaultContract.prototype = {init: function () {}, save: function () {var deposit = this.bankVault.get (Blockchain.transaction.from); var value = new BigNumber (Blockchain.transaction.value); if (deposit! = null && deposit var balance = new BigNumber (deposit.balance); value = value.plus (balance)} var content = {balance: value.toString ()}; this.bankVault.put (Blockchain.transaction .from, content)}, takeout: function (amount) {var deposit = this.bankVault.get (Blockchain.transaction.from); if (deposit == null) {return 0} var balance = new BigNumber (deposit.balance var value = new BigNumber (amount); if (balance.lessThan (value)) {return 0} var result = Blockchain.tr ansfer (Blockchain.transaction.from, value); if (result> 0) {deposit.balance = balance.dividedBy (value) .toString (); this.bankVault.put (Blockchain.transaction.from, deposit)} return result result }}; module.exports = BankVaultContract; "," args ":" "} '

// Result
{
    "txhash": "9ea7c434f717123ef335fc4e74671996a95b3dcef7873cbe77dc679d6b500a8c",
    "contract_address": "a111489535426e93883981f6a3de84ada9bdf2fbab714079"
}
```

The return value for deploying a smart contract is the transaction's hash address `txhash` (contract blue) and the contract's contract address` contract_address` (red).
Get the return value does not guarantee a successful deployment of the contract, because the sendTransaction () is an asynchronous process, needed to be packaged by the miner, as the previous transaction, the transfer does not arrive in real time, because its dependent on the miners packing speed, so you need to wait for a period of Time (about 1 minute), then you can verify the contract is deployed successfully by calling the contract address or calling the smart contract.

## Verify the deployment of the contract is successful
We get the contract address `contract_address` when deploying the smart contract, and we can easily check the contract's address information using the console to verify whether the contract has been deployed successfully.
! [key] (resources / 101-03-state.png)
As shown above, if we can get the contract information by the address of the contract, it means the contract has been deployed successfully.

## call smart contract
The way to call a smart contract in Nebulas is also very simple, you can call the smart contract through the rpc interface call () function.

```js
call (from, to, nonce, value, function_name, args)
```
from: user wallet address
to: smart contract address
nonce: user transaction ID, to ascending order
value: The amount of money to transfer 
function_name: function to be called
args: Parameters used by the smart contract function

Call smart contract save () function:

```js
// Request
curl -i -H 'Accept: application / json' -X POST http: // localhost: 8090 / v1 / call -H 'Content-Type: application / json' -d '{"from": "9341709022928b38dae1f9e1cfbad25611e81f736fd192c5", " to ":" a111489535426e93883981f6a3de84ada9bdf2fbab714079 "," value ":" 100 "," nonce ": 2," function ":" save "," args ":" "} '

// Result
{
  "hash": "b425abe633c4a8896aac0d02d7c75ae5eaa38318dd4c19ba3eec2f8be4bd5050"
}
```
The essence of a smart contract call is to submit a transaction, it depends on the miners to package the transaction, the miners will be successful after the transaction package call is considered successful, so the call to the smart contract is not immediately effective. We need to wait (about a minute) and than we can verify that our call was successful.
Above we call the save () function to the bank vault to deposit the amount of 100$, you need to deduct 100$ from the user's balance, so there is a transfer process, the amount of the transfer needs to pass the value field. After the contract is invoked, you only need to verify that the smart contract's address balance is 100$.
By using console we can easily check the current smart contract address amount:
! [key] (resources / 101-03-save-state.png)

Call the smart contract takeout () function:

```js
// Request
curl -i -H 'Accept: application / json' -X POST http: // localhost: 8090 / v1 / call -H 'Content-Type: application / json' -d '{"from": "9341709022928b38dae1f9e1cfbad25611e81f736fd192c5", " to ":" a111489535426e93883981f6a3de84ada9bdf2fbab714079 "," nonce ": 3," function ":" takeout "," args ":" [50] "} '

// Result
{
  "hash": "03bd2bbeafa03e2432d774a2b52e570b0f2e615b8a6c457b0e1ae4668faf1a15"
}
```
The above takeout () function is different from the save () function except that the value of 50 is taken out of the vault, and the amount withdrawn to the user is an operation inside the smart contract, so the value parameter does not need to have a value and the amount withdrawn is the operation of the Smart contract related parameters, so they are passed through args parameters.
Then we need to verify that the current smart contract address balance is not 50:
! [key] (resources / 101-03-takeout-state.png)

The picture above shows that the smart contract call result is correct, and the smart contract deployment to the call is successful.
