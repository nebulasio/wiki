# 智能合约

## 智能合约语言
Nebulas 支持两种智能合约语言：
* JavaScript
* TypeScript

Chrome V8是一个由Chromium开发的广受欢迎的JavaScript执行引擎，用于Google Chrome及Chromium中，Nebulas选用Chrome V8作为这两种智能合约语言的执行引擎。

## 执行模型
智能合约的执行模型如下图:

合约执行步骤：
*  打包合约代码和相关参数到本次Transaction中，部署合约到星云链。
*  合约的执行分为两个阶段：
     * 预处理阶段：注入跟踪指令。
     * 执行阶段：生成可执行代码并运行。

## 合约
Nebulas中的合约与面向对象语言中的类相似，包含状态变量和可以修改这些变量的函数。


### 编写合约
合约必须是JavaScript或TypeScript中的原型对象或类。

合约必须包含一个init函数，只会在部署时执行。 以_开头的函数是私有的，用户无法直接调用。 其他都是公有的，用户可直接调用。

由于合约是在Chrome V8中执行的，因此所有实例变量都在内存中，所以将它们全部保存为[state trie](https://github.com/nebulasio/wiki/blob/master/merkle_trie.md)并不明智。 在Nebulas中，我们提供```LocalContractStorage```和```GlobalContractStorage```对象来帮助开发人员定义需要保存的状态字段。 这些字段应该在合约构造函数中定义。

下面是一个简单的合约例子：
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

### 函数可见性
在JavaScript中，没有函数可见性，原型对象中定义的所有函数都是公有的。

在Nebulas中，我们定义了```public```和```private```两种可见性：

* ```public```
除init函数外，其他函数名与正则表达式 ```^[a-zA-Z$][A-Za-z0-9_$]*$``` 匹配的所有函数都是公有的。公有函数用户可直接调用。

* ```private```
函数名以```_```开头的函数都是私有的。 私有函数只能通过公有函数调用。



## 全局对象
### console
```console``` 模块提供了一个简单的调试控制台，与Web浏览器提供的JavaScript控制台机制类似。
全局控制台可以在不调用 ```require('console')``` 的情况下使用。

#### console.info([...args])
* ```...args <any>```
console.info() 方法是```console.log()```的别名.

#### console.log([...args])
* ```...args <any>```
在```info```级别输出```args```到 Nebulas Logger.

#### console.debug([...args])
* ```...args <any>```
在```debug```级别输出```args```到Nebulas Logger.

#### console.warn([...args])
* ```...args <any>```
在```warn```级别输出```args``` 到Nebulas Logger.

#### console.error([...args])
* ```...args <any>```
在```error```级别输出```args```到Nebulas Logger.

### LocalContractStorage
```LocalContractStorage```模块提供基于状态树的存储功能。 它仅接受字符串键值对。 并且所有数据都存储到与当前合约地址关联的私有状态树中，只有合约可以访问它们。

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
`BigNumber`模块使用 [bignumber.js](https://github.com/MikeMcl/bignumber.js)，这是一个用于任意精度十进制和非十进制算术的JavaScript库。 合约可以直接使用`BigNumber`来处理transaction值。
```js

var value = new BigNumber(0);
value.plus(1);
...
```

### Blockchain
`Blockchain` 为合约提供了一个对象，该对象可以取得当前合约所在的块和Transaction信息，此外该对象提供了transfer方法用于从合约中转出nas，提供了verifyAddress用于地址校验。

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

```

properties:

- `block`: 合约执行的当前块
    - `timestamp`: 块时间戳
    - `seed`: 随机数种子
    - `height`: 块高度
- `transaction`: 合约执行的当前Transaction
    - `hash`: 交易哈希
    - `from`: 交易发送地址
    - `to`: 交易目的地址
    - `value`: 交易金额, 一个BigNumber对象
    - `nonce`: 交易 nonce
    - `timestamp`: 交易时间戳
    - `gasPrice`: gas出价, 一个BigNumber对象
    - `gasLimit`: gas上限值, 一个BigNumber对象
- `transfer(address, value)`: 该函数将来自合约中的NAS发送到目的地址
    - 参数:
        - `address`: 接收NAS的nebulas地址
        - `value`: 交易金额，一个BigNumber对象
    - 返回值:
        - `0`: 交易成功
        - `1`: 交易失败   
- `verifyAddress(address)`: 该函数校验地址
    - 参数:
        - `address`: 需要校验的地址
    - 返回值:
        - `1`: 地址合法
        - `0`: 地址非法

使用样例:

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
    }
};

module.exports = SampleContract;

```


### Event
`Event` 模块记录合约中的执行事件。记录的事件存储在链上的事件树中，事件可在`FetchEvents`方法中通过执行事件的哈希来获取。所有合约事件topic都会加上前缀`chain.contract.`作为最终存储的topic。

你可以在之前的`SampleContract`智能合约中看到相关的使用样例。


### Math.random (当前仅在测试网支持)
* `Math.random()` 返回一个浮点伪随机数，范围从0到1，但不包含1。典型用法是：

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

* `Math.random.seed(myseed)` 可以使用此方法重置随机种子。 参数`myseed` 必须是 **string**。
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

提醒：
* 不支持的方法：`toDateString()`, `toTimeString()`, `getTimezoneOffset()`, `toLocaleXXX()`。
* `new Date()`/`Date.now()` 返回当前块的时间戳，单位为毫秒。
* `getXXX` 返回`getUTCXXX`的结果。


### accept(当前仅在测试网支持) 
该方法支持普通转账及转账到合约地址。`to`是智能合约地址，普通转账能成功的前提是合约声明了函数`accept()`，且该函数执行成功。 如果该转账是非普通转账，则它将被视为普通函数调用。

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



