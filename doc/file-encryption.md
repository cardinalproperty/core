This document serves to provide a detailed account of how files are encrypted
by default by Storj Core to promote interoperability between different
implementations of clients.

#### Sharding and Encryption Scheme

Before data is stored in the network, the Storj Core library automatically
handles file encryption, sharding, and key management for you.

In order of operations:

1. Complete file is encrypted with a unique key and initialization vector
2. File is demultiplexed (or "sharded") into individual chunks
3. Each shard is offered to the network and transferred
4. Key and IV is encrypted with a passphrase and stored locally

#### Key and Initialization Vector Generation

Files are encrypted using
[AES-256-CTR](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard). The
`key` is the result of a generated
[PBKDF2](https://en.wikipedia.org/wiki/PBKDF2). To generate the PBKDF2 value,
you will need a password and a salt. These two values are the result of some
randomly generated bytes:

* Password: 512 random bytes
* Salt: 32 random bytes

The key length of the PBKDF2 is 512 bytes, should use 25000 iterations, and use
SHA-512 as the digest. To create the cipher/decipher and initialization vector,
derive the SHA-256 hash of the resulting PBKDF2 for the cipher/decipher `key`
and use the first 16 bytes of the original salt as the `iv`.

The {@link KeyRing} class stores the original `password` and `salt` in an
encrypted JSON document (using a user supplied passphrase) keyed by the
ObjectId returned from [Storj Bridge](https://github.com/Storj/bridge) for the
file object. The cleartext JSON document has two properties: `pass` and `salt`.

```
{
  "pass": "c7c311ee213d10baefd620a004d76485190d82...",
  "salt": "6d33490c999e9d613ccf4b146446763df15de2..."
}
```

This JSON string is encrypted with AES-256-CBC and encoded as
[Base58](https://en.wikipedia.org/wiki/Base58). Each of those encrypted JSON
documents is stored in a directory called `key.ring/` and the file name is
the ObjectId returned from [Storj Bridge](https://github.com/Storj/bridge) for
the file object.

#### Portable Key Ring Format

A {@link KeyRing} created by Storj Core can be exported into a portable format,
which is simply a gzipped tape archive (`.tar.gz` or `.tgz`). Importing this
archive simply entails:

1. Decompress and unpack the archive
2. Decrypt each JSON document using the original passphrase
3. Encrypt each document with the passphrase of the target keyring
4. Move the files into the target keyring, optionally overwriting conflicts

#### References

* {@link DataCipherKeyIv}
* {@link EncryptStream}
* {@link DecryptStream}
* {@link KeyRing}