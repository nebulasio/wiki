# Crypto Design Doc

Similar to Bitcoin and Ethereum, Nebulas also adopts elliptic curve algorithm as its basic encryption algorithm for Nebulas transactions. Users' private key will be encrypted with user's passphrases and stored in keystore.

## Hash

Support generic hash functions, like sha256, sha3256 and ripemd160 etc.

## Keystore

Nebulas Keystore are designed to manage user's keys.

### Key

The Key interface is designed to support various keys, including symmetric keys and asymmetric keys.

### Provider

Keystore provide different methods to save keys, such as `memory_provider` and `persistence_provider`. Before saved, key has been encrypted in keystore.

* `memory provider`: This type of provider keep keys in memory. After the key has been encrypted with the passphrase when user setkey or load, it is cached in memory provider.

* `persistence provider`: This type of provider serialize the encrypted key to the file. The file is compatible with the ethereum's keystore file，users can back up the address with its privatekey in it.

### Signature

The Signature interface is used to provide applications the functionality of a digital signature algorithm. A Signature object can be used to generate and verify digital signatures.

There are two phases to use a Signature object for either signing data or verifying a signature:

* Initialization: with either a public key, which initializes the signature for verification (see initVerify), or a private key, which initializes the signature for signing (see initSign).

* Signing or Verifying a signature on all input bytes.

> Similar as [Android Keystore](https://developer.android.com/training/articles/keystore.html), TPM, TEE and hardware low level security protection will be supported as a provider later.
