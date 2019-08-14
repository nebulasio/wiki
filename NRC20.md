# NRC20


## Abstract

The following standard allows for the implementation of a standard API for tokens within smart contracts.
This standard provides basic functionality to transfer tokens, as well as allows tokens to be approved so they can be spent by another on-chain third party.


## Motivation

A standard interface allows that a new token can be created by any application easily : from wallets to decentralized exchanges.


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

Returns the account balance  of a address.

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

***Notice:Transfer `value` only supports numeric strings.***

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
// returns string, the value allowed to withdraw from `owner`.
function allowance(owner, spender)
```

### Events


#### transferEvent

MUST trigger when tokens are transferred, including zero value transfers.

A token contract which creates new tokens SHOULD trigger a Transfer event with the `from` address set to `totalSupply` when tokens are created.

``` js
function _transferEvent: function(status, from, to, value)
```



#### approveEvent

MUST trigger on any call to `approve(spender, currentValue, value)`.

``` js
function _approveEvent: function(status, from, spender, value)
```

***Notice:All event triggers must be the result of the execution of the contract method and cannot be called externally.***

## Implementation

#### Example implementations are available at
- [NRC20.js](https://github.com/nebulasio/go-nebulas/blob/master/nf/nvm/test/NRC20.js)

```js
'use strict';

let Allowed = function (obj) {
    this._allowed = {};
    this.parse(obj);
};

Allowed.prototype = {
    toString: function () {
        return JSON.stringify(this._allowed);
    },

    parse: function (obj) {
        if (typeof obj != "undefined") {
            let data = JSON.parse(obj);
            for (let key in data) {
                this._allowed[key] = new BigNumber(data[key]);
            }
        }
    },

    get: function (key) {
        return this._allowed[key];
    },

    set: function (key, value) {
        this._allowed[key] = new BigNumber(value);
    }
};

let StandardToken = function () {
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
        "_balances": {
            parse: function (value) {
                return new BigNumber(value);
            },
            stringify: function (o) {
                return o.toString(10);
            }
        },
        "_allowed": {
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
        this._decimals = decimals || 0;
        this._totalSupply = new BigNumber(totalSupply).mul(new BigNumber(10).pow(decimals));

        let from = Blockchain.transaction.from;
        this._balances.set(from, this._totalSupply);
        this._transferEvent(true, from, from, this._totalSupply);
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
        this._verifyAddress(owner);

        let balance = this._balances.get(owner);
        if (balance instanceof BigNumber) {
            return balance.toString(10);
        } else {
            return "0";
        }
    },
    _verifyAddress: function (address) {
        if (Blockchain.verifyAddress(address) === 0) {
            throw new Error("Address format error, address=" + address);
        }
    },

    _verifyValue: function(value) {
        let bigVal = new BigNumber(value);
        if (bigVal.isNaN() || !bigVal.isFinite()) {
            throw new Error("Invalid value, value=" + value);
        }
        if (bigVal.isNegative()) {
            throw new Error("Value is negative, value=" + value);
        }
        if (!bigVal.isInteger()) {
            throw new Error("Value is not integer, value=" + value);
        }
        if (value !== bigVal.toString(10)) {
            throw new Error("Invalid value format.");
        }
    },

    transfer: function (to, value) {
        this._verifyAddress(to);
        this._verifyValue(value);

        value = new BigNumber(value);
        let from = Blockchain.transaction.from;
        let balance = this._balances.get(from) || new BigNumber(0);

        if (balance.lt(value)) {
            throw new Error("transfer failed.");
        }

        this._balances.set(from, balance.sub(value));
        let toBalance = this._balances.get(to) || new BigNumber(0);
        this._balances.set(to, toBalance.add(value));

        this._transferEvent(true, from, to, value.toString(10));
    },

    transferFrom: function (from, to, value) {
        this._verifyAddress(from);
        this._verifyAddress(to);
        this._verifyValue(value);

        let spender = Blockchain.transaction.from;
        let balance = this._balances.get(from) || new BigNumber(0);

        let allowed = this._allowed.get(from) || new Allowed();
        let allowedValue = allowed.get(spender) || new BigNumber(0);
        value = new BigNumber(value);

        if (balance.gte(value) && allowedValue.gte(value)) {

            this._balances.set(from, balance.sub(value));

            // update allowed value
            allowed.set(spender, allowedValue.sub(value));
            this._allowed.set(from, allowed);

            let toBalance = this._balances.get(to) || new BigNumber(0);
            this._balances.set(to, toBalance.add(value));

            this._transferEvent(true, from, to, value.toString(10));
        } else {
            throw new Error("transfer failed.");
        }
    },

    _transferEvent: function (status, from, to, value) {
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
        this._verifyAddress(spender);
        this._verifyValue(currentValue);
        this._verifyValue(value);

        let from = Blockchain.transaction.from;

        let oldValue = this.allowance(from, spender);
        if (oldValue != currentValue) {
            throw new Error("current approve value mistake.");
        }

        let balance = new BigNumber(this.balanceOf(from));
        value = new BigNumber(value);

        if (balance.lt(value)) {
            throw new Error("invalid value.");
        }

        let owned = this._allowed.get(from) || new Allowed();
        owned.set(spender, value);

        this._allowed.set(from, owned);

        this._approveEvent(true, from, spender, value.toString(10));
    },

    _approveEvent: function (status, from, spender, value) {
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
        this._verifyAddress(owner);
        this._verifyAddress(spender);

        let owned = this._allowed.get(owner);
        if (owned instanceof Allowed) {
            let spenderObj = owned.get(spender);
            if (typeof spenderObj != "undefined") {
                return spenderObj.toString(10);
            }
        }
        return "0";
    }
};

module.exports = StandardToken;
```