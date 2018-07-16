# 智能合约

## 智能合约语言
Nebulas支持两种智能合约语言：
* JavaScript
* TypeScript

Chrome V8是一个由Chromium开发的广受欢迎的JavaScript执行引擎，用于Google Chrome及Chromium中，Nebulas选用Chrome V8作为这两种智能合约语言的执行引擎。

## 执行模型
智能合约的执行模型如下图:

![Smart Contract Execution Model](resources/smart_contract_execution_model.png "Smart Contract Execution Model")

合约执行步骤：
*  打包合约代码和相关参数到本次Transaction中，部署合约到星云链。
*  合约的执行分为两个阶段：
     * 预处理阶段：注入跟踪指令。
     * 执行阶段：生成可执行代码并运行。

## 合约
Nebulas中的合约与面向对象语言中的类相似，包含状态变量和可以修改变量的函数。


### 编写合约
合约必须是JavaScript或TypeScript中的原型对象或类。

合约必须包含一个init函数，只会在部署合约时执行。以_开头的函数是私有的，用户无法直接调用。其他都是公有的，用户可直接调用。

由于合约是在Chrome V8中执行的，因此所有实例变量都在内存中，所以将它们全部保存为[state trie](https://github.com/nebulasio/wiki/blob/master/merkle_trie.md)并不明智。在Nebulas中，我们提供```LocalContractStorage```和```GlobalContractStorage```对象来帮助开发人员定义需要保存的状态字段。这些字段应该在合约构造函数中定义。

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

在Nebulas中，我们定义了```public```和```private```两种可见性：

* ```public```
除init函数外，其他函数名与正则表达式 ```^[a-zA-Z$][A-Za-z0-9_$]*$``` 匹配的所有函数都是公有的。公有函数用户可直接调用。

* ```private```
函数名以```_```开头的函数都是私有的。私有函数只能通过公有函数调用。



## 全局对象
### console
```console```模块提供了一个简单的调试控制台，与Web浏览器提供的JavaScript控制台机制类似。全局控制台可以在不调用```require('console')```的情况下使用。

#### console.info([...args])
* ```...args <any>```
console.info() 方法是```console.log()```的别名.

#### console.log([...args])
* ```...args <any>```
在```info```级别输出```args```到Nebulas Logger.

#### console.debug([...args])
* ```...args <any>```
在```debug```级别输出```args```到Nebulas Logger.

#### console.warn([...args])
* ```...args <any>```
在```warn```级别输出```args```到Nebulas Logger.

#### console.error([...args])
* ```...args <any>```
在```error```级别输出```args```到Nebulas Logger.

### LocalContractStorage
```LocalContractStorage```模块提供基于状态树的存储功能。它仅接受字符串键值对。并且所有数据都存储到与当前合约地址关联的私有状态树中，只有合约可以访问它们。

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
`BigNumber`模块使用[bignumber.js](https://github.com/MikeMcl/bignumber.js)(v4.1.0)，这是一个用于任意精度十进制和非十进制算术的JavaScript库。合约可以直接使用`BigNumber`来处理transaction值。
```js

var value = new BigNumber(0);
value.plus(1);
...
```

### Blockchain
`Blockchain`为合约提供了一个对象，该对象可以取得当前合约所在的块和Transaction信息，此外该对象提供了transfer方法用于从合约中转出nas，提供了verifyAddress用于地址校验。

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

- `block`: 合约执行的当前块
    - `timestamp`: 块时间戳
    - `seed`: 随机数种子,从1.1.0开始返回`""`
    - `height`: 块高度
- `transaction`: 合约执行的当前Transaction
    - `hash`: 交易哈希
    - `from`: 交易发送地址
    - `to`: 交易目的地址
    - `value`: 交易金额, 一个BigNumber对象
    - `nonce`: 交易nonce
    - `timestamp`: 交易时间戳
    - `gasPrice`: gas出价, 一个BigNumber对象
    - `gasLimit`: gas上限值, 一个BigNumber对象
- `transfer(address, value)`: 该函数将来自合约中的NAS发送到目的地址
    - 参数:
        - `address`: 接收NAS的nebulas地址
        - `value`: 交易金额，一个BigNumber/Uint(推荐使用)对象；单位为wei，所以只能是整数，用小数会失败。
    - 返回值(布尔型):
        - `true`: 交易成功
        - `false`: 交易失败   
- `verifyAddress(address)`: 该函数校验地址
    - 参数:
        - `address`: 需要校验的地址
    - 返回值(数字型):
        - `87`: 用户钱包地址
        - `88`: 合约地址
        - `0`: 地址非法
- `getAccountState(address)`（testnet): 获取账户的余额和nonce
	- 参数:
		- `address`: 想要获取余额的地址
	- 返回值 (JSON 对象):
		- `balance`: 账户的余额
		- `nonce`: 账户的nonce
- `getPreBlockHash(offset)`（testnet): 得到之前的块的哈希
	- 参数:
		- `offset`: 想要查询的块的高度和当前高度的偏移量。这个参数必须是整型，且大于0，小于当前高度。如果offset为1，表示上一个区块。
	- 返回值(string 类型):
		- `hash`: 区块哈希
- `getPreBlockSeed(offset)`（testnet): 获取先前区块的随机种子
	- 参数:
		- `offset`: 和 Blockchain.getPreBlockHash() 中参数类似
	- 返回值(string 类型):
		- `seed`: 区块的随机种子

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
`Event`模块记录合约中的执行事件。记录的事件存储在链上的事件树中。用户可以使用[`GetEventsByHash`](https://github.com/nebulasio/wiki/blob/master/rpc.md#geteventsbyhash)通过当前交易哈希来获取该事件。所有合约事件的topic都会在合约代码中指定的topic前加上前缀`chain.contract.`作为最终存储的topic。

```js
Event.Trigger(topic, obj);
```

 - `topic`: user-defined topic
 - `obj`: JSON object

你可以在刚才的`SampleContract`智能合约例子中看到相关的使用样例。

### Math.random 
* `Math.random()`返回一个浮点伪随机数，范围从0到1，但不包含1。典型用法是：

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

* `Math.random.seed(myseed)`可以使用此方法重置随机种子。参数`myseed`必须是 **string**。**注意** 从1.1.0开始此方法被移除。
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
所有标准API均可使用。Date对象的Timezone固定为`UTC+0`，Locale固定为`en-US`。
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
* `new Date()`/`Date.now()`返回当前块的时间戳，单位为毫秒。

### accept
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

### Uint

`Uint`库基于bignumber.js封装了4种无符号整型：`Uint64`、`Uint128`、`Uint256`、`Uint512`。

静态属性：

- `MaxValue`: 实例对象，表示具体类型的最大有效值

实例方法:

- `div(o)`：求商
    - 参数:
        - `o`: 除数, 类型必须和被除数一致
    - 返回：运算结果，类型同被除数

- `pow(o)`：幂
    - 参数:
        - `o`: 指数, 类型必须和底数一致
    - 返回：运算结果，类型同底数

- `minus(o)`：减
    - 参数:
        - `o`: 减数, 类型必须和被减数一致
    - 返回：运算结果，类型同被减数

- `mod(o)`：取模
    - 参数:
        - `o`: 模数, 类型必须和被模数一致
    - 返回：运算结果，类型同被模数

- `mul(o)`：乘
    - 参数:
        - `o`: 乘数, 类型必须和被乘数一致
    - 返回：运算结果，类型同被乘数

- `plus(o)`：加
    - 参数:
        - `o`: 加数, 类型必须和被加数一致
    - 返回：运算结果，类型同被加数

- `cmp(o)`：大小比较
    - 参数:
        - `o`: 类型必须和`this`一致
    - 返回值:
        - `1`: `this` 大于 `o`
        - `0`: `this` 等于 `o`
        - `-1`: `this` 小于 `o`

- `isZero()`：零值判断
    - 返回值:
        - `true`: `this` 为 0
        - `false`: `this` 不为 0

- `toString(base)`:  以字符形式输出`this`值
    - 参数:
        - `base`: 以2 ~ 64进制格式输出, 默认10进制

使用示例：

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

`require`用于加载那些NVM启动时没有装载的第三方库. 

当前可用的第三方库有:

* `crypto.js`

典型用法:
    
```js
    var crypto = require('crypto.js');
    ...
```

### crypto

`crypto`库提供了常用的哈希函数，需要在合约中使用`require`显式加载。

APIs:

- `sha256(str)`
    - 参数:
        - `str`: 大小写敏感字符串
    - 返回值: 
        - 16进制字符串, 长度64

- `sha3256(str)`
    - 参数:
        - `str`: 大小写敏感字符串
    - 返回值: 
        - 16进制字符串, 长度64

- `ripemd160(str)`
    - 参数:
        - `str`: 大小写敏感字符串
    - 返回值: 
        - 16进制字符串, 长度40

- `md5(str)`
    - 参数:
        - `str`: 大小写敏感字符串
    - 返回值: 
        - 16进制字符串, 长度32

- `base64(str)`
    - 参数:
        - `str`: 大小写敏感字符串
    - 返回值: 
        - base64编码字符串

- `recoverAddress(alg, hash, sign)`: 使用公钥数据解码出签名钱包地址
    - 参数:
        - `alg`: 使用的签名算法，当前只有一个有效值`1`，表示使用的是secp256k1
        - `hash`: 被签名数据，为长度64的16进制字符串
        - `sign`: 使用私钥对`hash`签名得到的16进制字符串
    - 返回值: 
        - 星云钱包地址，如果失败返回`null`

使用示例:
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

### 合约间调用（1.1.0开始支持，测试网）

我们在智能合约中提供了一个简单的方法来调用另一个智能合约，下面的示例展示了proxyKvStore如何使用合约kvStore提供服务。

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
        return kvStore.call("get", key);
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

在上面的例子中，为了使用kvStore.js， 我们首先通过kvStore的合约地址创建一个合约对象：

```js
var kvStore  = new Blockchain.Contract(address);
```

随后我们就可以通过这个对象来调用kvStore这个智能合约：
```js
kvStore.value(2000000000000000000).call("save", key, value);
```
或者：
```js
kvStore.call("save", key, value);
```
'value'函数决定了多少nas会被从调用者智能合约转到被调用调用的智能合约。这个函数不是必须的，缺省值是0.

值的一提的是，在被调用合约的执行环境中，Blockchain.from 是调用合约的地址，Blockchain.value的值则是由调用合约执行的'value'函数的参数决定；被调用合约如果中如果throw错误，调用合约无法catch，所有修改的状态会被回滚。
