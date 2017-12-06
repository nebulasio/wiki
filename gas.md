# Transaction Gas

In Nebulas, either a normal transaction which transfer balance or a smart contract deploy & call burns gas, and charged from the balance of `from` address.  A transaction contains two gas parameters `gasPrice` and `gasLimit` :

* `gasPrice`: the price of per gas.
* `gasLimit`: the limit of gas use. 

The actual gas consumption of a transaction is the value: `gasPrice` * `gasUsed`, which will be the reward to the miner coinbase. The `gasUsed` value must less than or equal to the `gasLimit`.

## Normal transaction
When users submit a normal transaction, which only transfer balance from one address to the others, will burn a fixed number of gas, that would be defined in code as the following:
```
// TransactionGas default gas for normal transaction
TransactionGas
```
If the transaction executes or verifies failed, the gas and value transfer will rollback.

## Smart contract deploy & call
When a smart contract deploys or call in transaction submit, the gas is consumed in three points:

* **submit transaction**: Every transaction submit will cost the stationary gas value `TransactionGas `, as the transaction submission drain the miners' resources.
* **transaction's data**: When deploying a contract or call contract's method, the raw data of contract execution save in the transaction's data filed, which cost the storage of resources on the chain. A formula to calculate gas:
```
len(data) * TransactionDataGas
```
* **contract execution**:`ExecutionInstructions` and `TotalMemorySize`will burn gas when contract executing:
    * `ExecutionInstructions`: The v8 instruction counter calculates the execution instructions.
    * `TotalMemorySize`: The smart contract's `LocalContractStorage` which storage contract objects also burn gas. Only one gas per 32 bytes is consumed when stored(`set`/`put`), `get ` or `delete` not burns gas.
    
     The limit of **contract execution** is:
    
    ```
    gasLimit - TransactionGas - len(data) * TransactionDataGas
    ```

### Transaction pool 

In nebulas, the transaction pool of each node has a minimum `gasPrice` and maximum `gasLimit` value. If transaction's `gasPrice` below the pool's `gasPrice` or the `gasLimit` greater than the pool's gasLimit the transaction will be refused.


