# Transaction Gas

In Nebulas, either a normal transaction which transfer balance or a smart contract deploy & call burns gas, and charged from the balance of `from` address.  A transaction contains two gas parameters `gasPrice` and `gasLimit` :

* `gasPrice`: the price of per gas.
* `gasLimit`: the limit of gas use.

The actual gas consumption of a transaction is the value: `gasPrice` * `gasUsed`, which will be the reward to the miner coinbase. The `gasUsed` value must less than or equal to the `gasLimit`. Transaction's `gasUsed` can be estimate by RPC interface [estimategas](https://github.com/nebulasio/wiki/blob/master/rpc.md#estimategas) and store in transaction's execution result event.


## Design reason
Users want to avoid gas costs when the transaction is packaged. Like Bitcoin and Ethereum, Nebulas GAS is used for transaction fee, it have two major purposes:

- As a rewards for minter, to incentive them to pack transactions. The packaging of the transaction costs the computing resources, especially the execution of the contract, so the user needs to pay for the transaction.
- As a cost for attackers. The DDOS attach is quite cheap in Internet, black hackers hijack user's computer to send large network volume to target server. In Bitcoin and Ethereum network, each transaction must be paid, that significant raise the cost of attack.


## Gas constitution
When users submit a transaction, gas will be burned at these aspects:

- `transaction submition`
- `transaction data storage`
- `transaction payload addition` 
- `transaction payload execution`(smart contract execution) 

In all these aspects, the power and resources of the net will be consumed and the miners will need to be paid.

### Transaction submition
A transaction's submition will add a transaction to the tail block. Miners use resources to record the deal and need to be paid. It will burn a fixed number of gas, that would be defined in code as the following:

```
// TransactionGas default gas for normal transaction
TransactionGas = 20000
```
If the transaction verifies failed, the gas and value transfer will rollback.

### Transaction data storage
When deploying a contract or call contract's method, the raw data of contract execution save in the transaction's data filed, which cost the storage of resources on the chain. A formula to calculate gas:

```
TransactionDataGas = 1

len(data) * TransactionDataGas
```
The `TransactionDataGas` is a fixed number of gas defined in code.

### 
Different types of transactions' payload have different gas consumption when executed. The types of transactions currently supported by nebulas are as follows:

- `binary`: The `binary` type of transaction allows users to attach binary data to transaction execution. These binary data do not do any processing when the transaction is executed.
	-  The fixed number of gas defined **0**. 
- `deploy & call`: The `deploy` and `call` type of transaction allows users to deploy smart contract on nebulas. Nebulas must start `nvm` to execute the contract, so these types of transction must paid for the nvm start.
	- The fixed number of gas defined **60**. 

### Transaction payload execution(Smart contract deploy & call)

The `binary` type of transaction do not do any processing when the transaction is executed, so the execution need not be paid.

When a smart contract deploys or call in transaction submition, the contract execution will consume miner's computer resources and may store data on the chain.

* **execution instructions**: Every contract execution cost the miner's computer resources, the v8 instruction counter calculates the execution instructions. The limit of execution instructions will prevent the excessive consumption of computer computing power and the generation of the death cycle.

* **contract storage**: The smart contract's `LocalContractStorage` which storage contract objects also burn gas. Only one gas per 32 bytes is consumed when stored(`set`/`put`), `get ` or `delete` not burns gas.

The limit of **contract execution** is:

    ```
    gasLimit - TransactionGas - len(data) * TransactionDataGas - TransactionPayloadGasCount[type]
    ```

## Gas Count Matrix
The gas count matrix of smart contract execution 

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

In nebulas, the transaction pool of each node has a minimum and maximum `gasPrice` and maximum `gasLimit` value. If transaction's `gasPrice` is not in the range of the pool's `gasPrice` or the `gasLimit` greater than the pool's gasLimit the transaction will be refused.

Transaction pool gasPrice and gasLimit configuration:

- `gasPrice`
	- minimum: The minimum gasPrice can be set in the configuration file. If the minimum value is not configured, the default value is `20000000000`(2*10^10).
	- maximum: The maximum gasPrice is `1000000000000`(10^12), transaction pool's maximum configuration and transaction's `gasPrice` can't be overflow.

- `gasLimit`	
	- minimum: The transaction's minimum gasLimit must greater than zero.
	- maximum: The maximum gasPrice is `50000000000`(50*10^9), transaction pool's maximum configuration and transaction's `gasLimit` can't be overflow.


