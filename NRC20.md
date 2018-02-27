# NRC20


## Abstract

The following standard allows for the implementation of a standard API for tokens within smart contracts.
This standard provides basic functionality to transfer tokens, as well as allow tokens to be approved so they can be spent by another on-chain third party.


## Motivation

A standard interface allows any tokens on Nebulas to be re-used by other applications: from wallets to decentralized exchanges.


## Methods

#### name

Returns the name of the token - e.g. `"MyToken"`.

``` js
// returns string, the name of the token.
function name()
```

#### symbol

Returns the symbol of the token. E.g. "TK".

``` js
// returns string, the symbol of the token 
function symbol()
```

#### decimals

Returns the number of decimals the token uses - e.g. `8`, means to divide the token amount by `100000000` to get its user representation.

``` js
// returns number, the number of decimals the token uses
function decimals()
```


#### totalSupply

Returns the total token supply.

``` js
// returns string, the total token supply, the decimal value is decimals* total.
function totalSupply()
```

#### balanceOf

Returns the account balance of another account with address.

``` js
// returns string, the account balance of another account with address
function balanceOf(address)
```

#### transfer

Transfers `value` amount of tokens to `address `, and MUST fire the `Transfer` event.
The function SHOULD `throw` if the `from` account balance does not have enough tokens to spend.

*Note* Transfers of 0 values MUST be treated as normal transfers and fire the `Transfer` event.

``` js
// returns `true`, if transfer success, else throw error
function transfer(address, value)
```



#### transferFrom

Transfers `value` amount of tokens from address `from` to address `to`, and MUST fire the `Transfer` event.

The `transferFrom` method is used for a withdraw workflow, allowing contracts to transfer tokens on your behalf.
This can be used for example to allow a contract to transfer tokens on your behalf and/or to charge fees in sub-currencies.
The function SHOULD `throw` unless the `from` account has deliberately authorized the sender of the message via some mechanism.

*Note* Transfers of 0 values MUST be treated as normal transfers and fire the `Transfer` event.

``` js
// returns `true`, if transfer success, else throw error
function transferFrom(from, to, value)
```



#### approve

Allows `spender` to withdraw from your account multiple times, up the `currentValue` to the `value` amount. If this function is called again it overwrites the current allowance with `value`.

**NOTE**: To prevent attack vectors, the user needs to give a previous approve value, and the default value that is not approve is 0.

``` js
// returns `true`, if approve success, else throw error
function approve(spender, currentValue, value)
```

#### allowance

Returns the amount which `spender` is still allowed to withdraw from `owner`.

``` js
// returns string, the value of allowed to withdraw from `owner`.
function allowance(owner, spender)
```

### Events


#### transferEvent

MUST trigger when tokens are transferred, including zero value transfers.

A token contract which creates new tokens SHOULD trigger a Transfer event with the `from` address set to `totalSupply` when tokens are created.

``` js
function transferEvent: function(status, from, to, value)
```



#### approveEvent

MUST trigger on any call to `approve(spender, currentValue, value)`.

``` js
function approveEvent: function(status, from, spender, value)
```

## Implementation

#### Example implementations are available at
- [NRC20.js](https://github.com/nebulasio/go-nebulas/blob/master/nf/nvm/test/NRC20.js)

```js
'use strict';

var Allowed = function (obj) {
    this.allowed = {};
    this.parse(obj);
}

Allowed.prototype = {
    toString: function () {
        return JSON.stringify(this.allowed);
    },

    parse: function (obj) {
        if ( typeof obj != "undefined" ) {
            var data = JSON.parse(obj);
            for (var key in data) {
                this.allowed[key] = new BigNumber(data[key]);
            }
        }
    },

    get: function (key) {
        return this.allowed[key];
    },

    set: function (key, value) {
        this.allowed[key] = new BigNumber(value);
    }
}

var StandardToken = function () {
    LocalContractStorage.defineProperties(this, {
        _name: null,
        _symbol: null,
        _decimals: null,
        _totalSupply: {
            parse: function (value) {
                return new BigNumber(value);
            },
            stringify: function (o) {
                return o.toString(10);
            }
        }
    });

    LocalContractStorage.defineMapProperties(this, {
        "balances": {
            parse: function (value) {
                return new BigNumber(value);
            },
            stringify: function (o) {
                return o.toString(10);
            }
        },
        "allowed": {
            parse: function (value) {
                return new Allowed(value);
            },
            stringify: function (o) {
                return o.toString();
            }
        }
    });
};

StandardToken.prototype = {
    init: function (name, symbol, decimals, totalSupply) {
        this._name = name;
        this._symbol = symbol;
        this._decimals = decimals | 0;
        this._totalSupply = new BigNumber(totalSupply).mul(new BigNumber(10).pow(decimals));

        var from = Blockchain.transaction.from;
        this.balances.set(from, this._totalSupply);
        this.transferEvent(true, from, from, this._totalSupply);
    },

    // Returns the name of the token
    name: function () {
        return this._name;
    },

    // Returns the symbol of the token
    symbol: function () {
        return this._symbol;
    },

    // Returns the number of decimals the token uses
    decimals: function () {
        return this._decimals;
    },

    totalSupply: function () {
        return this._totalSupply.toString(10);
    },

    balanceOf: function (owner) {
        var balance = this.balances.get(owner);

        if (balance instanceof BigNumber) {
            return balance.toString(10);
        } else {
            return "0";
        }
    },

    transfer: function (to, value) {
        value = new BigNumber(value);

        var from = Blockchain.transaction.from;
        var balance = this.balances.get(from) || new BigNumber(0);

        if (balance.lt(value)) {

            this.transferEvent(false, from, to, value);
            throw new Error("transfer failed.");
        }

        this.balances.set(from, balance.sub(value));
        var toBalance = this.balances.get(to) || new BigNumber(0);
        this.balances.set(to, toBalance.add(value));

        this.transferEvent(true, from, to, value);
        return true;
    },

    transferFrom: function(from, to, value) {
        var txFrom = Blockchain.transaction.from;
        var balance = this.balances.get(from) || new BigNumber(0);

        var allowed = this.allowed.get(from) || new Allowed();
        var allowedValue = allowed.get(txFrom) || new BigNumber(0);

        if (balance.gte(value) && allowedValue.gte(value)) {

            this.balances.set(from, balance.sub(value));

            // update allowed value
            allowed.set(txFrom, allowedValue.sub(value));
            this.allowed.set(from, allowed);

            var toBalance = this.balances.get(to) || new BigNumber(0);
            this.balances.set(to, toBalance.add(value));

            this.transferEvent(true, from, to, value);
            return true;
        } else {
            this.transferEvent(false, from, to, value);
            throw new Error("transfer failed.");
        }
    },

    transferEvent: function(status, from, to, value) {
        Event.Trigger(this.name(), {
            Status: status,
            Transfer: {
                from: from,
                to: to,
                value: value
            }
        });
    },

    approve: function (spender, currentValue, value) {
        var from = Blockchain.transaction.from;

        var oldValue = this.allowance(from, spender);
        if (oldValue != currentValue.toString()) {
            this.approveEvent(false, from, spender, value);
            throw new Error("approve failed.");
        }

        var owned = this.allowed.get(from) || new Allowed();

        owned.set(spender, new BigNumber(value));

        this.allowed.set(from, owned);

        this.approveEvent(true, from, spender, value);
        
        return true;
    },

    approveEvent: function(status, from, spender, value) {
        Event.Trigger(this.name(), {
            Status: status,
			Approve: {
				owner: from,
				spender: spender,
				value: value
			}
        });
    },

    allowance: function (owner, spender) {
        var owned = this.allowed.get(owner);

        if (owned instanceof Allowed) {
            var spender = owned.get(spender);
            if ( typeof spender != "undefined") {
                return spender.toString(10);
            }
        }
        return "0";
    }
};

module.exports = StandardToken;

```