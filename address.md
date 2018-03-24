# Nebulas Address Design

Nebulas address sys is carefully designed. As u will see below, both account and smart contract address are strings starting with an "n" in JSON, which could be thought of as our faith &#x1f618;Nebulas/NAS&#x1f618;

## Account Address

Similar to Bitcoin and Ethereum, Nebulas also adopts elliptic curve algorithm as its basic encryption algorithm for Nebulas accounts. The address is derived from **public key**, which is in turn derived from the **private key** that encrypted with user's **passphrase**.Also we have the checksum design aiming to prevent a user from sending _Nas_ to a wrong user account accidentally due to entry of several incorrect characters.

The specific calculation formula is as follows:
```
1.  content = ripemd160(sha3_256(public key))
    length: 20 bytes
                         +--------+--------+------------------+
2.  checksum = sha3_256( |  0x19  +  0x57  |      content     |   )[:4]
                         +--------+--------+------------------+
    length: 4 bytes

                        +--------+---------+-----------------+------------+
3.  address = base58(   |  0x19  |  0x57   |     content     |  checksum  | ）
                        +--------+---------+-----------------+------------+
    length: 35 chars
```

 **0x57** is a one-byte "type code" for account address, **0x19** is a one-byte fixed "padding"

At this stage, Nebulas just adopts the normal bitcoin [base58](https://en.wikipedia.org/wiki/Base58) encoding schema. A valid address is like:  _n1TV3sU6jyzR4rJ1D7jCAmtVGSntJagXZHC_

## Smart Contract Address

Calculating contract address differs slightly from account, passphrase of contract sender is not required but address & nonce. For more information, plz check [smart contract](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md) and [rpc.sendTransaction](https://github.com/nebulasio/wiki/blob/master/rpc.md#sendtransaction). Calculation formula is as follows:

```
1.  content = ripemd160(sha3_256(tx.from, tx.nonce))
    length: 20 bytes
                            +--------+--------+------------------+
2.  checksum = sha3_256(    |  0x19  |  0x58  +      content     |   )[:4]
                            +--------+--------+------------------+
    length: 4 bytes

                        +--------+---------+-----------------+------------+
3.  address = base58(   |  0x19  |  0x58   |     content     |  checksum  | ）
                        +--------+---------+-----------------+------------+
    length: 35 chars
```

 **0x58** is a one-byte "type code" for smart contract address, **0x19** is a one-byte fixed "padding"

A valid address is like:  _n1sLnoc7j57YfzAVP8tJ3yK5a2i56QrTDdK_