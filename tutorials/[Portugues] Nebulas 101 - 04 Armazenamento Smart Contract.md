# Nebulas 101 - 04 Armazenamento de Smart Contracts

[Tutorial YouTube](https://www.youtube.com/watch?v=Ofs4AyRaSlw)

Antes discutiu-se como escrever smart contracts e como os implementar e invocar na Nebulas.

Agora vamos introduzir em detalhe o armazenamento de um smart contract. Os “contractos inteligentes” de Nebulas fornecem capacidades de armazenamento de dados on-chain (na blockchain). Semelhantes ao sistema de armazenamento key-value tradicional (ex: redis), smart contracts podem ser armazenados em Nebulas ao pagar com gás (gas).

## LocalContractStorage

O ambiente de armazenamento de Smart Contract de Nebulas vem com armazenamento de objectos embutido, `LocalContractStorage`, que pode armazenar números, cadeias de caracteres (strings), e objectos JavaScript. Dados armazenados apenas podem ser usados em smart contracts. Outros contractos não serão capazes de ler dados armazenados.

### Básicos

O API `LocalContractStorage` inclui `set`, `get` e `del`, que permitem-lhe guardar, ler, e apagar dados. O armazenamento pode conter números, strings, e objectos.

#### Armazenar Dados de `LocalContractStorage`

```js
// armazenar dados. Os dados vão ser guardados como strings JSON
LocalContractStorage.put(key, value);
// Ou
LocalContractStorage.set(key, value);
```

#### Ler Dados de `LocalContractStorage`

```js
// obtém o valor da chave
LocalContractStorage.get(key);
```

#### Apagar Dados de `LocalContractStorage`

```js
// apagar dados, dados não podem ser lidos após a sua eliminação
LocalContractStorage.del(key);
// Ou
LocalContractStorage.delete(key);
```

Exemplos:

```js
'use strict';

var SampleContract = function () {
};

SampleContract.prototype = {
    init: function () {
    },
    set: function (name, value) {
        // Armazenar uma string
        LocalContractStorage.set("name",name);
        // Armazenar um número (valor)
        LocalContractStorage.set("value", value);
        // Armazenar um objecto
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

### Avançado

Além dos métodos básicos `set`, `get`, and `del`, `LocalContractStorage` também fornece métodos para vincular propriedades de smart contracts. Pode ler e escrever propriedades vinculadas directamente sem invocar interfaces `LocalContractStorage` para `get` e `set`.

#### Propriedades Vinculadas

Instância de um objecto,nome de campo, e descritor devem ser fornecidos para vincular propriedades.

##### Interface de Vinculação

```js
// define a propriedade do objecto chamada `fieldname` para `obj` com descritor.
// descritor padrão é o descritor JSON.parse/JSON.stringify.
// retorna
defineProperty(obj, fieldName, descriptor);

// define propriedades do objecto para `obj` de `props`.
// descritor padrão é o descritor JSON.parse/JSON.stringify.
// retorna
defineProperties(obj, descriptorMap);
```

Eis um exemplo para vincular propriedades num smart contract:

```js
'use strict';

var SampleContract = function () {
    // O tamanho da propriedade `size` do SampleContract é uma propriedade armazenada. Lê e escreve para ` size` e será armazenada na chain.
    // O `descritor` está definido como nulo aqui, o JSON.stringify () e JSON.parse () padrão serão utilizados.
    LocalContractStorage.defineMapProperty(this, "size");

    // O valor da propriedade `value` do SampleContract é uma propriedade armazenada. Lê e escreve para `value` e será armazenada na chain.
    // Aqui está uma implementação de `descritor` personalizada, armazenada como uma string, e a retornar o objecto BigNumber durante a análise. 
    LocalContractStorage.defineMapProperty(this, "value", {
        stringify: function (obj) {
            return obj.toString();
        },
        parse: function (str) {
            return new BigNumber(str);
        }
    });
    // Múltiplas propriedades do SampleContract estão definidas como propriedades de armazenamento em quantidade/lotes, e os descritores correspondentes usam serialização JSON por padrão
    LocalContractStorage.defineProperties(this, {
        name: null,
        count: null
    });
};

module.exports = SampleContract;
```

Depois, pode ler e escrever essas propriedades directamente como o seguinte exemplo:

```js
SampleContract.prototype = {
 // Usado quando o contracto é implementado inicialmente, e não pode ser usado uma segunda vez
 init: function (name, count, size, value) {
 // Armazena os dados na chain ao implementar o contracto
 this.name = name;
 this.count = count;
 this.size = size;
 this.value = value;
 },
 testStorage: function (balance) {
 // O valor será lido do armazenamento de dados na chain, e automaticamente convertido num conjunto BigNumber de acordo com o descritor
 var amount = this.value.plus(new BigNumber(2));
 if (amount.lessThan(new BigNumber(balance))) {
 return 0
 }
 }
};
```

#### Vinculação de Propriedades do Mapa

E mais, `LocalContractStorage` também fornece métodos mpara vincular propriedades do mapa. Eis um exemplo para vincular propriedades do mapa e usá-las num smart contract.

```js
'use strict';

var SampleContract = function () {
    // Define a propriedade do `SampleContract` para `userMap`. Dados do mapa podem depois ser armazenados na chain usando `userMap`
    LocalContractStorage.defineMapProperty(this, "userMap");

    // Define a propriedade do `SampleContract` para `userBalanceMap`, e define o armazenamento e serialização das funções de leitura.
    LocalContractStorage.defineMapProperty(this, "userBalanceMap", {
        stringify: function (obj) {
            return obj.toString();
        },
        parse: function (str) {
            return new BigNumber(str);
        }
    });

    // Define as propriedades do `SampleContract` para vários lotes de mapas
    LocalContractStorage.defineMapProperties(this,{
        key1Map: null,
        key2Map: null
    });
};

SampleContract.prototype = {
    init: function () {
    },
    testStorage: function () {
        // Armazena os dados em userMap e serializa os dados na chain
        this.userMap.set("robin","1");
        // Armazena os dados em userBalanceMap e guarda os dados na chain usando uma função de serialização personalizada
        this.userBalanceMap.set("robin",new BigNumber(1));
    },
    testRead: function () {
        // Lê e armazenada dados
        var balance = this.userBalanceMap.get("robin");
        this.key1Map.set("robin", balance.toString());
        this.key2Map.set("robin", balance.toString());
    }
};

module.exports = SampleContract;
```

##### Iterar mapa

No contracto, mapa não suporta iteradores. Se precisar de iterar o mapa, pode usar a seguinte maneira: defina dois mapas, arrayMap e dataMap. ArrayMap com um contador estrictamente crescente como chave, e dataMap com a chave de dados como chave. 

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
### Próximo passo: Tutorial 5

 [Interação com Nebulas por API RPC](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BPortugues%5D%20Nebulas%20101%20-%2005%20Interacao%20com%20Nebulas%20por%20API%20RPC.md)
