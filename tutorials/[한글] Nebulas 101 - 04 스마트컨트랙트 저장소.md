# Nebulas 101 - 04 스마트 컨트랙트 저장소

[유튜브 튜토리얼](https://www.youtube.com/watch?v=Ofs4AyRaSlw)

이전까지 네뷸러스에서 스마트 컨트랙트를 어떻게 작성하고 배포하고 호출하는지에 대해 알아보았습니다.

스마트 컨트랙트 저장소에 대해서 상세하게 소개하겠습니다. 네뷸러스 스마트 컨트랙트는 체인 위에서 데이터 저장 기능을 제공합니다. 전통적인 키-값 저장소 시스템 (예: redis)와 유사하게, 스마트 컨트랙트는 (가스)를 지불함으로써 네뷸러스 위에 저장합니다.

## LocalContractStorage

네뷸러스의 스마트 컨트랙트 환경은 내장된 저장소 객체인 `LocalContractStorage`를 가지고 있고, 이것은 숫자와 문자열 그리고 자바스크립트 객체를 저장할 수 있습니다. 저장된 데이터는 오직 스마트 컨트랙트에서 사용될 수 있습니다. 다른 컨트랙트는 저장된 데이터를 읽을 수 없습니다.

### 기초

`LocalContractStorage` API는 `set`, `get` 그리고 `del`을 가지고 있고, 각각 데이터를 저장하고, 불러오고 삭제할 수 있습니다. 저장소는 숫자, 문자열 그리고 객체가 될 수 있습니다.

#### `LocalContractStorage` 데이터 저장하기:

```js
// 데이터 저장하기. 데이터는 JSON 문자열로 저장될 것입니다.
LocalContractStorage.put(key, value);
// 혹은
LocalContractStorage.set(key, value);
```

#### `LocalContractStorage` 데이터 불러오기

```js
// 키로부터 값을 얻습니다.
LocalContractStorage.get(key);
```

#### Deleting `LocalContractStorage` Data:
#### `LocalContractStorage` 데이터 삭제:

```js
// 데이터 삭제하기, 데이터는 삭제 후에 불러올 수 없습니다.
LocalContractStorage.del(key);
// 혹은 
LocalContractStorage.delete(key);
```

예제:

```js
'use strict';

var SampleContract = function () {
};

SampleContract.prototype = {
    init: function () {
    },
    set: function (name, value) {
        // 문자열 저장하기
        LocalContractStorage.set("name", name);
        // 숫자 저장하기
        LocalContractStorage.set("value", value);
        // 객체 저장하기
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

### 심화

기초적인 `set`, `get`, 그리고 `del` 메소드에 더불어, `LocalContractStorage`는 스마트 컨트랙트의 프로퍼티를 바인딩하기 위해 메소드들을 제공합니다. `get`과 `set`을 하기 위해 `LocalContractStorage` 인터페이스를 호출할 필요 없이 직접적으로 바인딩된 프로퍼티를 읽고 쓸 수 있습니다.

#### 프로퍼티 바인딩

객체 인스턴스, 필드 이름과 descripter는 프로퍼티를 바인딩하기 위해 제공되야 합니다.

##### 인터페이스 바인딩

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

스마트 컨트랙트에서 프로퍼티를 바인딩하는 예제입니다.

```js
'use strict';

var SampleContract = function () {
    // The SampleContract `size` property is a storage property. Reads and writes to `size` will be stored on the chain.
    // The `descriptor` is set to null here, the default JSON.stringify() and JSON.parse() will be used.
    LocalContractStorage.defineMapProperty(this, "size");

    // The SampleContract `value` property is a storage property. Reads and writes to `value` will be stored on the chain.
    // Here is a custom `descriptor` implementation, stored as a string, and returning Bignumber object during parsing. 
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

그리고, 다음 예제에 따라 이러한 프로퍼티를 직접 읽고 쓸 수 있습니다.

```js
SampleContract.prototype = {
    // Used when the contract first deploys, can not be used a second time after the first deployment
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

#### 맵 프로퍼티 바인딩

게다가, `LocalContractStorage`는 맵 프로퍼티를 바인딩하는 메소드를 제공합니다. 맵 프로퍼티를 바인딩하고 스마트 컨트랙트에서 그것을 사용하는 예제입니다.

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

    // Set `SampleContract`'s properties to multiple map batches
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
        // Store the data in userBalanceMap and save the data onto the chain using a custom serialization function
        this.userBalanceMap.set("robin",new BigNumber(1));
    },
    testRead: function () {
        // Read and store data
        var balance = this.userBalanceMap.get("robin");
        this.key1Map.set("robin", balance.toString());
        this.key2Map.set("robin", balance.toString());
    }
};

module.exports = SampleContract;
```

##### 맵 반복하기

컨트랙트에서 맵은 이터레이터를 지원하지 않습니다. 맵을 반복하는게 필요하다면, 다음 방법을 사용할 수 있습니다: `arrayMap`과 `dataMap` 두 개의 맵을 정의합니다. `arrayMap`은 증가하는 카운터로 키를 사용하고, `dataMap`은 데이터 키를 키로 사용합니다.

```js
"use strict";

var SampleContract = function () {
   LocalContractStorage.defineMapProperty(this, "arrayMap");
   LocalContractStorage.defineMapProperty(this, "dataMap");
   LocalContractStorage.defineProperty(this, "size");
};

SampleContract.prototype = {
    init: function() {
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
### 다음 단계: 튜토리얼 5

 [RPC API로 Nebulas와 상호작용하기](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2005%20Interacting%20with%20Nebulas%20by%20RPC%20API.md)
