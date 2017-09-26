# Crypto Design Doc

Similar to Bitcoin and Ethereum, Nebulas also adopts elliptic curve algorithm as its basic encryption algorithm for Nebulas transaction. A user’s private key stored in keystore, which is designed like java keystore. The private key gets needed a passphrase, which encrypted with pbkdf2 algorithm.

## Hash

Contains generic data hash algorithm, like sha256,sha3256 and ripemd160 etc.

## Keystore

The Keystore system is designed like java and android keystore, which lets you store cryptographic keys in a container to make it more difficult to extract from the device. Once keys are in the keystore, they can be used for cryptographic operations with the key material remaining non-exportable. Moreover, it offers facilities to restrict when and how keys can be used.

Keystore support multiple providers to holds the keys,like memory provider, which keep encrypted keys in memory , or persistence provider, which serialize the encrypted key to the file.TPM, TEE and hardware low level security protection will be supported as a provider later.

### key

Keystore manages different types of keys. Each type of key implements the Key interface.For asymmetric encryption, privateKey and publicKey basic key interface are provided:

* `PrivateKey`: This type of key holds a cryptographic PrivateKey, which is optionally stored in a protected format to prevent unauthorized access. 

* `PublicKey`:This type of key contains a single public key belonging to another party. The keystore owner trusts that the public key indeed belongs to the identity identified by the subject (owner) of the private key.

### provider

Teh keystore has different providers to save keys. Currently we provide two ways to save keys, memory_provider and persistence_provider.Before saving, key has been encrypted in keystore. 

* `memory provider`:This type of provider keep keys in memory.After the key has been encrypted with the passphrase when user setkey or load, it cached in memory provider.

* `persistence provider`:This type of provider serialize the encrypted key to the file.The file is compatible with the ethereum's keystore file，users can back up the address with it's privatekey in it.

### signature

The Signature interface is used to provide applications the functionality of a digital signature algorithm. A Signature object can be used to generate and verify digital signatures.

There are two phases to the use of a Signature object for either signing data or verifying a signature:

* Initialization:with either a public key, which initializes the signature for verification (see initVerify), or a private key, which initializes the signature for signing (see initSign).

* Signing or Verifying a signature on all input bytes.

