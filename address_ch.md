# Nebulas 地址设计

Nebulas 地址系统是精心设计的。正如你将在下面看到的，账户和智能合约地址都是 “n” 开头的字符串，可以认为它们就是我们信仰的 Nebulas(NAS)。

## 账户地址

与比特币和以太坊类似，Nebulas 也采用了椭圆曲线算法作为 Nebulas 账户的加密算法。Nebulas 账户的地址是通过**公钥**计算出来的，而公钥则通过用户**密码短语**进行加密的**私钥**计算而来。此外，我们也有校验设计，旨在防止用户因为输入几个错误字符而将 _Nas_ 意外发送给错误的用户账户。

具体的计算公式如下：
```
1.  content = ripemd160(sha3_256(public key))
    length: 20 bytes
                         +--------+--------+------------------+
2.  checksum = sha3_256( |  0x19  +  0x57  |      content     | )[:4]
                         +--------+--------+------------------+
    length: 4 bytes

                        +--------+---------+-----------------+------------+
3.  address = base58( |    0x19  |  0x57   |     content     |  checksum  | ï¼‰
                        +--------+---------+-----------------+------------+
    length: 35 chars
```

**0x57** 是一个单字节“类型码”，表示账户地址，**0x19** 则是一个固定的单字节“填充”。

在这个阶段，Nebulas 只采用了普通的比特币 [base58](https://en.wikipedia.org/wiki/Base58) 编码模式。一个有效的地址示例：_n1TV3sU6jyzR4rJ1D7jCAmtVGSntJagXZHC_

## 智能合约地址

计算合约地址和计算账户地址略有不同，合约发送方的密码短语不是必需的，地址和 nonce 才是必需。有关更多信息，请查看 [smart contract](https://github.com/nebulasio/wiki/blob/master/tutorials/%5BEnglish%5D%20Nebulas%20101%20-%2003%20Smart%20Contracts%20JavaScript.md) 和 [rpc.sendTransaction](https://github.com/nebulasio/wiki/blob/master/rpc.md#sendtransaction)。计算公式如下：

```
1.  content = ripemd160(sha3_256(tx.from, tx.nonce))
    length: 20 bytes
                         +--------+--------+------------------+
2.  checksum = sha3_256( |  0x19  |  0x58  +      content     | )[:4]
                         +--------+--------+------------------+
    length: 4 bytes

                      +--------+---------+-----------------+------------+
3.  address = base58( |  0x19  |  0x58   |     content     |  checksum  | ï¼‰
                      +--------+---------+-----------------+------------+
    length: 35 chars
```

**0x58** 是一个单字节“类型码”，表示账户地址，**0x19** 则是一个固定的单字节“填充”。

一个有效的地址示例：  _n1sLnoc7j57YfzAVP8tJ3yK5a2i56QrTDdK_