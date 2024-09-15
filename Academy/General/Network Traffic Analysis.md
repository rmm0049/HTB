# Network Traffic Analysis

`Network Traffic Analysis (NTA)` can be described as the act of examining network traffic to characterize common ports and protocols utilized, establish a baseline for our environment monitor and respond to threats, and ensure the greatest possible insight into our organization's network

Different forms:

- `Collecting` real-time traffic within the network to analyze upcoming threats
- `Setting` a baseline for day-to-day network communications
- `Identifying` and analyzing traffic from non-standard ports, suspicious hosts, and issues with networking protocols such as HTTP erros, problems with TCP, or other networking misconfigurations
- `Detecting` malware on the wire, such as ransomware, exploits, and non-standard interactions

#### Environment and Equipment

Common Traffic Analysis Tools
- `tcpdump`
- `tshark`
- `wireshark`
- `NGrep`
- `tcpick`
- `Network Taps`
- `Networking Span Ports`
- `Elastic Stack`
- `SIEMS`

#### BPF Syntax

*Berkeley Packet Filer (BPF)* syntax is a technology that enables a raw interface to read and write from the Data-Link layer.

![[Pasted image 20220528184102.png]]

##### 1. Ingest Traffic

Once we decided our placement, begin capturing traffic.

##### 2. Reduce Noise by Filtering

Need to filter out unnecessary traffic from our view can make analysis easier (Broadcast and Multicast traffic, for example)

##### 3. Analyze and Explore

Now is the time to start carving out data pertinent to the issue we are chasing down. Look at specific hosts, protocols, even things as specific as flags set in the TCP header. The following questions will help us:

1. Is the traffic encrypted or plain text?
2. Can we see users attempting to access resources to which they should not have access?
3. Are different hosts talking to each other that typically do not?

##### 4. Detect the Root Issue

1. Are we seeing any errors?
2. Use our analysis to decide if what we see is benign or potentially malicious?
3. Other tools like IDS and IPS can come in handy at this point. They can run heuristics and signatures against the traffic to determine if anything within is potentially malicious

##### 5. Fix and Monitor

## Networking Primer - Layers 1 - 4

![[Pasted image 20230221122438.png]]

**TCP Convesation**
- Ending the convo
	1. FIN, ACK
	2. FIN, ACK,
	3. ACK

## Networking Primer - Layers 5-7

##### HTTP

A stateless Application Layer protocol that has been in use since 1990, HTTP enables the transfer of data in clear text between a client and server over TCP. The client would send an HTTP request to the server, asking for a resource. A session is established, and the server responds with the requested media

Methods:
- HEAD
- GET
- POST
- PUT
- DELETE
- TRACE
- OPTIONS
- CONNECT

##### HTTPS

HTTP over SSL

Has TLS Handshake to establish a connection


    1. Client and server exchange hello messages to agree on connection parameters.
    2. Client and server exchange necessary cryptographic parameters to establish a premaster secret.
    3. Client and server will exchange x.509 certificates and cryptographic information allowing for authentication within the session.
    4. Generate a master secret from the premaster secret and exchanged random values.
    5. Client and server issue negotiated security parameters to the record layer portion of the TLS protocol.
    6. Client and server verify that their peer has calculated the same security parameters and that the handshake occurred without tampering by an attacker.

#### FTP

TCP Port 20 (data) and 21 (issuing commands)

Runs in `active` or `passive`. 
- Active is default
- passive can be used to access FTP servers located behind firewalls or a NAT-enabled link
- The client would send `PASV` command

![[Pasted image 20230221124242.png]]

## The Analysis Process

1. `What is the issue?`
	- Suspected breach? Networking issue?
2. `Define our scope and the goal`
	1. Target
	2. When
	3. Supporting info
3. Define our target(s) (net / hosts(s) / protocol )
	1. Scope: 192.168.100.0/24

##### Diagnostic Analysis

4. `Capture network traffic`
	1. Plug into a link with access to the network to capture live traffic and try to grab one of the executables in the transfer
5. `ID of required network traffic components (filtering)`
	1. Filter out any packets not needed for this investigation to include
6. `An undestanding of capture network traffic`
	1. Dig for our target-filter on things like `ftp-data`

##### predictive Anlaysis

7. `Note-taking and mind maping of the found resulting`
	1. Annotating everything that's done
	2. Timeframes
	3. Suspicious hosts
	4. COnversations containing the files in question
8. `Summary of the analysis`

#### Prescriptive Analysis

# tcpdump Fundamentals

`tcpdump` is a CLI packet sniffer that can directly capture and interpret data frames from a file or network interface. It was built for use on any Unix-like OS and had a Windows twin called `WinDump`.

Requires `root` level privileges to run

#### Traffic Capture with tcpdump

Switches that can be used to modify how our captures run. These switches can be chained together to craft how the tool output is shown to us in STDOUT and what is saved to the capture file.

- `-D` will display any intefaces available to capture from
- `-i` selects the interface to listen on
- `-n` don't resolve hostnames
- `-nn` do not resolve hostnames or well-known ports
- `-e` will grab the ethernet header along with upper-layer data
- `-X` Show Contents of packets in hex and ASCII
- `-XX` Will also specify ethernet headers (like Xe)
- `-v, -vv, -vvv` Increasing level of verbosity
- `-c` grab a specific amount of packets
- `-s` how much of a packet to grab
- `-S` change relative sequence numbers to absolute sequence numbers
- `-q` print less protocol information
- `-r <file>` read from file
- `-w <file>` write to file

##### tcpdump output

##### file input/output with tcpdump

`tcpdump -i eth0 -w output.pcap`

`tcpdump -r output.pcap`

## Filtering and Advanced Syntax Options

![[Pasted image 20230221134801.png]]

##### Looking for TCP Protocol Flags

Ex)
`tcpdump -i eth0 'tcp[13] & 2 != 0'`

Counting to the 13th byte in the structure and looking at the 2nd bit. If it is se to 1, or ON the SYN flag is set.

# Wireshark

### Decrypting RDP Connections

The purpose of this lab is to give a taste of the power Wireshark. In this lab, we will be working with RDP traffic. If one has the required key utilized between the two hosts for encrypting the traffic, Wireshark can deobfuscate the traffic for us

If you acquire the RDP certificate from the server, `OpenSSL` can pull the private key out of it


