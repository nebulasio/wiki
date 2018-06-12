# Nebulas 101 - 04 Almacenaje de Smart Contract

Anteriormente nos cubrimos cómo escribir smart contratos y cómo implementarlos en el Nebulas.

Ahora les presentamos en detalle el almacenaje del smart contrato. Smart contratos de Nebulas proporcian en-chain capacidades de almacenja de datos. Similar al tradicional systema de clave-valor almacenaje (eg: redis), smart contratos se pueden almacenar en Nebulas por pagando gas.


## LocalContractStorage

El ambiante de smart contrato de Nebulas ha integrado objeto almacenaje `LocalContractStorage`,  que  puede guardar numeros, cuerdas, y objetos de Javascript. Los datos almacenados solo puden usarse en smart contratos. Otros contratos no pueden leer los datos almacenados.


### Basicos

`LocalContractStorage` API incluye `set`, `get` y `del`, que permite almancenar, leer, y eliminar datos. Almacenaje puede ser numeros, cuerdas y objetos..

#### Almancene `LocalContractStorage` Datos:

```js
// almacene dato. Los datos se almacenarán como JSON cuerdas.
LocalContractStorage.put(key, value);
// Or
LocalContractStorage.set(key, value);
```

#### Leyendo `LocalContractStorage` Datos:

```js
// obtene el valor de clave
LocalContractStorage.get(key);
```

#### Eliminar `LocalContractStorage` Datos:

```js
// eliminar datos, los datos no se pueden leer despues de la elminación.
LocalContractStorage.del(key);
// Or
LocalContractStorage.delete(key);
```

Ejemplos:

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

### Avanzado

Además de los métodos basicos  `set`, `get`, and `del`, `LocalContractStorage` también proporciona métodos para enlazar propiedades de smart contratos. Podemos leer y escribir binding propiedades sin directamente invocar `LocalContractStorage` interfaz para `get` y `set.`

#### Binding propiedades

Object instance, field name y descriptor se debe proporcionar para bind propiedades.

##### Bind Interfaz

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

Ahí es un ejemplo para unir esteas propiedades en un smart contrato.

```js
'use strict';

var SampleContract = function () {
    // El SampleContract `size` propiedad es un almacenaje propiedad. Lee y escribe a `size` se almencará en la cadena.
    // El `descriptor` es null ahí, la predeterminada JSON.stringify() y JSON.parse() serán usados.
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

Despues, podems leer y escribir estes propiedades directamente como el siguiente ejemplo:

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

#### Binding Map Properties

`LocalContractStorage` también proporciona métodos para bind map propiedades.
Ahí es un ejemplo para bind map propiedades y usarlos en el smart contrato.

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

##### Iterate Map
En contrato, map no apoya iterators. Si necesitas iterate map, se necesitas iterate el map, puedes usar esta manera: define two maps, arrayMap, y dataMap. arrayMap con subindo contador como key, dataMap with data key como key.

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
