# NVM - Nebulas Virtual Machine

NVM is the most important components in Nebulas. As the name described, it used to provide a managed virtual machine execution environments for Smart Contract and Protocol Code.

[go-nebulas](https://github.com/nebulasio/go-nebulas) now support two kinds of Virtual Machine:
 - V8: [Chrome V8](https://developers.google.com/v8/)
 - LLVM: [Low-Level Virtual Machine](https://llvm.org)

## V8

In go-nebulas, we design and implement [V8 Engine](https://github.com/nebulasio/wiki/blob/master/v8_engine.md) based on Chrome V8.

V8 Engine provides a high performance sandbox for [Smart Contract](https://github.com/nebulasio/wiki/blob/master/smart_contract.md) Execution, make sure user deployed codes is running in a managed environments, prevent massive resource consumption to host. By using Chrome V8, that means [JavaScript](https://en.wikipedia.org/wiki/JavaScript) and [TypeScript](https://en.wikipedia.org/wiki/TypeScript) are the [Smart Contract](https://github.com/nebulasio/wiki/blob/master/smart_contract.md) language in Nebulas, anyone who familiar with JavaScript or TypeScript can write their own Smart Contract and run in Nebulas.

The following content is a example of Smart Contract written in JavaScript:

```javascript
"use strict";

var BankVaultContract = function() {
    LocalContractStorage.defineMapProperty(this, "bankVault");
};

// save value to contract, only after height of block, users can takeout
BankVaultContract.prototype = {
    init:function() {},
    save:function(height) {
        var deposit = this.bankVault.get(Blockchain.transaction.from);
        var value = new BigNumber(Blockchain.transaction.value);
        if (deposit != null && deposit.balance.length > 0) {
            var balance = new BigNumber(deposit.balance);
            value = value.plus(balance);
        }
        var content = {
            balance:value.toString(),
            height:Blockchain.block.height + height
        };
        this.bankVault.put(Blockchain.transaction.from, content);
    },
    takeout:function(amount) {
        var deposit = this.bankVault.get(Blockchain.transaction.from);
        if (deposit == null) {
            return 0;
        }
        if (Blockchain.block.height < deposit.height) {
            return 0;
        }
        var balance = new BigNumber(deposit.balance);
        var value = new BigNumber(amount);
        if (balance.lessThan(value)) {
            return 0;
        }
        var result = Blockchain.transfer(Blockchain.transaction.from, value);
        if (result > 0) {
            deposit.balance = balance.dividedBy(value).toString();
            this.bankVault.put(Blockchain.transaction.from, deposit);
        }
        return result;
    }
};

module.exports = BankVaultContract;
```
For more information about the smart contract in Nebulas, please go to [Smart Contract](https://github.com/nebulasio/wiki/blob/master/smart_contract.md).

For more formation about the design of V8 Engine, please go to [V8](https://github.com/nebulasio/wiki/blob/master/v8_engine.md).

## LLVM

TBD.
