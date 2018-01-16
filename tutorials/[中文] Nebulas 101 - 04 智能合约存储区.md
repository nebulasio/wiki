# Nebulas 101 - 04 智能合约存储区

前面我们介绍了怎么编写智能合约以及怎样在星云链部署和调用智能合约。

今天我们来详细的介绍有关星云链智能合约的存储。星云链智能合约(smart contract)提供了链上数据存储功能。类似于传统的key-value存储系统（eg:redis），可以付费（消耗gas）将数据存储到星云链上。

## LocalContractStorage存储使用存储介绍
星云链的智能合约运行环境内置了存储对象`LocalContractStorage`,可以存储数字，字符串，JavaScript对象，存储数据只能在智能合约内使用，其他合约不能读取存储的内容。

#### 基础用法
`LocalContractStorage`的简单接口包括`set`,`get`,`del`接口，实现了存储，读取，删除数据功能。存储可以是数字，字符串，对象。

##### `LocalContractStorage `存储数据：

```js
// 存储数据，数据会被json序列化成字符串保存
LocalContractStorage.put(key, value);
LocalContractStorage.set(key, value);
```
##### `LocalContractStorage `读取数据：
```js
// 获取数据
LocalContractStorage.get(key);
```
##### `LocalContractStorage `删除数据：
```js
// 删除数据, 数据删除后无法读取
LocalContractStorage.del(key);
LocalContractStorage.delete(key);
```
下面是一些使用例子：

```js
'use strict';

var SampleContract = function () {
};

SampleContract.prototype = {
    init: function () {
    },
    set: function (name, value) {
        // 存储字符串
        LocalContractStorage.set("name",name);
        // 存储数字
        LocalContractStorage.set("value", value);
        // 存储对象
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

#### 高级用法
`LocalContractStorage`除了基本的`set`,`get`,`del`方法，还支持向对象设置存储属性和存储map，并支持设置存储时的序列化方法。

##### 存储属性
在写智能合约时可以将合约的属性设为设为存储位，使用时对属性的读写会直接使用`LocalContractStorage`的读写。读写时还可以自定义属性的字符和对象映射关系。一般为合约做存储位时在初始化中将属性设置为存储属性。

`LocalContractStorage`的属性方法定义：

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

使用`LocalContractStorage`做合约的属性绑定例子:

```js
'use strict';
var SampleContract = function () {
    // SampleContract的`size`属性为存储属性，对`size`的读写会存储到链上，
    // 此处的`descriptor`设置为null，将使用默认的JSON.stringify()和JSON.parse()
    LocalContractStorage.defineMapProperty(this, "size", null);

    // SampleContract的`value`属性为存储属性，对`value`的读写会存储到链上，
    // 此处的`descriptor`自定义实现，存储时直接转为字符串，读取时获得Bignumber对象
    LocalContractStorage.defineMapProperty(this, "value", {
        stringify: function (obj) {
            return obj.toString();
        },
        parse: function (str) {
            return new BigNumber(str);
        }
    });
    // SampleContract的多个属性批量设置为存储属性，对应的descriptor默认使用JSON序列化
    LocalContractStorage.defineProperties(this, {
        name: null,
        count: null
    });
};
module.exports = SampleContract;
```
在定义完合约后，可以直接对绑定的属性做读写操作，对应的属性就会直接保存，并存储到链上：

```js
SampleContract.prototype = {
    // 合约部署时调用，部署后无法二次调用
    init: function (name, count, size, value) {
        // 在部署合约时将数据存储到链上
        this.name = name;
        this.count = count;
        this.size = size;
        this.value = value;
    },
    testStorage: function (balance) {
        // 使用value时会从存储中读取链上数据，并根据descriptor设置自动转换为Bignumber
        var amount = this.value.plus(new BigNumber(2));
        if (amount.lessThan(new BigNumber(balance))) {
            return 0
        }
    }
};
```

##### 存储Map数据

在智能合约中，如果需要存储key-value时，可以将合约属性定义为Map集合，并保存到链上。使用Map的合约可以用下面的方式来写：

```js
'use strict';

var SampleContract = function () {
    // 为`SampleContract`定义`userMap`的属性集合，数据可以通过`userMap`存储到链上
    LocalContractStorage.defineMapProperty(this, "userMap");

    // 为`SampleContract`定义`userBalanceMap`的属性集合，并且存储和读取序列化方法自定义
    LocalContractStorage.defineMapProperty(this, "userBalanceMap", {
        stringify: function (obj) {
            return obj.toString();
        },
        parse: function (str) {
            return new BigNumber(str);
        }
    });

    // 为`SampleContract`定义多个集合
    LocalContractStorage.defineMapProperties(this,{
        key1Map: null,
        key2Map: null
    });
};

SampleContract.prototype = {
    init: function () {
    },
    testStorage: function () {
        // 将数据存储到userMap中，并序列化到链上
        this.userMap.set("robin","1");
        // 将数据存储到userBalanceMap中，使用自定义序列化函数，保存到链上
        this.userBalanceMap.set("robin",new BigNumber(1));
    },
    testRead: function () {
        //读取存储数据
        var balance = this.userBalanceMap.get("robin");
        this.key1Map.set("robin", balance.toString());
        this.key2Map.set("robin", balance.toString());
    }
};

module.exports = SampleContract;
```


