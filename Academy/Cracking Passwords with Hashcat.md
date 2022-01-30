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

