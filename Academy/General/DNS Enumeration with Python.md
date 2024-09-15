### DNS Structure

Another significant source of info is the DNS. The components of DNS consist of:
- `Name Servers`
- `Zones`
- `Domain names`
- `IP addresses`

The NS contain so-called `zones` or `zone files`. It's like a telephone book for a city. These zone files contain IP addresses to specific domains and hosts.

#### Domain Structure

Fully qualified domain name (FQDN):

- `www.domain-A.com`

![[Pasted image 20220823154856.png]]

Each domain consists of at least two parts:
1. `Top-level Domain (TLD)`
2. `Domain name`

#### Recursive DNS Resolver

The recursive acts as an agent between the client and the name server. After the recursive resolver receives a DNS query from a web client, it responds to this query with cached data, or it sends a query to a root name server, followed by a query to a TLD name server and finally a final query to an authoratative name server.

#### Root Name Server

Thirteen root name servers can be reached under IPv4 and IPv6 addresses. An international non-profit org. maintains these root name servers called `ICANN`. The zone files contain all domain names and IP addresses of the TLDs. Every recursive resolver knows these 13 root name servers. These are the first stations in the search for DNS entries for each recursive resolver. Each of these root name servers accepts a recursive resolver query that contains a domain name.

You can find the 13 root name servers on the domain `root-servers.net` with their corresponding letter as a subdomain

```bash
dig ns root-servers.net | grep NS | sort -u
...
```

#### TLD Name Server

A TLD name server manages the info on all domain names that have the same TLD. These TLD name servers are the responsibility of the IANA and are managed by it.

#### Authoritative Name server

Authoritative name servers store DNS record info for domains. These servers are responsible for providing answers to requests from name servers with IP address and other DNS entries for a web page so the web page can be addressed and accessed by a client. The authoritative name server is the last step to get an IP address.

### DNS Zones

**Primary DNS server**

The primary DNS server is the server of the zone file, which contains all authoritative info for a domain and is responsible for administering this zone. The DNS records of a zone can only be edited on the primary DNS

**Secondary DNS server**

Contains read only copies of the zone file from the primary DNS server. These servers compare their data with the primary DNS server at regular intervals and thus serve as a backup server.

**Zone Files**

Two types of zone transfers
- `AXFR` - Asynchronous Full Transfer Zone - All entries of zone file
- `IXFR` - Incremental Zone Transfer - only changed and new DNS records

### DNS Records and Queries

Records -> Description

- `A` -> IPv4 address records
- `AAAA` -> IPv6 address records
- `CNAME` -> Canonical Name records
- `HINFO` -> Host information records
- `ISDN` -> Integrated Services Digital Network records
- `MX` -> Mail exchanger records
- `NS` -> Name Server records
- `PTR` -> reverse-lookup Pointer records
- `SOA` -> Start of Authority records
- `TXT` -> Text records

`dig` and `nslookup` are common DNS command line tools

##### NS Servers

We want to automate the enumeration process for potential zone transfers on misconfigured DNS servers to check for this vulnerability quickly with the tool we are creating.

```
dig NS inlanefreight.com
```

```
dig SOA inlanefreight.com
```

```
nslookup -type=SPF inlanefreight.com
```

```
nslookup -type=txt _dmarc.inlanefreight.com
```

### DNS Security

Many companies recognize the value of the DNS as an active `line of defense`, embedded in an in-depth and comprehensive security concept

`DNS Threat Intelligence` can be integrated with other open-source and other threat intelligence feeds. Analytics systems such as `EDR (Endpoint Detection and Response)` and `SIEM (Security Info and Event Management)` can provide a holistic and situation-based picture of the security situation.

##### DNSSEC

Domain Name System Security Extensions, designed to ensure the authenticity and integrity of data transmitted through the DNS by securing resource records with digital certificates. `Private keys` are used to sign the `resource records` digitally.

Each zone has its zone kes, each consisting of a `private` and a `public key`. `DNSSEC` specifies a new resource record type with the `RRSIG`. It contains the signature of the respective DNS record, and these used keys have a specific validity period and are provided with a `start` and `end data`

### DNS Enumeration

As with any service we work with and want to find out info

- `DNS Records`
- `Subdomains/Hosts`
- `DNS Security`

Techniques:
- OSINT
- Certificate Transparency
- Zone transfer

- Google Dorks
- VirusTotal
- DNSdumpster
- Netcraft

##### Certficiate Transparency

`Certificate Transparency (CT)` logs contain all certificates issued by a participating `Certificate Authority` for a specific domain. 

`ctfr.py` tool can be used to find these registered domains

##### Zone transfer

If the DNS server is vulnerable to having its zones transferred

```
dig axfr inlanefreight.com @10.129.2.67
```


### Python Modules

`dnspython` and `IPython` are tools

##### AXFR Function

1. want a function that tries to perform a zone transfer using the given domain and DNS servers
2. If the zone transfer ws successful, we want the foud subdomains to be added to the list
3. If an error occurs, we want to be informed of said error

#### Argparse

