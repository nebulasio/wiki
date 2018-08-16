# Smart Contract （[中文版链接](https://github.com/nebulasio/wiki/blob/master/smart_contract_ch.md)）

## Languages

In Nebulas, there are two supported smart contract languages:
 - [JavaScript](https://en.wikipedia.org/wiki/JavaScript)
 - [TypeScript](https://en.wikipedia.org/wiki/TypeScript)

They are supported by the integration of [Chrome V8](https://developers.google.com/v8/), a widely used JavaScript engine developed by The Chromium Project for Google Chrome and Chromium web browsers.

## Execution Model

The diagram below is the Execution Model of Smart Contract:

![Smart Contract Execution Model](resources/smart_contract_execution_model.png "Smart Contract Execution Model")

1. All src of Smart Contract and arguments are packaged in Transaction and deployed on Nebulas.
2. The execution of Smart Contract are divided into two phases:
    1. Preprocess: inject tracing instruction, etc.
    2. Execute: generate executable src and execute it.

## Contracts

Contracts in Nebulas are similar to classes in object-oriented languages. They contain persistent data in state variables and functions that can modify these variables.


### Writing Contract

A contract must be a Prototype Object or Class in JavaScript or TypeScript.

A Contract must include an ```init``` function, it will be executed only once when deploying. Functions, named starting with ```_``` are ```private```, can't be executed in Transaction. The others are all ```public``` and can be executed in Transaction.

Since Contract is executed in Chrome V8, all instance variables are in memory, it's not wise to save all of them to [state trie](https://github.com/nebulasio/wiki/blob/master/merkle_trie.md) in Nebulas. In Nebulas, we provide ```LocalContractStorage``` and ```GlobalContractStorage``` objects to help developers define fields needing to be saved to state trie. And those fields should be defined in ```constructor``` of Contract, before other functions.

The following is a sample contract:

```javascript
class Rectangle {
    constructor() {
        // define fields stored to state trie.
        LocalContractStorage.defineProperties(this, {
            height: null,
            width: null,
        });
    }

    // init function.
    init(height, width) {
        this.height = height;
        this.width = width;
    }

    // calc area function.
    calcArea() {
        return this.height * this.width;
    }

    // verify function.
    verify(expected) {
        let area = this.calcArea();
        if (expected != area) {
            throw new Error("Error: expected " + expected + ", actual is " + area + ".");
        }
    }
}
```

### Visibility

In JavaScript, there is no function visibility, all functions defined in prototype object are public.

In Nebulas, we define two kinds of visibility ```public``` and ```private```:

* ```public```
All functions whose name matches regexp ```^[a-zA-Z$][A-Za-z0-9_$]*$``` are public, except ```init```. Public functions can be called via Transaction.

* ```private```
All functions whose name starts with ```_``` are private. A private function can only be called by public functions.

## Global Objects

### console

The ```console``` module provides a simple debugging console that is similar to the JavaScript console mechanism provided by web browsers.

The global console can be used without calling ```require('console')```.

#### console.info([...args])
* ```...args <any>```
The console.info() function is an alias for ```console.log()```.

#### console.log([...args])
* ```...args <any>```
Print ```args``` to Nebulas Logger at level ```info```.

#### console.debug([...args])
* ```...args <any>```
Print ```args``` to Nebulas Logger at level ```debug```.

#### console.warn([...args])
* ```...args <any>```
Print ```args``` to Nebulas Logger at level ```warn```.

#### console.error([...args])
* ```...args <any>```
Print ```args``` to Nebulas Logger at level ```error```.

### LocalContractStorage

The ```LocalContractStorage``` module provides a state trie based storage capability. It accepts string only key value pairs. And all data are stored to a private state trie associated with current contract address, only the contract can access them.

```typescript
interface Descriptor {
    // serialize value to string;
    stringify?(value: any): string;

    // deserialize value from string;
    parse?(value: string): any;
}

interface DescriptorMap {
    [fieldName: string]: Descriptor;
}

interface ContractStorage {
    // get and return value by key from Native Storage.
    rawGet(key: string): string;
    // set key and value pair to Native Storage,
    // return 0 for success, otherwise failure.
    rawSet(key: string, value: string): number;

    // define a object property named `fieldname` to `obj` with descriptor.
    // default descriptor is JSON.parse/JSON.stringify descriptor.
    // return this.
    defineProperty(obj: any, fieldName: string, descriptor?: Descriptor): any;

    // define object properties to `obj` from `props`.
    // default descriptor is JSON.parse/JSON.stringify descriptor.
    // return this.
    defineProperties(obj: any, props: DescriptorMap): any;

    // define a StorageMap property named `fieldname` to `obj` with descriptor.
    // default descriptor is JSON.parse/JSON.stringify descriptor.
    // return this.
    defineMapProperty(obj: any, fieldName: string, descriptor?: Descriptor): any;

    // define StorageMap properties to `obj` from `props`.
    // default descriptor is JSON.parse/JSON.stringify descriptor.
    // return this.
    defineMapProperties(obj: any, props: DescriptorMap): any;

    // delete key from Native Storage.
    // return 0 for success, otherwise failure.
    del(key: string): number;

    // get value by key from Native Storage,
    // deserialize value by calling `descriptor.parse` and return.
    get(key: string): any;

    // set key and value pair to Native Storage,
    // the value will be serialized to string by calling `descriptor.stringify`.
    // return 0 for success, otherwise failure.
    set(key: string, value: any): number;
}

interface StorageMap {
    // delete key from Native Storage, return 0 for success, otherwise failure.
    del(key: string): number;

    // get value by key from Native Storage,
    // deserialize value by calling `descriptor.parse` and return.
    get(key: string): any;

    // set key and value pair to Native Storage,
    // the value will be serialized to string by calling `descriptor.stringify`.
    // return 0 for success, otherwise failure.
    set(key: string, value: any): number;
}
```

### BigNumber

The `BigNumber` module use the [bignumber.js](https://github.com/MikeMcl/bignumber.js)(v4.1.0), a JavaScript library for arbitrary-precision decimal and non-decimal arithmetic. The contract can use `BigNumber` directly to handle the value of the transaction and other values transfer.

```js

var value = new BigNumber(0);
value.plus(1);
...
```

### Blockchain
The `Blockchain` module provides a object for contracts to obtain transactions and blocks executed by the current contract. Also, the NAS can be transferred from the contract and the address check is provided.

Blockchain API:

```js

// current block 
Blockchain.block;

// current transaction, transaction's value/gasPrice/gasLimit auto change to BigNumber object
Blockchain.transaction;

// transfer NAS from contract to address
Blockchain.transfer(address, value);

// verify address
Blockchain.verifyAddress(address);

// get accout state
Blockchain.getAccountState(address);

// get previous block's hash
Blockchain.getPreBlockHash(offset);

// get previous block's random seed
Blockchain.getPreBlockSeed(offset);

```
properties:

- `block`: current block for contract execution
	- `timestamp`: block timestamp
	- `seed`: random seed, return `""` since v1.1.0.
	- `height`: block height
- `transaction`: current transaction for contract execution
	- `hash`: transaction hash
	- `from`: transaction from address
	- `to`: transaction to address
	- `value`: transaction value, a BigNumber object for contract use
	- `nonce`: transaction nonce
	- `timestamp`: transaction timestamp
	- `gasPrice`: transaction gasPrice, a BigNumber object for contract use
	- `gasLimit`: transaction gasLimit, a BigNumber object for contract use
- `transfer(address, value)`: transfer NAS from contract to address
	- params:
		- `address`: nebulas address to receive NAS
		- `value`: transfer value, a BigNumber/**Uint(recommended)** object. The unit is wei, only integer value is valid.
	- return (boolean value):
		- `true`: transfer success
		- `false`: transfer failed   
- `verifyAddress(address)`: verify address
	- params:
		- `address`: address need to check
	- return (number value):
		- `87`: user wallet address
		- `88`: smart-contract address
		- `0`: address is invalid 
- `getAccountState(address)`: get account's balance and nonce
	- params:
		- `address`: whose address you want to get
	- return (JSON object):
		- `balance`: account's balance
		- `nonce`: account's nonce
- `getPreBlockHash(offset)`: get a previous block's hash
	- params:
		- `offset`: the offset between the block and current block. This param should be an integer larger then 0 and less than current height. eg: If you want to get the hash of the previous block just before current block, the offset should be set as 1.
	- return(string value):
		- `hash`: block hash
- `getPreBlockSeed(offset)`: get a previous block's seed
	- params:
		- `offset`: same as the one in Blockchain.getPreBlockHash()
	- return(string value):
		- `seed`: block random seed
Example to use:

```js

'use strict';

var SampleContract = function () {
    LocalContractStorage.defineProperties(this, {
        name: null,
        count: null
    });
    LocalContractStorage.defineMapProperty(this, "allocation");
};

SampleContract.prototype = {
    init: function (name, count, allocation) {
        this.name = name;
        this.count = count;
        allocation.forEach(function (item) {
            this.allocation.put(item.name, item.count);
        }, this);
        console.log('init: Blockchain.block.coinbase = ' + Blockchain.block.coinbase);
        console.log('init: Blockchain.block.hash = ' + Blockchain.block.hash);
        console.log('init: Blockchain.block.height = ' + Blockchain.block.height);
        console.log('init: Blockchain.transaction.from = ' + Blockchain.transaction.from);
        console.log('init: Blockchain.transaction.to = ' + Blockchain.transaction.to);
        console.log('init: Blockchain.transaction.value = ' + Blockchain.transaction.value);
        console.log('init: Blockchain.transaction.nonce = ' + Blockchain.transaction.nonce);
        console.log('init: Blockchain.transaction.hash = ' + Blockchain.transaction.hash);
    },
    transfer: function (address, value) {
        var result = Blockchain.transfer(address, value);
        console.log("transfer result:", result);
        Event.Trigger("transfer", {
			Transfer: {
				from: Blockchain.transaction.to,
				to: address,
				value: value
			}
		});
    },
    verifyAddress: function (address) {
    	 var result = Blockchain.verifyAddress(address);
        console.log("verifyAddress result:", result);
    },
	getAccountState: function (address) {
		var state = Blockchain.getAccountState(address);
		console.log("getAccountState result:", state);
	},
	getPreBlockHash: function (offset) {
		var hash = Blockchain.getPreBlockHash(offset);
		console.log("getPreBlockHash result", hash);
	},
	getPreBlockSeed: function (offset) {
		var seed = Blockchain.getPreBlockSeed(offset);
		console.log("getPreBlockSeed result", seed);
	}
};

module.exports = SampleContract;

```

### Event

The `Event` module records execution events in contract. The recorded events are stored in the event trie on the chain, which can be fetched by [`GetEventsByHash`](https://github.com/nebulasio/wiki/blob/master/rpc.md#geteventsbyhash)API with the execution transaction hash. All contract event topics have a `chain.contract.` prefix before the topic they set in contract.

```js
Event.Trigger(topic, obj);
```

 - `topic`: user-defined topic
 - `obj`: JSON object


You can see the example in `SampleContract` before.

### Math.random 
* `Math.random()` returns a floating-point, pseudo-random number in the range from 0 inclusive up to but not including 1. The typical usage is:

```js
"use strict";

var BankVaultContract = function () {};

BankVaultContract.prototype = {

	init: function () {},

	game: function(subscript){
	
		var arr =[1,2,3,4,5,6,7,8,9,10,11,12,13];

		for(var i = 0;i < arr.length; i++){
			var rand = parseInt(Math.random()*arr.length);
			var t = arr[rand];
			arr[rand] =arr[i];
			arr[i] = t;
		}

		return arr[parseInt(subscript)];
	},
};
module.exports = BankVaultContract;
```

* `Math.random.seed(myseed)` if needed, you can use this method to reset random seed. The argument `myseed` must be a **non empty string**. **NOTE** This method will be removed since v1.1.0.
```js
"use strict";

var BankVaultContract = function () {};

BankVaultContract.prototype = {

	init: function () {},
	
	game:function(subscript, myseed){
	
		var arr =[1,2,3,4,5,6,7,8,9,10,11,12,13];
		
		console.log(Math.random());
	
		for(var i = 0;i < arr.length; i++){
		
			if (i == 8) {
				// reset random seed with `myseed`
				Math.random.seed(myseed);
			}

			var rand = parseInt(Math.random()*arr.length);
			var t = arr[rand];
			arr[rand] =arr[i];
			arr[i] = t;
		}
		return arr[parseInt(subscript)];
	},
};

module.exports = BankVaultContract;
```

### Date 
All standardized APIs are available. Note that the timezone is fixed to "UTC+0" and locale to "en-US".
```js
"use strict";

var BankVaultContract = function () {};

BankVaultContract.prototype = {
	init: function () {},
	
	test: function(){
		var d = new Date();
		return d.toString();
	}
};

module.exports = BankVaultContract;
```
Tips:
* `new Date()`/`Date.now()` returns the timestamp of current block in milliseconds.


### accept
this method is aimed to make it possible to send a binary transfer to a contract account. As the `to` is a smart contact address, which has declared a function `accept()` and it excutes correctly, the transfer will succeed. If the Tx is a non-binary Tx,it will be treated as a normal function.
```js
"use strict";
var DepositeContent = function (text) {
	if(text){
        	var o = JSON.parse(text);
        	this.balance = new BigNumber(o.balance);//余额信息
        	this.address = o.address;
	}else{
        	this.balance = new BigNumber(0);
        	this.address = "";
        }
};

DepositeContent.prototype = {
	toString: function () {
  		return JSON.stringify(this);
	}
};

var BankVaultContract = function () {
	LocalContractStorage.defineMapProperty(this, "bankVault", {
 		parse: function (text) {
  			return new DepositeContent(text);
 		},
 		stringify: function (o) {
  			return o.toString();
 		}
	});
};

BankVaultContract.prototype = {
	init: function () {},

	save: function () {
  		var from = Blockchain.transaction.from;
  		var value = Blockchain.transaction.value;
  		value = new BigNumber(value);
  		var orig_deposit = this.bankVault.get(from);
  		if (orig_deposit) {
    			value = value.plus(orig_deposit.balance);
  		}
		
  		var deposit = new DepositeContent();
  		deposit.balance = new BigNumber(value);
  		deposit.address = from;
  		this.bankVault.put(from, deposit);
	},

	accept:function(){
		this.save();
		Event.Trigger("transfer", {
			Transfer: {
				from: Blockchain.transaction.from,
				to: Blockchain.transaction.to,
				value: Blockchain.transaction.value,
			}
   		});
	}

};
module.exports = BankVaultContract;
```

### Uint

The `Uint` encapsulates 4 unsigned integer types based on bignumber.js, i.e., `Uint64`, `Uint128`, `Uint256`, `Uint512`.

Static properties:

- `MaxValue`: an instance for the maximum value of specific uint type

Instance APIs:

- `div(o)`: __/__
    - params:
        - `o`: divisor, must be the same type with `this`
    - return: arithmetic result of the same type with `this`

- `pow(o)`: __^__
    - params:
        - `o`: exponent, must be the same type with `this`
    - return: arithmetic result of the same type with `this`

- `minus(o)`: __-__
    - params:
        - `o`: subtractor, must be the same type with `this`
    - return: arithmetic result of the same type with `this`

- `mod(o)`: __%__
    - params:
        - `o`: modulo, must be the same type with `this`
    - return: arithmetic result of the same type with `this`

- `mul(o)`: __*__
    - params:
        - `o`: multiplier, must be the same type with `this`
    - return: arithmetic result of the same type with `this`

- `plus(o)`: __+__
    - params:
        - `o`: addend, must be the same type with `this`
    - return: arithmetic result of the same type with `this`

- `cmp(o)`
    - params:
        - `o`: must be the same type with `this`
    - return:
        - `1`: `this` > `o`
        - `0`: `this` = `o`
        - `-1`: `this` < `o`

- `isZero()`
    - return:
        - `true`: `this` is 0
        - `false`: `this` is not 0

- `toString(base)`: convert `this` to string value
    - params:
        - `base`: 2 ~ 64, default 10

Example:

```js
'use strict';

var Uint64 = Uint.Uint64;
// var Uint128 = Uint.Uint128;
// var Uint256 = Uint.Uint256;
// var Uint512 = Uint.Uint512;

var Contract = function() {};

Contract.prototype = {
    init: function() {},

    testUint64: function() {
        var a  = new Uint64(7);
        var b = new Uint64("2");

        return {
            'a+b': a.plus(b).toString(10),  // 9
            'a-b': a.minus(b).toString(10), // 5
            'a*b': a.mul(b).toString(10),   // 14
            'a/b': a.div(b).toString(10),   // 3
            'a%b': a.mod(b).toString(10),   // 1
            'a^b': a.pow(b).toString(10),   // 49
            'a>b': a.cmp(b) == 1,           // true
            'a==0': a.isZero(),             // false
            'a': a.toString(),              // 7
            'MaxUint64': Uint64.MaxValue.toString(16), // ffffffffffffffff
        };
    }
};

module.exports = Contract;
```

### require

The `require` function is used to explicitly load third-party libraries those Nebulas NVM doesn't load at startup. 

Available libraries are:

* `crypto.js`

The typical usage is:
    
```js
    var crypto = require('crypto.js');
    ...
```


### crypto

The `crypto` provides several frequently-used cryptographic hash functions. This module need to be explicitly required.

APIs:

- `sha256(str)`
    - param:
        - `str`: any string, case-sensitive
    - return: 
        - hexadecimal hash string, 64 chars

- `sha3256(str)`
    - param:
        - `str`: any string, case-sensitive
    - return: 
        - hexadecimal hash string, 64 chars

- `ripemd160(str)`
    - param:
        - `str`: any string, case-sensitive
    - return: 
        - hexadecimal hash string, 40 chars

- `md5(str)`
    - param:
        - `str`: any string, case-sensitive
    - return: 
        - hexadecimal hash string, 32 chars

- `base64(str)`
    - param:
        - `str`: any string, case-sensitive
    - return: 
        - base64 string

- `recoverAddress(alg, hash, sign)`: recover signer address from public key
    - param:
        - `alg`: signature algorithm, only one value __`1`__, indicating secp256k1
        - `hash`: input message, hex string, 64 chars
        - `sign`: signature, computed with signer's private key on `hash`
    - return: 
        - Nebulas address string, or `null` if failed.

Example:
```js
'use strict';

// explicitly require
var crypto = require('crypto.js');

var Contract = function() {};

Contract.prototype = {
    init: function() {},

    sha256: function(str) {
        // str='Nebulas is a next generation public blockchain, aiming for a continuously improving ecosystem.'

        // return "a32d6d686968192663b9c9e21e6a3ba1ba9b2e288470c2f98b790256530933e0"
        return crypto.sha256(str);
    },

    sha3256: function(str) {
        // str='Nebulas is a next generation public blockchain, aiming for a continuously improving ecosystem.'

        // return "564733f9f3e139b925cfb1e7e50ba8581e9107b13e4213f2e4708d9c284be75b"
        return crypto.sha3256(str);
    },

    ripemd160: function(str) {
        // str='Nebulas is a next generation public blockchain, aiming for a continuously improving ecosystem.'

        // return "4236aa9974eb7b9ddb0f7a7ed06d4bf3d9c0e386"
        return crypto.ripemd160(str);
    },

    md5: function(str) {
        // str='Nebulas is a next generation public blockchain, aiming for a continuously improving ecosystem.'

        // return "9954125a33a380c3117269cff93f76a7"
        return crypto.md5(str);
    },

    base64: function(str) {
        // str='Nebulas is a next generation public blockchain, aiming for a continuously improving ecosystem.'

        // return "TmVidWxhcyBpcyBhIG5leHQgZ2VuZXJhdGlvbiBwdWJsaWMgYmxvY2tjaGFpbiwgYWltaW5nIGZvciBhIGNvbnRpbnVvdXNseSBpbXByb3ZpbmcgZWNvc3lzdGVtLg=="
        return crypto.base64(str);
    },

    recoverAddress: function(alg, hash, sign) {
        // alg = 1
        // hash = '564733f9f3e139b925cfb1e7e50ba8581e9107b13e4213f2e4708d9c284be75b'
        // sign = 'd80e282d165f8c05d8581133df7af3c7c41d51ec7cd8470c18b84a31b9af6a9d1da876ab28a88b0226707744679d4e180691aca6bdef5827622396751a0670c101'

        // return 'n1F8QbdnhqpPXDPFT2c9a581tpia8iuF7o2'
        return crypto.recoverAddress(alg, hash, sign);
    }
};

module.exports = Contract;
```

### Call between contracts（since 1.1.0, testnet）

We provide a simple method in a smart contract to call another contract, the following example shows that how proxyKvStore provides service by using the contract kvStore.

proxyKvStore.js:
```js
"use strict"

var proxyKvStore = function() {
};

proxyKvStore.prototype = {
    init: function() {
        //
    },


    save: function(address, key, value) {

        var kvStore  = new Blockchain.Contract(address);
        kvStore.value(20000000000000000).call("save", key, value);
    },

    get: function(address, key) {
        var kvStore = new Blockchain.Contract(address);
        return kvStore.call("get", key)
    },
}

module.exports = proxyKvStore;
```
kvStore.js:

```js
"use strict";

var item = function(text) {
  if (text) {
    var obj = JSON.parse(text);
    this.key = obj.key;
    this.value = obj.value;
    this.author = obj.text;
  } else {
      this.key = "";
      this.author = "";
      this.value = "";
  }
};

item.prototype = {
  toString: function () {
    return JSON.stringify(this);
  }
};

var kvStore = function () {
    LocalContractStorage.defineMapProperty(this, "repo", {
        parse: function (text) {
            return new item(text);
        },
        stringify: function (o) {
            return o.toString();
        }
    });
};

kvStore.prototype = {
    init: function () {
        // todo
    },

    save: function (key, value) {
        console.log("reach child contract");

        key = key.trim();
        value = value.trim();
        if (key === "" || value === ""){
            throw new Error("empty key / value");
        }
        if (value.length > 128 || key.length > 128){
            throw new Error("key / value exceed limit length")
        }

        var from = Blockchain.transaction.from;
        var item = this.repo.get(key);
    
        if (item){
            throw new Error("value has been taken");
        }

        item = new item();
        item.author = from;
        item.key = key;
        item.value = value;
        this.repo.put(key, item);
    },

    get: function (key) {
        key = key.trim();
        if ( key === "" ) {
            throw new Error("empty key")
        }
        return this.repo.get(key);
    }, 

    throwErr: function() {
        throw("err for test");
    }
};
module.exports = kvStore;
```

In the example, to use KvStore, we first create a contract object with the address of the contract you want to call:

```js
var kvStore  = new Blockchain.Contract(address);
```

After that, we can call function in KvStore.js through this object.
```js
kvStore.value(2000000000000000000).call("save", key, value);
```
or
```js
kvStore.call("save", key, value);
```

The 'value' function decides how much NAS will be transfered to the called contract. It is not necessary, and the default value is 0.

It should be noted that in the execution environment of the called contract, Blockchain.from is the address of the calling contract, and Blockchain.value is determined by the parameter of the 'value' function executed by the calling contract.

If the called contract has an throw error, the calling contract cannot catch it, and all modified data will be rolled back.

Label:

- `Multi-level calls allow up to 3 orders, such as A->B->C. Can't call D` again
	- When A->B, the from address of B is the contract address of A. If B wants to get the address of caller A, the interface is directly passed as a parameter to the calling function of the B contract.
- `gas consumption`
	- The cost of the gas after the instrument code is inserted into the pile, using the single-order call consistent consumption rule
	- When calling B across contract A, you need to pay 32,000 gas
	- The contract call fails when there is insufficient gas in any order. Roll back the entire process
- `mem limit`
	- When the single-order contract is called, the maximum consumption value is limited to 40M. When multi-level calls are made, the maximum memory allowed to be consumed is 40M.
	- When there is insufficient mem in any contract, the contract call fails and the whole process is rolled back.
- `event`
	- Each call to cross-contract will trigger an event.topic:chain.innerContract. For example, A->B->C. will trigger the event of A->B and B->C