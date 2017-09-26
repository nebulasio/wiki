# crypto spec

## Hash

## Keystore

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
