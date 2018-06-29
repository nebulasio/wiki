# Nebulas 101 - 04 スマート・コントラクトメモリ

[YouTube チュートリアル](https://www.youtube.com/watch?v=Ofs4AyRaSlw)

前にNebulasでスマートコントラクトをどう書くか、配備するかと呼び出すかをカバーした。

今スマートコントラクトのメモリを詳しく紹介する。Nebulasのスマートコントラクトはオンチェーンデータストレージ機能を提供している。伝統的なキー-値ストレージシステム(例: redis)に似ていて、(gas)を払うことを通じて、スマートコントラクトがNebulasに保存されることができる。

## ローカルコントラクトストレージ

Nebulasのスマートコントラクト環境はストレージオブジェクト`LocalContractStorage`にビルドインしている。これが数字、文字列とJavaScriptオブジェクトを保存することができる。保存されたデータはスマートコントラクトの使用に限る。他のコントラクトがこれらの保存するデータを読み取ることができない。

### 基礎

`LocalContractStorage`のAPIは `set`、`get`と`del`を含めている。これによってデータの保存、読み取りと削除が可能である。ストレージには数字、文字列とオブジェクトが使用できる。

#### `LocalContractStorage` データを保存する:

```js
// データを保存。データがJSON文字列に保存される
LocalContractStorage.put(key, value);
// もしくは
LocalContractStorage.set(key, value);
```

#### `LocalContractStorage` データを読み取る:

```js
// キーから値を取得する
LocalContractStorage.get(key);
```

#### `LocalContractStorage` データを削除する:

```js
// データを削除、削除したのデータが読み取られることができない
LocalContractStorage.del(key);
// もしくは
LocalContractStorage.delete(key);
```

実例:

```js
'use strict';

var SampleContract = function () {
};

SampleContract.prototype = {
    init: function () {
    },
    set: function (name, value) {
        // 文字列を保存
        LocalContractStorage.set("name",name);
        // ナンバー(値)を保存
        LocalContractStorage.set("value", value);
        // オブジェクトを保存
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

### 高級

基礎の`set`、`get`と`del`方法に加えて、`LocalContractStorage` はスマートコントラクトのプロパティーをバインドする方法も提供する。`LocalContractStorage` インターフェースを呼び出して`get`と`set`を無しに、直接にバインドしたプロパティーを読み取ると書くことができる。

#### バインディングプロパティー

オブジェクトインスタンス、フィールド名と記述子がプロパティーをバインドすることに提供されるすべき。

##### バインディングインターフェース

```js
// オブジェクトプロパティーを定義して記述子で`fieldname`から`obj`に名をつく。
// デフォルトdescriptorは JSON.parse/JSON.stringify記述子。
// これをリターンする。
defineProperty(obj, fieldName, descriptor);

// `props`から`obj`にオブジェクトプロパティーを定義する。
// デフォルトdescriptorは JSON.parse/JSON.stringify記述子。
// これをリターンする。
defineProperties(obj, descriptorMap);
```

ここは一つの例としてプロパティーをスマートコントラクトにバインドする。

```js
'use strict';

var SampleContract = function () {
    // SampleContractの`size`プロパティーはストレージプロパティー。`size`に読み取ると書くことがチェーンに保存される。
    // `descriptor`はここにヌルに設置している。デフォルトのJSON.stringify()とJSON.parse()が使用される。
    LocalContractStorage.defineMapProperty(this, "size");

    // SampleContractの`value`プロパティーはストレージプロパティー。`value`に読み取ると書くことがチェーンに保存される。
    // ここにはカスタマイズ`descriptor`の実装。文字列に保存され、解析中にはビッグナンバーオブジェクトをリターンする。
    LocalContractStorage.defineMapProperty(this, "value", {
        stringify: function (obj) {
            return obj.toString();
        },
        parse: function (str) {
            return new BigNumber(str);
        }
    });
    // SampleContractの多数のプロパティーがバッチでストレージプロパティーに設置されている。対応する記述子はデフォルトでJSONシリアライゼーションを使用する。
    LocalContractStorage.defineProperties(this, {
        name: null,
        count: null
    });
};

module.exports = SampleContract;
```

次いで、以下の例のように、これらのプロパティーを直接に読み取ると書くことができる。

```js
SampleContract.prototype = {
    // コントラクトを初めに配備するときに使用する。最初配備するあと、もう一度使用することはできない
    init: function (name, count, size, value) {
        // コントラクトを配備するときにデータをチェーンに保存する
        this.name = name;
        this.count = count;
        this.size = size;
        this.value = value;
    },
    testStorage: function (balance) {
        // 値はチェーン上のストレージデータから読み取られる。そして記述子によって自動的にビッグナンバーセットに変換される
        var amount = this.value.plus(new BigNumber(2));
        if (amount.lessThan(new BigNumber(balance))) {
            return 0
        }
    }
};
```

#### マッププロパティーをバインディングする

その上に、 `LocalContractStorage` はマッププロパティーをバインドする方法も提供する。ここは一つの例としてマッププロパティーをバインドしてスマートコントラクトに使用する。

```js
'use strict';

var SampleContract = function () {
    // `SampleContract`のプロパティーを`userMap`にセットする。`userMap`を使用してマップデータをチェーンに保存することができる
    LocalContractStorage.defineMapProperty(this, "userMap");

    // `SampleContract`のプロパティーを`userBalanceMap`にセットして、ストレージ関数とシリアル化の読み取る関数をカスタマイズに定義する。
    LocalContractStorage.defineMapProperty(this, "userBalanceMap", {
        stringify: function (obj) {
            return obj.toString();
        },
        parse: function (str) {
            return new BigNumber(str);
        }
    });

    // `SampleContract`のプロパティーを複数のマップバッチにセットする
    LocalContractStorage.defineMapProperties(this,{
        key1Map: null,
        key2Map: null
    });
};

SampleContract.prototype = {
    init: function () {
    },
    testStorage: function () {
        // userMapにデータを保存してチェーン上にデータをシリアライズする
        this.userMap.set("robin","1");
        // userBalanceMapにデータを保存してカスタマイズシリアル化関数を使用してデータをチェーン上にセーブする 
        this.userBalanceMap.set("robin",new BigNumber(1));
    },
    testRead: function () {
        // データを読み取り、保存する
        var balance = this.userBalanceMap.get("robin");
        this.key1Map.set("robin", balance.toString());
        this.key2Map.set("robin", balance.toString());
    }
};

module.exports = SampleContract;
```

##### 繰り返しのマップ

コンタクトでのマップはイテレートを対応しない。マップを往復処理する必要があれば、以下の方法を使用する: 二つのマップarrayMapとdataMapを定義する。厳密に増加しているキーとするカウンターarrayMapとキーとするデータキーdataMap。

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
### 次の章: チュートリアル 5

 [RPC APIを通じてNebulasに接続](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2005%20Interacting%20with%20Nebulas%20by%20RPC%20API.md)
