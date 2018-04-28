# Nebulas 101 - 04 Smart Contract Storage

[YouTube Tutorial](https://www.youtube.com/watch?v=Ofs4AyRaSlw)

Earlier we covered how to write smart contracts and how to deploy and invoke smart contracts in the Nebulas.

Now we introduce in detail the storage of the smart contract. Nebulas smart contracts provide on-chain data storage capabilities. Similar to the traditional key-value storage system (eg: redis), smart contracts can be stored on the Nebulas by paying with (gas).

## LocalContractStorage

Nebulas' Smart Contract environment has built-in storage object `LocalContractStorage`, which can store numbers, strings, and JavaScript objects. The stored data can only be used in smart contracts. Other contracts can not read the stored data.

### Basics

The `LocalContractStorage` API includes `set`, `get` and `del`, which allow you to store, read, and delete data. Storage can be numbers, strings, objects

#### Storing `LocalContractStorage` Data：

```js
// store data. The data will be stored as JSON strings
LocalContractStorage.put(key, value);
// Or
LocalContractStorage.set(key, value);
```

#### Reading `LocalContractStorage ` Data：

```js
// get the value from key
LocalContractStorage.get(key);
```

#### Deleting `LocalContractStorage` Data：

```js
// delete data, data can not be read after deletion
LocalContractStorage.del(key);
// Or
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
        LocalContractStorage.set("obj", {name:name, value:value});
    },
    get: function () {
        var name = LocalContractStorage.get("name");
        console.log("name:" + name)
        var value = LocalContractStorage.get("value");
        console.log("value:" + value)
        var obj = LocalContractStorage.get("obj");
        console.log("obj:" + JSON.stringify(obj))
    },
    del: function () {
        var result = LocalContractStorage.del("name");
        console.log("del result:" + result)
    }
};

module.exports = SampleContract;
```

### Advanced

In addition to the basic `set`, `get`, and `del` methods, `LocalContractStorage` also provides methods to bind properties of smart contracts. We could read and write binded properties directly without invoking `LocalContractStorage` interfaces to `get` and `set`.

#### Binding Properties

Object instance, field name and descriptor should be provided to bind properties.

##### Binding Interface

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

Here is an example to bind properties in a smart contract.

```js
'use strict';

var SampleContract = function () {
    // The SampleContract `size` property is a storage property. Reads and writes to` size` will be stored on the chain.
    // The `descriptor` is set to null here, the default JSON.stringify () and JSON.parse () will be used.
    LocalContractStorage.defineMapProperty(this, "size");

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

Then, we can read and write these properties directly as the following example.

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

#### Binding Map Properties

What's more, `LocalContractStorage` also provides methods to bind map properties. Here is an example to bind map properties and use them in a smart contract.

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

##### Iterate Map 
In  contract, map does't support iterator. if you need to iterate the map, you can use the following way: define two map, arrayMap, dataMap, arrayMap with a strictly increasing counter as key, dataMap with data key as key. 

```js
"use strict";

var SampleContract = function () {
   LocalContractStorage.defineMapProperty(this, "arrayMap");
   LocalContractStorage.defineMapProperty(this, "dataMap");
   LocalContractStorage.defineProperty(this, "size");
};

SampleContract.prototype = {
    init: function () {
        this.size = 0;
    },
    
    set: function (key, value) {
        var index = this.size;
        this.arrayMap.set(index, key);
        this.dataMap.set(key, value);
        this.size +=1;
    },
    
    get: function (key) {
        return this.dataMap.get(key);
    },

    len:function(){
      return this.size;
    },
    
    iterate: function(limit, offset){
        limit = parseInt(limit);
        offset = parseInt(offset);
        if(offset>this.size){
           throw new Error("offset is not valid");
        }
        var number = offset+limit;
        if(number > this.size){
          number = this.size;
        }
        var result  = "";
        for(var i=offset;i<number;i++){
            var key = this.arrayMap.get(i);
            var object = this.dataMap.get(key);
            result += "index:"+i+" key:"+ key + " value:" +object+"_";
        }
        return result;
    }
    
};

module.exports = SampleContract;
```
### Next step: Tutorial 5

 [Interacting with Nebulas by RPC API](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2005%20Interacting%20with%20Nebulas%20by%20RPC%20API.md)
