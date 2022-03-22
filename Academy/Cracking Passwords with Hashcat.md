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
Hashcat has 5 different attack modes that have different applications depending on the type of hash you are trying to crack and the complexity of the password. The most straightforward but extremely effective attack type is the dictionary attack.

There's many large wordlists and past password breaches such as `rockyout.txt` or `CrackStation's Password Cracking Dictionary`, which contains 1,493,677,782 words and is 15GB in size.


##### Straight or Dictionary Attack
As the name suggests, this attack reads from a wordlist and tries to crack the supplied hashes. Dictionary attacks are useful if you know that the target organization uses weak passwords or just wants to run through some cracking attempts rather quickly.


**Hashcat - Syntax**
`hashcat -a 0 -m <hash-type> <hash-file> <wordlist>`

For example:
`hash -a 0 -m 1400 sha256_hash /opt/useful/SecLists/Passwords/...`

At any time during the cracking process, you hit the "`s`" key to get a status on the cracking job, which shows that to attempt every password in the `rockyout.txt` wordlist will take over 1.5 hours. Applying more rounds of the algorithm will increase cracking time exponentially. In the case of hashes such as bcrypt, it is often better to use smaller, more targeted, wordlists.

### Combination Attack
The combination attack modes take in two wordlists as input and create combinations from them. This attack is useful because it is not uncommon for users to join two or more words together, thinking that this creates a stronger password, i.e., `welcomehome` or `hotelcalifornia`.

Hashcat can combine wordlists together with the `--stdout` flag.

**Hashcat - Syntax**
Combination attack:

`hashcat -a 1 -m <hash type> <hash file> <wordlist1> <wordlist2>`

### Mask Attack
Mask attacks are used to generate words matching a specific pattern. This type of attack is useful when the password length or format is known. A mask can be created by using static characters, ranges of characters (e.g. [a-z] or [A-Z0-9]) or placeholders. The following list shows some important placeholders.

- ?l -> lower-case ASCII (a-z)
- ?u -> upper-case ASCII (A-Z)
- ?d -> digits (0-9)
- ?h -> 0123456789acbcdef
- ?H -> 0123456789ABCDEF
- ?s -> special characters (space!"#$%^&*()'+-/:;<=>?@[]{}'*")
- ?a -> ?l?u?d?s
- ?b -> 0x00 - 0xff

The above placeholders can be combined with options "`-1`" to "`-4`" which can be used for custom placeholders.

Consider the company Inlane Freight, which this time has passwords with the scheme `"ILFREIGHT<userid><year>"`, where userid is 5 characters long. The mask `"ILFREIGHT?l?l?l?l?l20[0-1]?d"` can be used to crack passwords with the specified pattern, where "`?l`" is a letter and `20[0-1]?d` which will include all years 2000 to 2019.

Attack mode is `-a 3`

Example syntax

`hashcat -a 3 -m 0 md5_hash -1 01 'ILREIGHT?l?l?l?l?l20?1?d'`

### Hybrid Mode
Hybrid mode is a variation of the combinator attack, wherein multiple modes can be used together for a fine-tuned wordlist creation. This mode can be used to perform very targeted attacks by creating very customized wordlists. it is particularly useful when you know or have a general idea of the organization's password policy or common password syntax. The attack mode for the hybrid attack is `6`.

Hashcat reads words from the wordlist and appends a unique string based on the mask supplied. In this case, the mask `?d?s` tells hashcat to append a digit and then a special character at the end of each word in the `rockyout.txt` wordlist.

Ex)

`hashcat -a 6 -m 0 hybrid_hash <wordlist> '?d?s'`

Attack mode `"7"` can be used to prepend characters to words using a given mask. The following example shows a mask using a custom character set to add a prefix to each word in the `rockyou.txt` wordlist. The custom character mask "`20?1?d`" with the custom character set "`-1 01`" will prepend various years to each word in the wordlist.

Ex)

`hashcat -a 7 -m 0 hybrid_hash -1 01 '20?1?d' <wordlist>`

### Creating Cutom Wordlists
Many open-source tools help in creating customized passwords wordlists based on our requirements.

##### Crunch
Crunch can create wordlists based on parameters such as words of a specific length, a limited character set, or a certain pattern. It can generate both permutations and combinations.

It is installed by default on Parrot OS.

**Syntax**

`crunch <min length> <max length> <charset> -t <pattern> -o <output file>`

The `-t` option is used to specify the pattern for generated passwords. The pattern can contain `@` representing lowercase characters `,` will insert upper case characters, `%` will insert numbers and `^` will insert symbols.

##### CUPP
CUPP stands for `Common User Password Profiler` and is used to create highly targeted and customized wordlists based on informationi gained from social engineering and OSINT. People tend to use peronsal information while creating passwords, such as phone numbers, pet names, birth dates, etc. CUPP takes in this information and creates passwords from them. These wordlists are mostly used to gain access to social media accounts.

`cupp -i`

##### KWPROCESSOR
A tool that creates wordlists with keyboard walks. Another common password generation technique is to follow patterns on the keyboard. These passwords are called keyboard walks, as they look like a walk along the keys.

The help menu shows the various options supported by kwp. The pattern is based on the geographical directions a user coudl choose on the keyboard. For example, the `--keywalk-west` option is used to specify movement towards the west from the base character. The program takes in base characters as a parameter, which is the character set the pattern will start with. Next, it needs a keymap, which maps the locations of keys on language-specific keyboard layouts. The final option is used to specify the route to be used. A route is a pattern to be followed by passwords. It defines how passwords will be formed, starting from the base characters.

**Example**
`kwp -s 1 basechars/full.base keymaps/en-us.keymap routes/2-to-10-max-3-direction-changes.route`

The command above generates words with characters reachable while holding shift (`-s`), using the full base, the standard en-us keymap, and 3 direction changes route.

##### Pinceprocessor
PRINCE or `Probability INfinite Chained Elements` is an efficient password guessing algorithm to improve cracking rates.

The `PRINCE` algorithm considers various permutation and combinations while creating each word. The binary can be downloaded from the releases page.

##### CeWL
Another tool to create custom wordlists. It spiders and scrapes a website and creates a list of the words that are present. This kind of wordlist is effective, as people tend to use passwords associated with those topics. This is due to human nature, as such passwords are also easy to remember. Organizations often have passwords associated with their branding and industry-specific vocabulary.

**Syntax**
`cewl -d <depth to spider> -m <min-length> -w <output> <url-of-website>`

Can also extract emails from websites with the `-e` option.

### Working with Rules

