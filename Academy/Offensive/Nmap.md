# Network Enumeration With Nmap
#### Enumeration
This is the most critical part of all. The art, the difficulty, and the goal are not to gain access to our target computer. Instead, it is identifying all the ways we could attack a target we must find.

Most of the ways we can get access we can narrow down to the following two points
- Functions and/or resources that allow us to interact with the target and/or provide additional information
- Information that provides us with even more important info to access our target.

`Manual enumeration` is `critical` component. Many scanning tools simplify and accelerate the process however, these cannot always bypass security measures of the services.

#### Intro to Nmap
Network Mapper (`Nmap`) is an open-source network analysis and security auditing tool written in C, C++, Python and Lua. It's designed to scan networks and identify which hosts are available on the newrok, services and applications, including name and version, where possible. It can also identify the OS's of these hosts, and can determine if packet filters, firewalls, or intrusion detection systems (IDS) are properly configured.

#### Use Cases
The tool is one of the most used tools by network administrators and IT security specialists. It is used to:

    Audit the security aspects of networks
    Simulate penetration tests
    Check firewall and IDS settings and configurations
    Types of possible connections
    Network mapping
    Response analysis
    Identify open ports
    Vulnerability assessment as well.

#### Nmap Architecture
Nmap offers many different types of scans that can be used to obtain various results about our targets.
- Host discovery
- Port scanning
- Service enumeration and detection
- OS detection
- Scriptable interaction with the target service (Nmap scripting Engine)

#### Syntax
`nmap <scan types> <options> <target>`

#### Scan techniques
Nmap offers many different scan techniques to use:
- sS/sT/sA/sW/sM: TCP SYN/Connect()/ACK/Window/Maimon scans
- sU: UDP Scan
- sN/sF/sX: TCP null, FIN and Xmas scan
- --scanflags `<flags>`: Customize TCP scan flags
- sY/sZ: SCTP INIT/COOKIE-ECHO scan
- sO: IP protocol scan
- b `<FTP relay host>`: FTP bounce scan

The default scan technique unless otherwise specified is `-sS` which is the TCP-SYN scan. Nmap sends SYN flagged packets and does not establish a full TCP connection
- If the target sends back a `SYN-ACK` response, Nmap detects that the port is **open**
- If the packet receives a `RST` flag, it is an indicator the port is **closed**
- If Nmap does not receive a packet back, it will display it as `filtered`. Depending on the firewall configuration, certain packets may be dropped or ignored by the firewall.

#### Host Discovery
If we're doing an entire network, we should first of all, get an overview of which systems are online that we can work with. You can use various `Nmap` host discovery options. The most effective way is **ICMP echo requests**

###### Scan Network Range
`sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5`

###### Scan IP List
It's not uncommon to be provided with a IP list with the hosts we need to test. `Nmap` also gives us the option of working with lists and reading the hosts from this list instead of manually defining or typing them in.

A file with the hosts can be used in place of putting a singular IP or subnet using the `iL` option.

To do multiple IPs you can do them with a space or for example `10.129.2.18-20`

#### Scan Single IP
Same method as before but just a single IP address.

`sudo nmap 10.129.2.18 -sn -oA host`

###### Used Scanning Options
Disabled port scan `-sn` and so Nmap automatically does a ping scan with `ICMP Echo Requests (-PE)`. Once a request is sent, we usually expect an `ICMP reply` if the host is alive. You can check in the packets with `--packet-trace`

You can disable ARP pings by setting `--disable-arp-ping`

#### Host and Port Scanning
The information we need includes:
- `open` : connection to scanned port has been established
- `closed` :  Received a `RST` flag packet back
- `filtered` : Cannot correctly identify whether it is open or closed either no response back or there's an error code.
- `unfiltered` : Only occurs during the TCP-ACK scan and means that the port is accesible, but can't be determined if it is open or closed.
- `open|filtered` : If we do not get a response from a specific port. This indicates a firewall or packet filter may protect the port.
- `closed|filtered` : Only occurs in the IP ID idle scans and indicates that is was impossible to determine if the scanned port is closed or filtered by a firewall.

#### Discovering Open TCP Ports
By default, `Nmap` scans the top 1000 TCP ports with the SYN scan (-sS). This is set only to default when we run it as root because of the socket permissions required to create raw TCP packets. Otherwise, the TCP scan (-sT) is performed by default.

#### Nmap - Trace the Packets
`sudo nmap $IP -p 21 --packet-trace -Pn -n --disable-arp-ping`

With these options we see that the target machine sends a TCP packet with the RST flag set indicated it was closed.

#### Discovering open UDP ports
Some sys admins forget to filter the UDP ports in addition to the TCP ones. Since UDP is a stateless protocol and does not require a 3-way handshake like TCP, we do not receive any acknowledgement. Consequently the timeout is much longer, making the whole UDP scan (-sU) much slower than the TCP scan (-sT)

If we get an ICMP response with error code 3 (port unreachable), we know that the port is indeed closed. For all other ICMP responses, the scanned ports are marked as (open|filtered)

#### Saving the Results
###### Different Formats
While we run various scans, we should always save the results. They can be used later to examine or cross-reference.
- Normal Output (-oN) with the `.nmap` extension
- Grepable output (-oG) with the `.gnmap` extension
- XML output (-oX) with the `.xml` extension

If you specify the `-oA` it will output to all the different formats.

The tool `xsltproc` can be used to convert the .xml format output of the nmap scan to HTML so it can viewed in a browser.

`xsltproc target.xml -o target.html`

#### Service Enumeration
For us, it is essential to determine the application and its version as accurately as possible. We can use this information to scan for known vulnerabilities and analyze the source code for that version if we find it. An exact version number allows us to search for a more precise exploit that fits the service and the OS of our target.

###### Service Version Detection
It is reommended to perform a quick port scan first, which gives a small overview of the ports available. This causes significantly less traffic. We can dealwith these first and run a sport scan in the background, which shows all open ports (-p-). We can use the version scan to scan the specific ports for services and their versions (-sV)

To view the status of a scan you can press the [Space Bar] during the scan. You can also specifiy the verbosity level (-v or -vv) which will show information as it happens.

###### Banner Grabbing
Once the scan is complete, we will see all the TCP ports with the corresponding wervice and their versions that are active on the system. Primarily, Nmap looks at the banners of the scanned ports and prints them out. If it cannot identify versions through the banners, Nmap attempts to identify them through a signature-based matching system, but it significantly increases the scan's duration.

#### Nmap Scripting Engine
Nmap Scripting Engine (NSE) is another handy feature of Nmap. It provides us with the possibility to create scripts in Lua for interaction with certain services. There are a total of 14 different categories scripts can be in

- auth: determination of authentication credentials
- broadcast: used for host discovery by broadcasting
- brute: try to log in by brute-forcing credentials
- default: executed with the -sC option
- discovery: evaluation of accessible services
- dos: check services for denial of service vulnerabilities
- exploit: tries to exploit known vulnerabilities on scanned port
- external: use external services for further processing
- fuzzer: identify vulnerabilities and unexpected packet handling by sending different fields, which can take much time
- intrusive: can negatively affect the target system
- malware: checks if some malware infects the target system
- safe: defensive scripts that do not perform intrusive of destructive actions
- version: extension for service detection
- vuln: identify specific vulnerabilities

##### Default scripts
`sudo nmap <target> -sC`

##### Specific script categories
`sudo nmap <target> --script <category>`

##### Defined scripts
`sudo nmap <target> --script <script-name>,<script-name>,...`

##### Specifying Scripts
`sudo nmap <target> -p 25 --script banner,smtp-commands`

#### Vulnerability Assessment
now let us move ont o HTTP port 80 and see what info and vulnerabilities we can find using the `vuln` script category

`sudo nmap <target> -p 80 -sV --script vuln`

#### Performance
We can use various options to tell Nmap how fast (-T <1-5>), with which frequency (`--min-parallelism <number>`), which timeouts (`--max-rtt-timeout <time>`) the test packets should have, how many should be sent simultaneously (`--min-rate <num>`) and with the number of retries (`--max-retries <num>`) for scanned ports.

Generally Nmap starts with a high timeout (`--min-RTT-timeout`) of 100ms. A comparison of scanning a network with 256 hosts, including the top 100 ports.

##### Default Scan (39.44 seconds)
`sudo nmap <ip/24> -F`

##### Optimized RTT (12.29 seconds)
`sudo nmap <ip/24> -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms`

#### Max Retries
Another way to increase speed is to set the (`--max-retries`) option to 0. Default is 1

#### Rates
You can significantly speed up scan when setting minimum rate (`--min-rate <number>`) 

Default:
`sudo nmap <ip/24> -F -oN tnet.default`

Optimized:
`sudo nmap <ip/24> -F -oN tnet.minrate300 --min-rate 300`

#### Timing
Nmap offers six different timing templates (`-T <0-5>`) for us to use. These values (`0-5`) determine the aggressiveness of our scans. Default timing used is (`-T 3`)

- -T 0 / -T paranoid
- -T 1 / -T sneaky
- -T 2 / -T polite
- -T 3 / -T normal
- -T 4 / -T aggressive
- -T 5 / -T insane


#### Firewall and IDS/IPS Evasion
Nmap gives us many different wasy to bypass firewall rules and IDS/IPS. These methods include the fragmentation of packets, the use of decoys, and others that will be discussed.

##### Firewalls
It checks whether individual network packets are being passed, ignored or blocked. This mechanism is designed to prevent unwanted connections that could be potentially dangerous.

##### IDS/IPS
The intrusion detection system (IDS) and intrusion prevention system (IPS) are also software-based components. IDS scans the network for potential attacks, analyzes them and reports any detected attacks. IPS complements IDS by taking specific defensive measures if a potential attack should have been detected.

##### Determine Firewalls and their Rules
In most cases if a port is filtered, it is a firewall that has certain rules set to handle specific connections. The packets can either be dropped, or rejected. The dropped packets are ignored, and no response is returned to the host.

For rejected packets that are returned with an RST flag. These packets contain different types of ICMP error codes or contain nothing at all.

Such errors are:
- Net unreachable
- Net prohibited
- Host unreachable
- Host prohibited
- Port unreachable
- Proto unreachable

Nmap's TCP ACK scan (-sA) is much harder to filter for firewalls and IDS/IPS systems than regular SYN scans (-sS) or Connect Scans (-sT) because they only send a TCP packet with only the ACK flag set.

#### Detect IDS/IPS
Unlike firewalls the detection of these are much more difficult because these are passive traffic monitoring systems. `IDS` systems examine all connections between hosts. If the IDS finds packets containing the defined contents of specifications, the admin is notified and takes appropriate action in the worst case.

`IPS` systems take measures configured by the admin independently to prevent potential attacks automatically. It is essential to know that IDS and IPS are different and that IPS serves as a complement to IDS.

One method to determine whether such IPS system is present in the target network is to scan from a single host (VPS). If at any time this host is blocked and has no access to the target network, we know the admin has taken some security measures. Accordingly, you can continue from another VPS.

#### Decoys
There are cases in which admins block specific subnets from different regions in principle. For this reason, the Decoy scanning method (-D) is the right choice. Nmap generates various random IP addresses inserted into the IP header to disguise the origin of the packet sent. With this method, we can generate random a specific number (for example: 5) of IP addresses separated by a colon (:). Our real IP addr is then randomly placed between the generated IP addresses.

##### Scan using decoys
`sudo nmap <ip> -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5`


The spoofed packets are often filtered out by ISPs and routers, even though they come from the same network range. Therefore, we can also specify our VPS servers' IP addresses and use them in combination with "IP ID" manipulation to scan the target.

Another scenario is only individual subnets can have access, so we manually specify IP address (-S). Decoys can be used with SYN, ACK, ICMP scans and OS detection scans.

#### DNS proxying
Most DNS queries are passed in, on UDP port 53. TCP port 53 was previously for so-called "Zone Transfers" but many DNS requests are now made via TCP port 53. To specify DNS servers ourselves (`--dns-server <ns>,<ns>`) You can also use (`--source-port`) to get around IDS/IPS filter.

If we see firewall accepts TCP port 53, it is very likely that IDS/IPS filters might also be configured much weaker than others. You can test this by using `Netcat`

`ncat -nv --source-port 53 <ip> <port>`


## Firewall and IDS/IPS Evasion Labs

#### Easy Lab
We do not know, however, according to which guidelines these changes will be made. Our goal is to find out specific info from the given situations. We are only ever provided with a machine protected by IDS/IPS systems and can be tested. We have access to the status web page. We know that if a specific amount of alerts go off, we will be `banned`. Therefore we must test the target system as `quietly` as possible.

#### Medium Lab

`sudo nmap -Pn --disable-arp-ping -sU -sV -p 53 -T2 $IP --source-port 53 -D RND:5 -n --packet-trace`
Returned the banner with the flag HTB{...} on port 53

#### Hard Lab
Found open TCP port 50000 when doing an all port scan with the source port set to 53. Then, using `sudo nc -p 53 <ip> 50000` returned the flag in the banner.

