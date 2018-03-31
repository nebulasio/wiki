# Event functionality

The ```Event``` functionality is used to make users or developers subscribe interested events. These events are generated during the execution of the blockchain, and they record the key execution steps and execution results of the chain. To query and verify the execution results of transactions and smart contracts, we record these two types of events into a trie and save them to the chain.

Event structure:

```go
type Event struct {
	Topic string // event topic, subscribe keyword
	Data  string // event content, a json string
}
```

After a event is generated, it will be collected for processing in [eventEmitter](https://github.com/nebulasio/go-nebulas/blob/master/core/event.go). Users can use the emitter subscription event. If the event is not subscribed, it will be discarded, and for the event that has been subscribed, the new event will be discarded because of the non-blocking mechanism, if the channel is not blocked in time.

## Events list:

* [TopicNewTailBlock](#topicnewtailblock)
* [TopicRevertBlock](#topicrevertblock)
* [TopicLibBlock](#topiclibblock)
* [TopicPendingTransaction](#topicpendingtransaction)
* [TopicTransactionExecutionResult](#topictransactionexecutionresult)
* [EventNameSpaceContract](#eventnamespacecontract)

## Event Reference

#### TopicNewTailBlock
This event occurs when the tail block of the chain is updated.

- Topic:`chain.newTailBlock`
- Data:
	-  `height`: block height
	-  `hash`: block hash
	-  `parent_hash`: block parent hash
	-  `acc_root`: account state root hash
	-  `timestamp`: block timestamp
	-  `tx`: transaction state root hash
	-  `miner`: block miner

#### TopicRevertBlock
This event occurs when a block is revert on the chain.

- Topic:`chain.revertBlock`
- Data: The content of this topic is like [TopicNewTailBlock](#topicnewtailblock) data.

#### TopicLibBlock
This event occurs when the latest irreversible block change.

- Topic:`chain.latestIrreversibleBlock`
- Data: The content of this topic is like [TopicNewTailBlock](#topicnewtailblock) data.

#### TopicPendingTransaction
This event occurs when a transaction is pushed into the transaction pool.

- Topic:`chain.pendingTransaction`
- Data:
	- `chainID`: transaction chain id
	- `hash`: transaction hash
	- `from`: transaction from address string
	- `to`: transaction to address string
	- `nonce`: transaction nonce
	- `value`: transaction value
	- `timestamp`: transaction timestamp
	- `gasprice`: transaction gas price
	- `gaslimit`: transaction gas limit
	- `type`: trsnaction type

#### TopicTransactionExecutionResult
This event occurs when the end of a transaction is executed. This event will be recorded on the chain, and users can query with RPC interface [GetEventsByHash](https://github.com/nebulasio/wiki/blob/master/rpc.md#geteventsbyhash).

This event records the execution results of the transaction and is very important.

- Topic:`chain.transactionResult`
- Data:
	- `hash`: transaction hash
	- `status`: transaction status, 0 failed, 1success, 2 pending
	- `gasUsed`: transaction gas used
	- `error`: transaction execution error. If the transaction is executed successfully, the field is empty.

#### EventNameSpaceContract
This event occurs when the contract is executed. When the contract is executed, the contract can record several events in the execution process. If the contract is successful, these events will be recorded on the chain and can be subscribed, and the event of the contract will not be recorded at the time of the failure. This event will also be recorded on the chain, and users can query with RPC interface [GetEventsByHash](https://github.com/nebulasio/wiki/blob/master/rpc.md#geteventsbyhash).

- Topic:`chain.contract.[topic]` The topic of the contract event has a prefix `chain.contract.`, the content is defined by the contract writer.
- Data: The content of contract event is defined by contract writer.


## Subscribe
All events can be subscribed and the cloud chain provides a subscription RPC interface [Subscribe](https://github.com/nebulasio/wiki/blob/master/rpc.md#subscribe). It should be noted that the event subscription is a non-blocking mechanism. New events will be discarded when the RPC interface is not handled in time.

## Query
Only events recorded on the chain can be queried using the RPC interface [GetEventsByHash](https://github.com/nebulasio/wiki/blob/master/rpc.md#geteventsbyhash). Current events that can be queried include:

* [TopicTransactionExecutionResult](#topictransactionexecutionresult)
* [EventNameSpaceContract](#eventnamespacecontract)






