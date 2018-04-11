# Blockchain

## Model

Nebulas use accounts model instead of UTXO model.
The execution of transactions will consume gas.

## Data Structure

``` txt
Block Structure
+---------------+----------------+--------------+
|  blockHeader  |  transactions  |  dependency  |
+---------------+----------------+--------------+
blockHeader: header info
transactions: transactions array
dependency: the dependency relationship among transactions

Block Header Structure
+-----------+--------+--------------+------------+-------------+-------+--------+
|  chainid  |  hash  |  parentHash  |  coinbase  |  timestamp  |  alg  |  sign  |
+-----------+--------+--------------+------------+-------------+-------+--------+
+-------------+-----------+--------------+-----------------+
|  stateRoot  |  txsRoot  |  eventsRoot  |  consensusRoot  |
+-------------+-----------+--------------+-----------------+
chainid: chain identity the block belongs to
hash: block hash
parentHash: parent block hash
coinbase: account to receive the mint reward
timestamp: the number of nanoseconds elapsed since January 1, 1970 UTC
alg: the type of signature algorithm
sign: the signature of block hash
stateRoot: account state root hash
txsRoot: transactions state root hash
eventsRoot: events state root hash
consensusRoot: consensus state, including proposer and the dynasty of validators


Transaction Structure
+-----------+--------+--------+------+---------+---------+-------------+
|  chainid  |  hash  |  from  |  to  |  value  |  nonce  |  timestamp  |
+-----------+--------+--------+------+---------+---------+-------------+
+--------+------------+------------+
|  data  |  gasPrice  |  gasLimit  |
+--------+------------+------------+
chainid: chain identity the block belongs to
hash: transaction hash
from: sender's wallet address
to: receiver's wallet address
value: transfer value
nonce: transaction nonce
timestamp: the number of seconds elapsed since January 1, 1970 UTC
alg: the type of signature algorithm
sign: the signature of block hash
data: transaction data, including the type of transaction(binary transfer/deploy smart contracts/call smart contracts) and payload
gasPrice: the price of each gas consumed by the transaction
gasLimit: the max gas that can be consumed by the transaction
```

## Blockchain Update

In our opinion, **Blockchain** only needs to care about how to process new blocks to grow up safely and efficiently. What's more, **Blockchain** can only get new blocks in the following two channels.

### A new block from network

Because of the unstable network latency, we cannot make sure any new block received can be linked to our current **Chain** directly. Thus, we need the **Blocks Pool** to cache new blocks.

### A new block from local miner

At first, we need the **Transactions Pool** to cache transactions from network. Then, we wait for a new block created by local **Consensus** component, such as DPoS.

No matter where a new block comes from, we use the same steps to process it as following.

![](resources/blockpool.png)

## World State

Every block contains the current world state, consist of following four states. They are all maintained as [Merkle Trees](./merkle_trie.md).

### Accounts State

All accounts in current block are stored in Accounts State.
Accounts are divided into two kinds, normal account & smart contract account.

Normal Account, including

- **wallet address**
- **balance**
- **nonce**: account's nonce, it will increment in steps of 1

Smart Contract Account， including

- **contract address**
- **balance**
- **birth place**: the transaction hash where the contract is deployed
- **variables**: contains all variables' values in the contract

### Transactions State

All transactions submitted on chain are storage in Transactions State.

### Events State

While transactions are executed, many events will be triggered.
All events triggered by transactions on chain are stored in Events State.

### Consensus State

The context of consensus algorithm is stored in consensus state.

As for DPoS, the consensus state includes

- **timestamp**: current slot of timestamp
- **proposer**: current proposer
- **dynasty**: current dynasty of validators

### Serialization

We choose Protocol Buffers to do general serialization in consideration of the following benefits:

- Large scale proven.
- Efficiency. It omits key literals and use varints encoding.
- Multi types and multilangue client support. Easy to use API.
- Schema is good format for communication.
- Schema is good for versioning/extension, i.e., adding new message fields or deprecating unused ones.

Specially, we use json to do serialization in smart contract codes instead of protobuf for the sake of readability.

## Synchronization

Sometimes we will receive a block with height much higher than its current tail block. When the gap appears, we need to sync blocks from peer nodes to catch upd them.

Nebulas provides two method to sync blocks from peers: Chunks Downloader and Block Downloader. If the gap is bigger than 32 blocks, we'll choose Chunk Downloader to download a lot of blocks in chunks. Otherwise, we choose Block Downloader to download block one by one.

### Chunks Downloader

Chunk is a collection of 32 successive blocks. Chunks Downloader allows us to download at most 10 chunks after our current tail block each time. This chunk-based mechanism could help us minimize the number of network packets and achieve better safety.

The procedure is as following,

```txt
1. A sends its tail block to N remote peers.
2. The remote peers locate the chunk C that contains A's tail block.
   Then they will send back the headers of 10 chunks, including the chunk C and 9 C's subsequent chunks, and the hash H of the 10 headers.
3. If A receives >N/2 same hash H, A will try to sync the chunks represented by H.
4. If A has fetched all chunks represented by H and linked them on chain successfully, Jump to 1.
```

In steps 1~3, we use majority decision to confirm the chunks on canonical chain. Then we download the blocks in the chunks in step 4.

**Note:** `ChunkHeader` contains an array of 32 block hash and the hash of the array. `ChunkHeaders` contains an array of 10 `ChunkHeaders` and the hash of the array.

Here is a diagram of this sync procedure:

![](resources/the-diagram-of-sync-process.png)

### Block Downloader

When the length gap between our local chain with the canonical chain is smaller than 32, we'll use Block downloader to download the missing blocks one by one.

The procedure is as following,

```txt
1. C relays the newest block B to A and A finds B's height is bigger than current tail block's.
2. A sends the hash of block B back to C to download B's parent block.
3. If A received B's parent block B', A will try to link B' with A's current tail block.
   If failed again, A will come back to step 2 and continue to download the parent block of B'. Otherwise, finished.
```

This procedure will repeat until A catch up the canonical chain.

Here is a diagram of this download procedure:

![](resources/the-diagram-of-download-process.png)