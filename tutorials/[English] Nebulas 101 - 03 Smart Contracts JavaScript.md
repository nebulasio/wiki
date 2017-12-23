# Nebulas 101 - 03 Write and run a smart contract using JavaScript

Today we will learn how to write, deploy, and execute smart contracts in Nebulas using JavaScript

## First things first

Before making a smart contract, first we need to review the following:

1. Install, compile and start neb application
2. Create a wallet address, set coinbase, and start mining
3. Query neb node information, wallet address balance
4. Make a transfer and verify the transaction was successful

If you have doubts about the above content, they can go back to the previous chapters to review. 

Now we will learn and use the smart contracts through the following steps:

1. Write a smart contract
2. Deploy the smart contract
3. Call the smart contract to verify the contract execution results


## Write a smart contract

Like Ethereum, Nebulas implements NVM virtual machines to run smart contracts and NVM implementations use the JavaScript V8 engine, so for the current development we can write smart contracts using JavaScript and even use TypeScript.

Write a brief description of a smart contract:

1. The smart contract code must be a Prototype object;
2. Smart contract code must have a init () method, this method will only be executed once the contract is deployed;

smart contracts inside the private method is _ at the beginning of the method, the private method can not be external direct call;


Below we use JavaScript to write our first smart contract: bank vault.
This smart contract needs to fulfill the following functions:

1. The user can save money to his bank vault.
2. Users can withdraw money from his bank vault.

Smart contract example:

```js

"use strict";

var BankVaultContract = function() {

   // LocalContractStorage is a built in storage to store JavaScript objects
   LocalContractStorage.defineMapProperty(this, "bankVault");
};




BankVaultContract.prototype = {
    init:function() {},


            // Check the contract balance information from the bank vault
    save:function() {

            // The amount of money this user will deposit into the bank vault
        var deposit = this.bankVault.get(Blockchain.transaction.from);

            // Update the Vault balance information (new balance = previous balance + amount of money deposited)
        var value = new BigNumber(Blockchain.transaction.value);
        if (deposit != null && deposit.balance.length > 0) {
            var balance = new BigNumber(deposit.balance);
            value = value.plus(balance);
        }

            // Store the new balance information on the chain
        this.bankVault.put (Blockchain.transaction.from, content);
        var content = {
            balance:value.toString()
        };
        this.bankVault.put(Blockchain.transaction.from, content);
    },

    takeout:function(amount) {

                // get the balance for the bank vault
        var deposit = this.bankVault.get(Blockchain.transaction.from);

                // If the balance does not exist or has no value, return no value or 0.
        if (deposit == null) {
            return 0;
        }
        var balance = new BigNumber(deposit.balance);
        var value = new BigNumber(amount);

                // If the vault balance is less than the amount fetched, return no value or 0
        if (balance.lessThan(value)) {
            return 0;
        }

                // The value amount transferred to the users wallet address
        var result = Blockchain.transfer(Blockchain.transaction.from, value);
        if (result > 0) {

                // Update Vault Balance (New Balance = Previous Balance - pending Amount)
            deposit.balance = balance.dividedBy(value).toString();

                // Store the new balance information on the chain
            this.bankVault.put(Blockchain.transaction.from, deposit);
        }
        return result;
    }
};

module.exports = BankVaultContract;

```

## Deploy smart contracts
Here's how to write a smart contract in Nebulas, and now we need to deploy the smart contract to the chain.
Earlier, I introduced how users made an account transaction in Nebulas, and we used the sendTransation () interface to initiate an account transaction. Deploying a smart contract in Nebulas is actually sending a transaction, just by calling the sendTransation () interface.

```js
sendTransation (from, to, nonce, source, args)
```
We agree that if from and to are the same address, we assume that we are deploying a smart contract.

source: Source code for the smart contract to be deployed

args: Parameters used to deploy smart contracts

Example of deploying a smart contract using curl:

```js

// Request
curl -i -H 'Accept: application / json' -X POST http: // localhost: 8090 / v1 / transaction -H 'Content-Type: application / json' -d '{ "from": "8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf", " to ":" 8a209cec02cbeab7e2f74ad969d2dfe8dd24416aa65589bf "," nonce ": 1," source ":" \ "use strict \"; var BankVaultContract = function () {LocalContractStorage.defineMapProperty (this, \ "bankVault \")}; BankVaultContract.prototype = {init: function () {}, save: function () {var deposit = this.bankVault.get (Blockchain.transaction.from); var value = new BigNumber (Blockchain.transaction.value); if (deposit! = null && deposit var balance = new BigNumber (deposit.balance); value = value.plus (balance)} var content = {balance: value.toString ()}; this.bankVault.put (Blockchain.transaction .from, content)}, takeout: function (amount) {var deposit = this.bankVault.get (Blockchain.transaction.from); if (deposit == null) {return 0} var balance = new BigNumber (deposit.balance var value = new BigNumber (amount); if (balance.lessThan (value)) {return 0} var result = Blockchain.tr ansfer (Blockchain.transaction.from, value); if (result> 0) {deposit.balance = balance.dividedBy (value) .toString (); this.bankVault.put (Blockchain.transaction.from, deposit)} return result result }}; module.exports = BankVaultContract; "," args ":" "} '

// Result
{
    "txhash": "9ea7c434f717123ef335fc4e74671996a95b3dcef7873cbe77dc679d6b500a8c",
    "contract_address": "a111489535426e9
```