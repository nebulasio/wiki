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

## Gas Count Matrix

| Operator      | Gas Count/Opt. | Description  |
| ------------- | -------------: | :-----|
| Binary | 1  | Binary & logical operator|
| Load   | 2  | Load from memory |
| Store  | 2  | Save to memory |
| Return | 2  | Return value, save to memory |
| Call (inner) | 4 | Call functions in the same Smart Contract |
| Call (external) | 100 | Call functions from other Smart Contract |


| Expression | Sample Code | Binary Opt. | Load Opt. | Store Opt. | Return Opt. | Call (inner) Opt. | Gas Count |
| ---------- | :---------- | ----------: | --------: | ---------: | ----------: | ----------------: | --------: |
| CallExpression | a(x, y) | 0 | 0 | 1 | 1 | 1 | 8 |
| AssignmentExpression | x&=y | 1 | 0 | 1 | 0 | 0 | 3 |
| BinaryExpression | x==y | 1 | 0 | 0 | 1 | 0 | 3 |
| UpdateExpression | x++ | 1 | 0 | 1 | 0 | 0 | 3 |
| UnaryExpression  | x+y | 1 | 0 | 0 | 1 | 0 | 3 |
| LogicalExpression | x||y | 1 | 0 | 0 | 1 | 0 | 3 |
| MemberExpression | x.y | 0 | 1 | 0 | 1 | 0 | 4 |
| NewExpression | new X() | 0 | 0 | 1 | 1 | 1 | 8 |
| ThrowStatement | throw x | 0 | 0 | 0 | 1 | 1 | 6 |
| MetaProperty | new.target | 0 | 1 | 0 | 1 | 0 | 4 |
| ConditionalExpression | x?y:z | 1 | 0 | 0 | 1 | 0 | 3 |
| YieldExpression | yield x | 0 | 0 | 0 | 1 | 1 | 6 |
| Event | | 0 | 0 | 0 | 0 | 0 | 20 |
| Storage | | 0 | 0 | 0 | 0 | 0 | 1 gas/bit |

## Tips

In nebulas, the transaction pool of each node has a minimum `gasPrice` and maximum `gasLimit` value. If transaction's `gasPrice` below the pool's `gasPrice` or the `gasLimit` greater than the pool's gasLimit the transaction will be refused.


