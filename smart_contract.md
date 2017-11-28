# Smart Contract

## Languages

In Nebulas, there are two supported smart contract languages:
 - [JavaScript](https://en.wikipedia.org/wiki/JavaScript)
 - [TypeScript](https://en.wikipedia.org/wiki/TypeScript)

They are supported by integration of [Chrome V8](https://developers.google.com/v8/) in Nebulas, which is the JavaScript engine in Google Chrome and widely used.

## Execution Model

The diagram below is the Execution Model of Smart Contract:

![Smart Contract Execution Model](resources/smart_contract_execution_model.png "Smart Contract Execution Model")

1. All src of Smart Contract and calling parameters are packaged in Transaction and be deployed on Nebulas.
2. The execution of Smart Contract are divided into two phase:
    1. Preprocess: inject tracing instruction, etc.
    2. Execute: generate executable src and execute it.

## Contracts

Contracts in Nebulas are similar to classes in object-oriented languages. They contains persistent data in state variables and functions that can modify these variables.


### Writing Contract

Contract must be a Prototype Object or Class in JavaScript or TypeScript.

A Contract must have ```init``` function, it will be executed only once, and be executed while deploying. Function name starts with ```_``` is ```private``` function, can't be executed in Transaction. The other functions are all ```public``` function, can be executed in Transaction.

Since Contract is executed in Chrome V8, all instance variables are in memory, it's not wise to save all of them to [state trie]() in Nebulas. In Nebulas, we provide ```LocalContractStorage``` and ```GlobalContractStorage``` objects to help developers define fields needing to save to state trie. And those fields should be defined in ```constructor``` of Contract, before other functions.

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
All functions that name matches regexp ```^[a-zA-Z$][A-Za-z0-9_$]*$``` are Public function, except ```init```. Public function can be called via Transaction.

* ```private```
All functions that name starts with ```_``` are Private function. Private function can only be called by Public function.

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

The ```LocalContractStorage``` module provides a state trie based storage capability. It accepts string only key value pairs. And all data are stored to a private state trie associate with current contract address, only the contract can access that their data.

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

### Blockchain
