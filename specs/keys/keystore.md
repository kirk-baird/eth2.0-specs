# Keystores

A keystore is a JSON file which stores a password encrypted version of a user's private key. It is designed to be an easy-to-implement format for storing and exchanging keys. Furthermore, this specification is designed to utilize as few crypto-constructions and make a minimal number of security assumptions.

![Keystore Diagram](./keystore.png)

## Definition

Private key is obtained by taking the bitwise XOR of the `ciphertext` and the `derived_key`. The `derived_key` is obtained by running scrypt with the user-provided password and the `scryptparams` obtained from within the keystore file as parameters. If a keystore file is being generated for the first time, the `salt` KDF parameter must be obtained from a CSPRNG. The `ciphertext` is simply read from the keystore file. The length of the `ciphertext` and the output key length of scrypt.

```python
def decrypt_keystore(password: str, dklen: int, n: int, p: int, r: int, salt: bytes, ciphertext) -> bytes:
    assert len(ciphertext) == dklen
    derived_key = scrypt(password, dklen, n, p, r, salt)
    return bytes(a ^ b for a, b in zip(derived_key, ciphertext))
```

## MAC

The `mac` acts as a method for verifying that password provided by the user is indeed correct. This is done by means of an equality check between the SHA256 hash of the `derived_key` and the `mac`.

```python
def verify_password(password: str, dklen: int, n: int, p: int, r: int, salt: bytes, mac: bytes) -> bool:
    derived_key = scrypt(password, dklen, n, p, r, salt)
    return sha256(derived_key) == mac
```

## UUIDs

The `id` provided in the keystore is a randomly generated UUID and is intended to be used as a 128-bit proxy for referring to a particular set of keys or account. This level of abstraction provides a means of preserving privacy for a secret-key or for referring to keys when they are not decrypted.

## Test vectors

**The following is not a valid test. It is a placeholder that will be populated with valid data soon** (tm):

Test values:

* Password: `testpassword`
* Secret: `7a28b5ba57c53603b0b07b56bba752f7784bf506fa95edc395f5cf6c7514fe9d`

```json
This is not a valid test, it is a placeholder.
{
    "crypto" : {
        "ciphertext" : "d172bf743a674da9cdad04534d56926ef8358534d458fffccd4e6ad2fbde479c",
        "mac": "e39cee865733698f50a84c33f13222da5857b8af9721f401ae011fa549c0b7f1",
        "scryptparams" : {
            "dklen" : 32,
            "n" : 262144,
            "p" : 8,
            "r" : 1,
            "salt" : "ab0c7876052600dd703518d6fc3fe8984592145b591fc8fb5c6d43190334ba19"
        },
    },
    "id" : "3198bc9c-6672-5ab3-d995-4942343ae5b6",
    "version" : 4
}
```

## FAQs

**Why are keystores needed at all?**

Keystores provide a common interface for all clients to ingest validator credentials. By standardising this, switching between clients becomes easier as there is a common interface through which to switch.

**Why not reuse Eth1 keystores?**

* The keystores in Eth1 are more complicated than is needed and they rely on many different assumptions
* There are too many parameters and options in Eth1 keystores
* Eth1 keystores use Keccak256 which makes them unfriendly to other projects who wish to only rely on SHA256

**Why use scrypt over PBKDF2?**\

scrypt and PBKDF2 both rely on the security of their underlying hash-function for their safety (SHA256), however scrypt additionally provides memory hardness. The benefit of this is greater ASIC resistance meaning brute-force attacks against scrypt are generally slower and harder.

**Why are private keys encoded with Big Endian?**

This is done because it is how keys are stored in Eth1 and because it is is the standard of most of the crypto libraries.