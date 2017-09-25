# Go-Nebulas Design Doc

Singularity, Version: 0.1.0

## 1 Design Overview

![](resources/overview.png)

> TODO: More features described in our [whitepaper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf), such as NVM, Smart Contracts, PoD, DIP and so on, will be integrated into the framework in later versions very soon.

### 1.1 Core Dataflow

Here is a core workflow example to explain how Nebulas works at version 0.1.0.
For each Nebulas node, it keeps receiving blocks or transactions from network and mining new block locally.

#### 1.1.1 A new block from network, steps 1~4

once a new block received, the node will search the new block’s parent block in current blockchain. If not found, the new block will be cached in blocks pool. Otherwise, the node will try to verify the new block. If the new block is valid, it will be linked to current blockchain.

#### 1.1.2 A new block from local miners, steps 5~7

once a new block minted, the node will broadcast the new block to the Internet.

![](resources/workflow.png)

<!-- 
@startuml workflow

-> Net: 1. received new block
Net -> Blockchain: 2. search its parent
Blockchain -> Blockchain: 3. verify new block
Blockchain -> Blockchain: 4.link new block
PoW -> PoW: 5. mining
PoW -> Net: 6. minted
Net -> : 7. broadcast new block

@enduml
-->

## 2 blockchain spec

### 2.1 Blockchain

``` txt
Block Structure
+---------------+----------------+
|  blockHeader  |  transactions  |
+---------------+----------------+

Block Header Structure
+--------+--------------+-------------+---------+------------+-------------+
|  hash  |  parentHash  |  stateRoot  |  nonce  |  coinbase  |  timestamp  |
+--------+--------------+-------------+---------+------------+-------------+

Transaction Structure
+--------+--------+------+---------+---------+-------------+--------+-----------+
|  hash  |  from  |  to  |  value  |  nonce  |  timestamp  |  sign  |  payload  |
+--------+--------+------+---------+---------+-------------+--------+-----------+
```

### 2.2 Block Pool

We use block pool to cache all valid blocks whose parents cannot be found in current blockchain yet. As for a new block received, we follow the below steps to process it.

![](resources/blockpool.png)

<!-- 
@startuml addBlockInPool

start

if (the new block exists in blockchain?) then (yes)
    stop
else (no)
    if (the block's nonce is right?) then (yes)
        if (the block's hash is right?) then (yes)
            while (more transactions in the block?) is (yes)
                if (the transaction's hash is right?) then (yes)
                    if (the transaction's sign is right?) then (yes)
                    else (no)
                        stop
                    endif
                else (no)
                    stop
                endif
            endwhile (no)
            if (the block's parent exists in blockchain?) then (yes)
                :clone the block's parent's world states;
                :execute all transactions in the block;
                while (more states in the block?) is (yes)
                    if (the state root matches?) then(yes)
                    else (no)
                        stop
                    endif
                endwhile (no)
                :append the block to blockchain;
            else (no)
                :cache the block in block pool;
            endif
        else (no)
            stop
        endif
    else (no)
        stop
    endif
endif

stop

@enduml 
-->

### 2.3 State (World State)

World States are maintained as merkle trees.
It’s divided into the following parts:

``` txt
stateRoot: key, address; value, balance

txRoot(TBD)

receiptRoot(TBD)

voteRoot(TBD)

rankRoot(TBD)
```

We verify these states using merkle patricia tree, more details in [merkle pacitria trie](https://github.com/nebulasio/wiki/blob/master/merkle_trie.md)

### 2.4 Serialization

At version 0.1.0, we use json serializer to make things work at first.

> TODO: use Protobuf serializer instead

### 2.5 Sync（TBD）

Here is a entire blockchain synchronization policy to be implemented.
We starts from the snapshot of the blockchain in local storage. All blocks generted after the snapshot will be synchronized from peers and replayed locally.

``` pseudo
push all tails of current blockchain into a queue, as Q
loop if Q is not empty
  set t as Q.pop()
  if t is on canonical chain on peers' blockchains
    if t is the tail on peers' blockchains
      ask local miners to start mining
    else
      Q.push(t.children)
  else
    Q.push(t.parent)
```

## 3. consensus spec

PoW(Proof-of-Work) Consensus Algorithm, similar as Bitcoin.

> TODO: use PoD(Proof-of-Devotion, introduced in our [whitepaper](https://nebulas.io/docs/NebulasTechnicalWhitepaper.pdf)) instead.

### 3.1 State Machine

We consider all consensus algorithms as state machines.
States transitions are triggered by internal events or network message event as following.

![](resources/pow.png)

<!-- 
@startuml overview
[*] \-\-> PoW
state PoW {
    [*] \-\-> Prepare
    Prepare: prepare a new block for mining
    Prepare \-\-> Mining : start mining
    Prepare \-\-> Minted : new block received
    Mining: search valid nonce for the new block
    Mining: broadcast the new block
    Mining \-\-> Minted : found the nonce/block
    Mining \-\-> Minted : new block received
    Minted:
    Minted: link the new block to blockchain
    Minted: fork choice
    Minted \-\-> Prepare : start over
}
PoW \-\-> [*] : stop
@enduml
 -->

### 3.2 Fork Choice

Always choose the longest chain as the canonical chain

## 4 crypto spec

### 4.1 Hash

### 4.2 Keystore

Keystore represents a storage facility for cryptographic keys, similar to java keystore.
Each key in a keystore is identified by an "alias" string. In the case of private keys, these strings distinguish among the different ways in which the key may authenticate itself.  keys storage can be unlock with passphrase and a duration. After the duration is timeout, users can only getkey with passphrase or unlock it again.
key
Keystore manages different types of keys. Each type of key implements the Key interface.For asymmetric encryption, privateKey and publicKey basic key interface are provided:

PrivateKey: This type of key holds a cryptographic PrivateKey, which is optionally stored in a protected format to prevent unauthorized access. 

PublicKey:This type of key contains a single public key PublicKey belonging to another party. The keystore owner trusts that the public key indeed belongs to the identity identified by the subject (owner) of the certificate.This type of key can be used to authenticate other parties.

provider

Teh keystore has different providers to save keys. Currently we provide two ways to save keys, memory_provider and persistence_provider.Before saving, key has been encrypted in keystore. TPM and hardware low level security protection will be supported as a provider later.

memory provider:This type of provider keep keys in memory.After the key has been encrypted with the passphrase when user setkey or load, it cached in memory provider.

persistence provider:This type of provider serialize the encrypted key to the file.The file is compatible with the ethereum's keystore file，users can backup the address with it's privatekey in it.

signature

The Signature interface is used to provide applications the functionality of a digital signature algorithm. Digital signatures are used for authentication and integrity assurance of digital data. A Signature object can be used to generate and verify digital signatures.

There are two phases to the use of a Signature object for either signing data or verifying a signature:

Initialization:with either a public key, which initializes the signature for verification (see initVerify), or a private key, which initializes the signature for signing (see initSign).

Signing or Verifying a signature on all input bytes.


Network Protocol(@Leon)


We build our peer-to-peer network connection based on libp2p and realize the basic  peer-to-peer connection，routing table update , node discovery and Simple implementation of message broadcast.

We always use Big-endian for our network transport protocol。
In go-nebulas version 0.1.0, we just build a simplest implementation of p2p network.
We defined three kinds of protocols  for different scenarios
Ping : nebulas/ping/1.0.0
Lookup: nebulas/lookup/1.0.0
Block: nebulas/block/1.0.0

For each of these different protocols, we will register the corresponding handler.
Perhaps we just need one protocol in the future, we can put the content of protocol to the packet data. so we only need to parse the message body to know the specific protocol.

In go-nebulas version 0.2.0, we will define specific protocol formats.

    01234567890123456789012345678901
   +------------------------------+
   |ptl_id|                       |
   +------------------------------+
   |            data_size         |
   +------------------------------+
   |             data             |
   +------------------------------+

```
[protocol_id][data_size][data]
```
The protocol_id take 1 byte and the data_size take 4 bytes.
E.g. 
Ping:
[1][0 0 0 16][...]
Pong:
[2][0 0 0 16][...]
GetRoutes:
[3][0 0 0 250][...]
NewBlock:
[4][0 0 1 18][...]
GetBlocks:
[5][0 0 0 40][...]
Blocks:
[6][0 0 1 80][...]

Our network simply provides the simplest peer-to-peer data propagation without the business.we threw our specific business to the top dispatcher.we use p2p_manager to manage our p2p message and message broadcast.

## 5 Smart Contract (TBD)

## 6 NF (TBD)

### 6.1 NVM (TBD)

## 7 NR (TBD)

## 8 DIP (TBD)

