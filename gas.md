# Transaction Gas

In Nebulas, either a normal transaction which transfer balance or a smart contract deploy & call burns gas, and charged from the balance of `from` address.  A transaction contains two gas parameters `gasPrice` and `gasLimit` :

* `gasPrice`: the price of per gas.
* `gasLimit`: the limit of gas use. 

The actual gas consumption of a transaction is the value: `gasPrice` * `gasUsed`, which will be the reward to the miner coinbase. The `gasUsed` value must less than or equal to the `gasLimit`.

## Gas constitution 
When users submit a transaction, gas will be burned at three places: `transaction submition`, `transaction data storage` and `contract deploy/call`. In all three places, the power and resources of the net will be consumed and the miners will need to be paid. 

### Transaction submition
A transaction's submition will add a transaction to the tail block. Miners use resources to record the deal and need to be paid. It will burn a fixed number of gas, that would be defined in code as the following:

```
// TransactionGas default gas for normal transaction
TransactionGas
```
If the transaction verifies failed, the gas and value transfer will rollback.

### Transaction data storage
When deploying a contract or call contract's method, the raw data of contract execution save in the transaction's data filed, which cost the storage of resources on the chain. A formula to calculate gas:

```
len(data) * TransactionDataGas
```
The `TransactionDataGas` is a fixed number of gas defined in code.

### Smart contract deploy & call
When a smart contract deploys or call in transaction submition, the contract execution will consume miner's computer resources and may store data on the chain. 

* **execution instructions**: Every contract execution cost the miner's computer resources, the v8 instruction counter calculates the execution instructions. The limit of execution instructions will prevent the excessive consumption of computer computing power and the generation of the death cycle.

* **contract storage**: The smart contract's `LocalContractStorage` which storage contract objects also burn gas. Only one gas per 32 bytes is consumed when stored(`set`/`put`), `get ` or `delete` not burns gas.

The limit of **contract execution** is:
    
    ```
    gasLimit - TransactionGas - len(data) * TransactionDataGas
    ```
    
## Gas metrics

TODO

## Tips 

In nebulas, the transaction pool of each node has a minimum `gasPrice` and maximum `gasLimit` value. If transaction's `gasPrice` below the pool's `gasPrice` or the `gasLimit` greater than the pool's gasLimit the transaction will be refused.


