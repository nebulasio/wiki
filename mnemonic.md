## Abstract
NAS's(Nebulas) mnemonic official path is [```m/44'/2718'/0'/0/0```](https://github.com/satoshilabs/slips/blob/master/slip-0044.md).

## Introduction
Accoding to the BIP-39, the mnemonic code is another form of private key. So we also support the mnemonic code, and we follow the BIP-44 protocal (Multi-Account Hierarchy for Deterministic Wallets). Therefore, mnemonic code can be generate not only one private key(block chain address), to a certain extent, you just need hold one mnemonic words to back up all your wallets.

The Nebulas's mnemonic official path is [```m/44'/2718'/0'/0/0```](https://github.com/satoshilabs/slips/blob/master/slip-0044.md). Accoding to the BIP-44, the different block chain's mnemonic path should have the different ```"Coin type"``` (Path level is:```m / purpose' / coin_type' / account' / change / address_index```).

We choose the ```2718``` as Nebulas's ```Coin type```, which is from the [Euler's number](https://en.wikipedia.org/wiki/E_(mathematical_constant)). The number ```e``` is a mathematical constant, approximately equal to <b>2.71828</b>, which appears in many different settings throughout mathematics. So we chosse 2718 hope that Nebulas can be go on forever.


## References
Nebulas's mnemonic role is follow the BIP's common convention, here is the references:  
[BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)  
[BIP-39-wordlists](https://github.com/bitcoin/bips/blob/master/bip-0039/bip-0039-wordlists.md)  
[BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)  
[SLIP-0044](https://github.com/satoshilabs/slips/blob/master/slip-0044.md)
