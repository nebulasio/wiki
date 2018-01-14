# Nebulas 101 - 04 Smart Contract Storage

Earlier we covered how to write smart contracts and how to deploy and invoke smart contracts in the Nebulas.

Now we introduce in detail the storage of the smart contract. Nebulas smart contracts provide on-chain data storage capabilities. Similar to the traditional key-value storage system (eg: redis), smart contracts can be stored on the Nebulas with a pay (gas).

## Storing LocalContractStorage
Nebulas' Smart Contract environment has built-in storage object `LocalContractStorage`, which can store numbers, strings, and JavaScript objects. The stored data can only be used in smart contracts. Other contracts can not read the stored data.

#### Basics
The `LocalContractStorage` API includes `set`, `get` and `del`, which allow you to store, read, and delete data. Storage can be numbers, strings, objects

##### Storing `LocalContractStorage` Data：

```js
// store data. The data will be stored as JSON strings
LocalContractStorage.put(key, value);
LocalContractStorage.set(key, value);
```
##### Reading `LocalContractStorage ` Data：
```js
// get the value from key
LocalContractStorage.get(key);
```
##### Deleting `LocalContractStorage ` Data：
```js
// delete data, data can not be read after deletion
LocalContractStorage.del(key);
LocalContractStorage.delete(key);
```
Examples:

```js
'use strict';

var SampleContract = function () {
};

SampleContract.prototype = {
    init: function () {
    },
    set: function (name, value) {
        // Storing a string
        LocalContractStorage.set("name",name);
        // Storing a number (value)
        LocalContractStorage.set("value", value);
        // Storing an objects
        LocalContractStorage.set("obj", {name:name,value:value});
    },
    get: function () {
        var name = LocalContractStorage.get("name");
        console.log("name:"+name)
        var value = LocalContractStorage.get("value");
        console.log("value:"+value)
        var obj = LocalContractStorage.get("obj");
        console.log("obj:"+JSON.stringify(obj))
    },
    del: function () {
        var result = LocalContractStorage.del("name");
        console.log("del result:"+result)
    }
};

module.exports = SampleContract;
```

#### Advanced
In addition to the basic` set`, `get`, and` del` methods, `LocalContractStorage` also supports setting storage properties and storing maps to objects, as well as serialization methods for setting storage.

##### Storage Properties
When writing a smart contract, its properties can be set to a storage bit. Reading and writing properties directly uses LocalContractStorage to read and write. When reading and writing, you can also customize the properties of strings and objects' mapping. Usually, for the contract to be storage bit,  properties will be set to storage property in the initialization.

Defining `LocalContractStorage` properties：

```js
    // define a object property named `fieldname` to `obj` with descriptor.
    // default descriptor is JSON.parse/JSON.stringify descriptor.
    // return this.
    defineProperty(obj, fieldName, descriptor);

    // define object properties to `obj` from `props`.
    // default descriptor is JSON.parse/JSON.stringify descriptor.
    // return this.
    defineProperties(obj, descriptorMap);
```

Property binding `LocalContractStorage` to make contracts Example:


```js
'use strict';
var SampleContract = function () {
    // The SampleContract `size` property is a storage property. Reads and writes to` size` will be stored on the chain.
    // The `descriptor` is set to null here, the default JSON.stringify () and JSON.parse () will be used.
    LocalContractStorage.defineMapProperty(this, "size", null);

    // The SampleContract `value` property is a storage property. Reads and writes to` value` will be stored on the chain.
    // Here is a custom `descriptor` implementation, storing as a string, and returning Bignumber object during parsing. 
    LocalContractStorage.defineMapProperty(this, "value", {
        stringify: function (obj) {
            return obj.toString();
        },
        parse: function (str) {
            return new BigNumber(str);
        }
    });
    // Multiple properties of SampleContract are set as storage properties in batches, and the corresponding descriptors use JSON serialization by default
    LocalContractStorage.defineProperties(this, {
        name: null,
        count: null
    });
};
module.exports = SampleContract;
```
After defining the contract, you can directly read and write the binding properties. The corresponding properties will be saved directly, and stored on the chain:

```js
SampleContract.prototype = {
    // Used when the contract first deploys, can not be used a second after the first deploy.
    init: function (name, count, size, value) {
        // Store the data on the chain when deploying the contract
        this.name = name;
        this.count = count;
        this.size = size;
        this.value = value;
    },
    testStorage: function (balance) {
        // value will be read from the storage data on the chain, and automatically converted to Bignumber set according to the descriptor
        var amount = this.value.plus(new BigNumber(2));
        if (amount.lessThan(new BigNumber(balance))) {
            return 0
        }
    }
};
```

##### Storing Map Data
In a smart contract, if you need to store the key-value, you can define the contract property as a Map batch and save it onto the chain. The contract that uses Map can be written as this:

```js
'use strict';

var SampleContract = function () {
    // Set `SampleContract`'s property to `userMap`. Map data then can be stored onto the chain using `userMap`
    LocalContractStorage.defineMapProperty(this, "userMap");

    // Set `SampleContract`'s property to `userBalanceMap`, and custom define the storing and serializtion reading functions.
    LocalContractStorage.defineMapProperty(this, "userBalanceMap", {
        stringify: function (obj) {
            return obj.toString();
        },
        parse: function (str) {
            return new BigNumber(str);
        }
    });

    // Set `SampleContract`'s properties to mulitple map batches
    LocalContractStorage.defineMapProperties(this,{
        key1Map: null,
        key2Map: null
    });
};

SampleContract.prototype = {
    init: function () {
    },
    testStorage: function () {
        // Store the data in userMap and serialize the data onto the chain
        this.userMap.set("robin","1");
        // Store the data into userBalanceMap and save the data onto the chain using a custom serialization function
        this.userBalanceMap.set("robin",new BigNumber(1));
    },
    testRead: function () {
        //Read and store data
        var balance = this.userBalanceMap.get("robin");
        this.key1Map.set("robin", balance.toString());
        this.key2Map.set("robin", balance.toString());
    }
};

module.exports = SampleContract;
```

### Next step: Tutorial 5:

 [Interacting with Nebulas by RPC API](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2005%20Interacting%20with%20Nebulas%20by%20RPC%20API.md)
