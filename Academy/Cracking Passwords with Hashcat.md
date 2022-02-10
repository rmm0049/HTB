# Cracking Passwords with Hashcat
### Introduction

##### Password cracking overview
This is an effective way of gaining access to unauthorized resources. Various applications and systems make use of cryptographic algorithms to hash or encrypt data. Doing so prevents the storage of plaintext information in data and disclosure of transmitted data in man-in-the-middle attack (MITM) attack scenarios.

Hashcat is a popular and powerful tool used in performing password cracking against a wide variety of algorithms.

### Hashing vs. Encryption

##### Hashing
The process of converting some text to a string, which is unique to that particular string. Usually a hash function always returns hashes with the same length irregardless of the size of the input. Hashing is one-way process, meaning there is no way of reconstructing the original plaintext from a hash. Hashing can be used for various purposes; for example, the MD5 and SHA256 algorithms are usually used to verify file integrity, while algorithms such as PBKDF2 are used to hash passwords before storage. Some hash functions can be keyed. One such example is HMAC, which acts as a checksum to verify if a particular message was tampered with during transmission.

Hasing is a one-way process, meaning the only way to attack is to use a list containing possible passwords. Each possible password from the list is hashed and compared to the actual password hash.

##### Encryption
The process of converting data into a format in which the original content is not accessible. Unlike hashing, encryption is reversible, i.e., it's possible to decrypt the ciphertext (encrypted data) and obtain the original content. Some classic examples of encryption ciphers are the Caesar cipher, Bacon's cipher and Substitution cipher. Encryption algorithms are of two types: Symmetric and Asymmetric.

##### Symmetric Encryption
Uses a key or secret to encrypt the data and use the same key to decrypt it. A basic example of symmetric encryption is XOR

##### Asymmetric Encryption
Divides the key into two parts (i.e., public and private). The public key can be given to anyone who wishes to encrypt some information and pass it securely to the owner. The owner then uses their private key to decrypt the content. Some examples of asymmetric algorithms are **RSA, ECDSA, and Diffie-Hellman**

One of the most prominent asymmetric encryption is the `Hypertext Transfer Protocol Secure (HTTPS)` in the form of `Secure Sockets Layer (SSL)`. When a client connects via HTTPS, a public key exchange occurs. The client's browser uses this public key to encrypt any kind of data sent to the server. The server decrypts the incoming traffic before passing it on to the processing service.

### Identifying Hashes
Most hashing algorithms produce hashes of a constant length. The length of a particular hash can be used to map it to the algorithm it was hashed with. For example, a hash of 32 characters in length can be an MD5 or NTLM hash.

Sometimes hashes are stored in certain formats. For example `hash:salt` or `$id$salt$hash`

The hash `2fc5a684737ce1bf7b3b239df432416e0dd07357:2014` is a SHA1 hash with the salt of 2014.

The hash `$6$vb1tLY1qiY$M.1ZCqKtJBxBtZm1gRi8Bbkn39KU0YJW1cuMFzTRANcNKFKR4RmAQVk4rqQQCkaJT6wXqjUkFcA/qNxLyqW.U/`
contains three fields delimited by `$`, where the first field is the `id`, i.e.,`6`. This is used to identify the type of algorithm used for hashing.

- `$1$`: MD5
- `$2a$`: Blowfish
- `$2y$`: Blowfish, with correct handling of 8 bit characters
- `$5%`: SHA256
- `$6$`: SHA512

The next field is, `vb1tLY1qiY`, is the salt used during the hashing, and the final field is the actual hash.

Open and closed source software use many different kinds of hash formats. For example, the `Apahce` web server stores its hashes in the format `$apr1$...` and `Wordpress` uses the form `$P$...`

### Hashid
This is a Python tool, that can be used to detect hash types. Can also give you the Hashcat hash mode by using the `-m` option.

### Context is Important
It is important to ntoe that `hashid` uses regex to make a best-effort guess for the type of hash used. However, there still may be a certain level of guesswork left to identify a given hash. Knowing where the hash came from will help a great deal.

### Hashcat
A popular open-source password cracking tool.

The `-a` and `-m` arguments are used to specify the type of attack mode and hash type. `Hashcat` supports the following attack modes:

- 0: Straight
- 1: Combination
- 3: Brute-force
- 6: Hybrid Wordlist + Mask
- 7: Hybrid Mask + Wordlist

They hash type value is based on the algorithm of the hash to be cracked.

##### Hashcat - Benchmark
Uses the `-b` flag and will test for a particular hash type its performance

##### Hashcat - Optimizations
Hashcat has two main ways to optimmize speed:

- Optimized Kernels: This is the `-O` flag, which according to the documentation, means `Enable optimized kernels (limits password length)`.
- Workload: This is the `-W` flag, which, according to docs, meas `Enable a specific workload profile`. The default number is `2`, but if you want to use your computer while Hashcat is running, use `1`. If the computer only wants to do Hashcat, use `3`.

### Dictionary Attack


