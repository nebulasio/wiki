# What Is Permission Control Of Smart Contract
The permission control of a smart contract refers to whether the contract caller has permission to invoke the function in the contract. There are two types of permission control: owner permission control and other permission control.

Owner permissions control: Only the creator of the contract can call this method, other callers can not call the method.

Other permission control: The contract method can be invoked if the contract developer defines a conditional caller according to the contract logic. Otherwise, it cannot be invoked.


# Owner Permission Control

If you want to specify a owner to a small contract and wish if some functions could be called only by the owner and no one else. You can use following lines of code in your smart contract.

```javascript
"use strict";
var onlyOwnerContract = function () {
	LocalContractStorage.defineProperty(this, "owner");
};
onlyOwnerContract.prototype = {
  init: function() {
  		this.owner=Blockchain.transaction.from;
  },
  onlyOwnerFunction: function(){
  		if(this.owner==Blockchain.transaction.from){
  			//your smart contract code
  			return true;
  		}else{
  			return false;
  		}
  }
};
module.exports = BankVaultContract;
```

Explanation:

The function init is only called once when the contract is deployed, there you specify the owner of the contract.The onlyOwnerFunctiuon ensures that the function is called by the owner of contact.

# Other Permission Control

In your smart contract, if you need to specify other permission control to a smart contract. For example, in your smart contract, you need to verify the transaction value in your smart contract. you can write your smart contract in the following way.

```javascript
'use strict';
var Mixin = function () {};
Mixin.UNPAYABLE = function () {
   if (Blockchain.transaction.value.gt(0)) {
       return false;
   }
   return true;
};
Mixin.PAYABLE = function () {
   if (Blockchain.transaction.value.gt(0)) {
       return true;
   }
   return false;
};
Mixin.POSITIVE = function () {
   console.log("POSITIVE");
   return true;
};
Mixin.UNPOSITIVE = function () {
   console.log("UNPOSITIVE");
   return false;
};
Mixin.decorator = function () {
   var funcs = arguments;
   if (funcs.length < 1) {
       throw new Error("mixin decorator need parameters");
   }
   return function () {
       for (var i = 0; i < funcs.length - 1; i ++) {
           var func = funcs[i];
           if (typeof func !== "function" || !func()) {
               throw new Error("mixin decorator failure");
           }
       }
       var exeFunc = funcs[funcs.length - 1];
       if (typeof exeFunc === "function") {
           exeFunc.apply(this, arguments);
       } else {
           throw new Error("mixin decorator need an executable method");
       }
   };
};
var SampleContract = function () {
};
SampleContract.prototype = {
   init: function () {
   },
   unpayable: function () {
       console.log("contract function unpayable:", arguments);
   },
   payable: Mixin.decorator(Mixin.PAYABLE, function () {
       console.log("contract function payable:",arguments);
   }),
   contract1: Mixin.decorator(Mixin.POSITIVE, function (arg) {
       console.log("contract1 function:", arg);
   }),
   contract2: Mixin.decorator(Mixin.UNPOSITIVE, function (arg) {
       console.log("contract2 function:", arg);
   }),
   contract3: Mixin.decorator(Mixin.PAYABLE, Mixin.POSITIVE, function (arg) {
       console.log("contract3 function:", arg);
   }),
   contract4: Mixin.decorator(Mixin.PAYABLE, Mixin.UNPOSITIVE, function (arg) {
       console.log("contract4 function:", arg);
   })
};
module.exports = SampleContract;
```

Explanation:

Mixin.UNPAYABLE,Mixin.PAYABLE,Mixin.POSITIVE ,Mixin.UNPOSITIVE  are  permission control functions。The permission control functions as follows:
* Mixin.UNPAYABLE: check the transaction sent value, if value is less than 0 return true, otherwise returns false
* Mixin.PAYABLE: check the transaction sent value, if value is greater than 0 return true, otherwise returns false
* Mixin.UNPOSITIVE: output log UNPOSITIVE
* Mixin.POSITIVE: output log POSITIVE

Implement permission control in Mixin.decorator：

* check arguments: if (funcs.length < 1)
* invoke permission control function: if (typeof func !== "function" || !func()) 
* if permission control function success ,invoke other function: var exeFunc = funcs[funcs.length - 1]

Permission control tests in smart contracts are as follows:

* The permission control function of the contract1 is Mixin.POSITIVE. If the permission check passes, the output is printed, otherwise the error is thrown by the permission check function.
```javascript
        contract1: Mixin.decorator(Mixin.POSITIVE, function (arg) {
            console.log("contract1 function:", arg);
        })
```

* The permission control function of the contract2 is Mixin.UNPOSITIVE. If the permission check passes, the output is printed, otherwise the error is thrown by the permission check function.
```javascript
        contract2: Mixin.decorator(Mixin.UNPOSITIVE, function (arg) {
       	       console.log("contract2 function:", arg);
        })
```

* The permission control function of the contract3 is Mixin.PAYABLE, Mixin.POSITIVE.  If the permission check passes, the output is printed, otherwise the error is thrown by the permission check function.
```javascript
       contract3: Mixin.decorator(Mixin.PAYABLE, Mixin.POSITIVE, function (arg) {
       	       console.log("contract3 function:", arg);
        })
```

* The permission control function of the contract4 is Mixin.PAYABLE, Mixin.UNPOSITIVE.  If the permission check passes, the output is printed, otherwise the error is thrown by the permission check function.
```javascript
        contract4: Mixin.decorator(Mixin.PAYABLE, Mixin.UNPOSITIVE, function (arg) {
                   console.log("contract4 function:", arg);
        })
```

Tips:

With reference to the above example, the developer needs only three steps to implement other permission controls:
* Implement permission control functions.
* Implement the decorator function, and the permission check is completed by the conditional statement if (typeof func !== "function" || !func()).
* Refer to the contract1 function to implement other permission control.
