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
The rule-based attack is the most advanced and complex password cracking mode. Rules help perform various operations on the input wordlist, such as prefixing, suffixing, toggling case, cutting, reversing, and much more. Rules take mask-based attacks to another level and provide increased cracking rates. Additionally, the usage of rules saves disk space and processing time incurred as a result of larger wordlists.

A rule can be created using functions, which take a word as input and output its modified version. The following table describes functions which are compatible with JtR as well as Hashcat.

**Function -> Description -> Input -> Output**
- l -> convert all letters to lowercase -> InlaneFreight2020 -> inlanefreight2020
- u -> convert all letters to uppercase -> InlaneFreight2020 -> INLANEFREIGHT2020
- c / C -> capitalize / lowercase first letter and invert rest
- t / TN -> Toggle case: whole word / at position N
- d / q / zN / ZN -> Duplicate word / all characters / first character / last character
- {/} -> Rotate word left / right
- ^X / $X -> Prepend / Append character X
- r -> Reverse

### Common Hash Types
During pentest engagements, we encounter a wide variety of hash types; some are extremely common and seen on most engagements, while others are seen very rarely or not at all.

**Hashmode -> Hash Name -> Example Hash**
- 0 -> MD5 -> `8743b52063cd84097a65d1633f5c74f5`
- 100 -> SHA1 -> `b89eaac7e61417341b710b727768294d0e6a277b`
- 1000 -> NTLM -> `b4b9b02e6f09a9bd760f388b67351e2b`
- 1800 -> sha512crypt -> `$6$52450745$k5ka2p8bFuSmoVT1tzOyyuaREkkKBcCNqoDKzYiJL9RaE8yMnPgh2XzzF0NDrUhgrcLwg78xs1w5pJiypEdFX/`
- 3200 -> bcrypt Blowfish(Unix) ->`$2a$05$LhayLxezLhK1LhWvKxCyLOj0j1u.Kj0jZ0pEmm134uzrQlFvQJLF6`
- 5500 -> NetNTLMv1 -> `u4-netntlm::kNS:338d08f8e26de93300000000000000000000000000000000:9526fb8c23a90751cdd619b6cea564742e1e4bf33006ba41:cb8086049ec4736c`
- 5600 -> NetNTLMv2 -> `admin::N46iSNekpT:08ca45b7d7ea58ee:88dcbe4446168966a153a0064958dac6:5c7830315c7830310000000000000b45c67103d07d7b95acd12ffa11230e0000000052920b85f78d013c31cdb3b92f5d765c783030`
- 13100 Kerberos 5 TGS -> `$krb5tgs$23$user$realm$test/spn$63386d22d359fe42230300d56852c9eb$ < SNIP >`

##### NetNTLMv2
During a pentest it is commonto run tools such as *Responder* to perform MITM attacks to attempt to "steal" credentials. In busy corporate networks it is common to retrieve many NetNTLMv2 password hashes using this method.

### Cracking Miscellaneous Files & Hashes

It's common to come across password protected documents such as Microsoft Word and Excel documents, KeePass database files, SSH private keys, PDF files, zip files, etc.

##### Tools
Various tools come with an option to output in a format that `Hashcat` can understand. `JohnTheRipper` also comes with a lot of tools to convert things to a format that `John` can crack.

##### Examples

Microsoft Office Documents -> *office2john.py* tool

Protected Zip Files -> *zip2john* tool

KeePass Files -> *keepass2john.py* tool

PDF files -> *pdf2john.py* tool


### Cracking Wireless (WPA/WPA2) Handshake
Another example is a wireless security assessment. Clients often ask for wireless assessments as part of an internal penetration test. While wireless is not always the most exciting, it can get interesting if you can capture a WPA/WPA2 handshake.

`Hashcat` can be used to successfuly crack both the MIC (4-way handshake) and PMKID (1st packet/handshake)

##### Cracking MIC
When a client connecting to the wireless network and the wireless access point (AP) communicate, they must ensure that they both have/know the wireless network key but are not transmitting the key across the network. The key is encrypted and verified by the AP.

These keys are used to generate a common key called the Message Integrity Check (MIC) used by an AP to verify each packet has not been compromised and receieved in its original state.

The 4-way handshake is illustrated in the following diagram:
![[Pasted image 20220503110755.png]]

Once you capture a 4-way handshake with a tool like `airodump-ng` you need to convert it to a form that can be supplied to `hashcat` for cracking. The format required is *hccapx*, and hashcat hosts an online service to convert to this format: *cap2hashcat online*. To perform offline, you need `hashcat-utils`

##### Cracking PMKID
This attack can be performed against wireless networks that use WPA/WPA2-PSK (pre-shared key) and allows us to obtain the PSK being used by the target wireless network by attacking the AP directly. The attack does not require deauthentication (deauth) of any users from the target AP. The PMK is the same as in the MIC (4-way handshake) attack but can generally be obtained faster and without interrrupting users.

The Pairwise Master Key Identifier  (PMKID) is the AP's unique identifier to keep track of the Pairwise Master Key (PMK) used by the client. The PMKID is located in the 1st packet of the 4-way handshake and can be easier to obtain since it does not require capturing the entire handshake. The PMKID is calculated with HMAC-SHA1 with the PMK (Wireless network password) used as a key, the string "PMK Name", MAC address of the access point, and the MAC address of the station.

![[Pasted image 20220503111541.png]]

To perform PMKID cracking, we need to obtain the mpkid hash. The first step is extracting it from the capture (.cap) file using a tool such as `hcxpcaptool` from `hcxtools` You can install `hcxtools` with `sudo apt install hcxtools`

Example:
`hcxpcaptool -z pmkidhash_corp cracking_pmkid.cap`

Hashcat - cracking PMKID
`hashcat -a 0 -m 22000 pmkidhash <wordlist>`


### Skills Assessment

1. Your colleague performed a successful SQL injection and obtained a hash. You must first identify the hash type and then crack the hash and provide the cleartext value. -> `Flower1`
2. The cracked hash from the SQL injection attack paid off, and your colleague was able to gain a foothold on the internal network! They have been running the Responder tool and collecting a variety of hashes. They have been unable to perform a successful SMB Relay attack, so they need to obtain the cleartext password to gain a foothold in the Active Directory environment. Crack the provided NetNTLMv2 hash to help them proceed. `bubbles1`
3. Great! Your colleague was able to use the cracked password and perform a Kerberoasting attack. One of the Kerberos TGS tickets retrieved is for a user that is a member of the Local Administrators group on one server. Can you help them crack this hash and move laterally to this server? `p@ssw0rdadmin`
4. Your colleague was able to access the server and obtain the local SAM database's contents. One of the hashes is the Domain Cached credentials for a Domain Administrator user. This hash is typically very difficult to crack but, if successful, will grant full administrative control over the entire Active Directory Environment.