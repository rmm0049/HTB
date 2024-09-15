# Brute Forcing
A Brute Force attack is a method of attempting to guess passwords or keys by automated probing. An example of a brute-force attack is password cracking. Passwords are usually stored as a hash value.

Files that contain hashed passwords:

Windows -> unattend.xml, sysprep.inf, SAM
Linux -> shadow, shadow.bak, password

There are many tools and methods to utilize brute forcing:
- Ncrack
- wfuzz
- medusa
- patator
- hydra
- and others...

This module mainly focuses on `hydra` as it is one of the most common and reliable tools

Outline:
- Brute forcing basic HTTP auth
- Brute force for default passwords
- Brute forcing login forms
- Brute force usernames
- Creating personalized username and password wordlists based on our target
- Brute forcing services logins, such as FTP and SSH

### Password Attacks
Many web servers or individual contents on the web servers are still often used with the *Basic HTTP AUTH* scheme. Like in our case, we found such a webserver with such a path, which should arouse some curiosity

The HTTP specification provides two parallel authentication mechanisms:
1. *Basic HTTP AUTH* is used to authenitcate the user to the HTTP server
2. *Proxy Server Authentication* is used to authenticate the user to an immediate proxy server.

There are several types of password attacks, such as:
- Dictionary Attack
- Brute Force
- Traffic interception
- Man In the Middle
- Key Logging
- Social Engineering

Will mainly focus on Brute Force and Dictionary Attacks. 

##### Brute Force
Does not depend on a wordlist of common passwords, but it works by trying all possible character combinations for the length we specified. All of this shows that relying completely on brute force attacks is not ideal, and this is especially true for brute-forcing attacks that take place over the network, like in `hydra`

##### DIctionary Attack
Tries to guess passwords with the help of lists. The goal is to use a list of known passwords to guess an unkown password. This method is useful whenever it can be assumed that passwords with reasonable character combinations are used.

We can check out the `SecLists` repo for wordlists, as it has a huge variety of wordlists, covering many types of attacks. We can find password wordlists in our PwnBox `/opt/useful/SecLists/Passwords/`, and username wordlists in `/opt/useful/SecLists/Usernames`

##### Methods of Brute Force Attacks
- Online: Attacking a live application over network like HTTP, HTTPS, SSH, FTP, etc.
- Offline: Offline Password Cracking, when you attempt to crack a hash of encrypted password.
- Reverse: Also known as username brute-forcing, when you try a single common password with a list of usernames on a certain service.
- Hybrid: Attacking a user by creating a customized password wordlist, built using known intelligence about the user or the service

### Default Passwords
Basic HTTP Authentication usually responses with a HTTP 401 Unauthorized response code. We will resort to a Brute Forcing attack, as we don't have enough information to attempt a different type of attack

##### Hydra
It can test any pair of credentials and verify whether they are successful or not but in huge numbers and a very quick manner. 

##### Default Passwords
We can either provide different wordlists for the usernames and passwords and iterate over all possible username and password combinations. However, we should keep this as a last resort.

We can find a list of default password login paris in the SecLists repository as well.

Hydra options example:
- `-C ftp-betterdefaultpasslist.txt`: Combined Credentials wordlist
- `SERVER_IP`: Target IP
- `-s PORT`: Target Port
- `http-get`: Request Method
- `/`: Target Path

It's pretty common for administrators to overlook test or default accounts and their credentials. That is why it is alwasy advised to start by scanning for default credentials, as they are very commonly left unchanged. It is even worth testing for the top 3-5 most common default credentials manually, as it can very often be found to be used.

### Username Brute Force
We now know the basic usage of `hydra`, so let us try another example of attacking HTTP basic auth by using separate wordlists for usernames and passwords

##### Wordlists
One of the most commonly used password lists is `rockyou.txt` which has over 14 million unique passwords sorted by how commonly they are used.

##### Username/Password Attack
`hydra` requires 3 specific flags if the credentials are in one single list to perform a brute attack against a web service:
1. Credentials
2. Target Host
3. Target Path

`-L` for usernames wordlist, `-P` for passwords wordlist. `-f` to stop after first successful login. `-u` to try all users on each password instead of all passwords on user first

### Hydra Modules
After trying popular admin credentials such as admin:admin, if none grant us access, we can try password spraying. Reusing  an already cracked passsword for example.

##### Brute Forcing Forms
IN this situation there are only two types of `http` modules interesting for us:
1. http[s]-{head|get|post}
2. http[s]-post-form

The 1st module serves for basic HTTP authentication, while the 2nd module is used for login forms, like .php or .aspx and others. Sinch the file extension is this example is ".php" we should try the `http[s]-post-form` module. To decide which module we need, we have to determine whether the application uses GET or POST form. We can test this by trying to login in and pay attention to the URL. If we recognize that any of our input was pasted into the URL, the web app is using a GET form. Otherwise, it uses a POST form.

We need to supply three parameters:
1. URL path, which holds the login form
2. POST parameters for username/password
3. A failed/success login string, which lets hydra recognize whether the login attempt was successful or not

- URL path -> /login.php
- POST parameters -> /login.php:[user parameter]=^USER^&[password parameter]=^PASS^
- Can't login so we don't know how the page would look like after a success: `/login.php:[user parameter]=^USER^&[password parameter]=^PASS^:[FAIL/SUCCESS]=[success/failed string]`

##### Fail/Success String
To make it possible for `hydra` to distinguish between successfuly submitted credentials and failed attempts, we have to specify a unique string from the source code of the page we're using to log in. `hydra` will examine the HTML code of the response page it gets after each attempt, looking for the string we provided.

- Fail: FALSE : F=html_content
- Success : TRUE : S=html_content

If we provide a `fail` string, it will keep looking until the string is **not found** in the repsonse. Another way is if we provide a `success` string, it will keep looking until the string is **found** in the responses

On the login page we see a distinct string "Admin Panel". So we may be able to use `Admin Panel` as our fail string. However, this may lead to false-positives because if the Admin Panel also exists in the page after logging in, it will not work.

A better strategy is to pick something from the HTML source of the login page. What we have to pick should be very *unlikely* to be present after logging in, like the **login button** or the *password field*. Let's pick the login button, as it is fairly safe to assume that there will be no login button after logging in.

We can click [Ctrl + U] in Firefox to show the HTML page source

So, the syntax for the http-post-form is:
`"/login.php:[user parameter]=^USER^&[password parameter]=^PASS^:F=<form name='login'"` 

### Determine Login Parameters
We can easily find POST parameters if we intercept the login request with burp suite or take a closer look at the admin panel's source code

#### Using Browser
One of the easiest ways to capture a form's parameters is through using a browser's built in developer tools. Network tools with [Ctrl + SHIFT + E]

After trying to login with any credentials (test:test) to run the form, after which the Network Tools would show the sent HTTP requests. Once we have the requests, we can simply right-click on one of them, and select Copy > Copy POST data

Another option would be to use `Copy > Copy as cURL` which would copy the entire `cURL` command which we can use in the terminal to get the same thing

Now to input into hyrdra:
`"/login.php:username=^USER^&password=^PASS^:F=<form name='login'"`

#### Attacking a web form login
1. Try default credentials list that brute forces those
2. If those don't work, try to use a password list on a specified user such as `admin, administrator, wpadmin, root, adm...`

### Personalized Wordlists
To create a personalized wordlist for th user, we will need to collect some information about them. As our example here is a known public figure, we can check out their Wikipedia page or do a basic Google search to gather the necessary information. Even if this was not a known figure, we can still carry out the same attack and create a personalized list for them.

##### CUPP
This is a tool to create a custom password wordlist. Can be run in interactive mode by doing `cupp -i` and input information about the person and it will generate a list.

##### Password Policy
The personalied wordlist we generated is about 43,000 lines long. Since we saw the password policy when we logged in, we know that the password must meet the follow criteria:
1. 8 characters or longer
2. contains special characters
3. contains numbers

Use `sed` to filter out those that do not meet this criteria
`sed -ri '/^.{,7}$/d' william txt `
`sed -ri '/[!-/:-@\[-`\{-~]+/!d`]'`
`sed -ri '/[0-9]+/!d'`

##### Mangling
Many great tools do word mangling and case permutation quickly and easily, like *rsmangler* or *The Mentalist*. These tools have many other options, which can make any small wordlist reach millionsof lines long.

##### Custom Username wordlist
The person's username could be `b.gates` or `gates` or `bill` and many other possible variations.

There's a username generator from GitHub that will generate potential usernames from a first and last name.

`git clone https://github.com/21y4d/usernameGenerator.git`

### Service Authentication Brute Forcing

##### SSH Attack
The command to attack a login service is fairly straightforward. We simply have to provide the username/password wordlists, and add `service://SERVER_IP:PORT` at the end. As usual we will ad the `-u -f` flags. Finally, when we run the command for the first time `hydra`, will suggest that we add the `-t 4` flag for a max number of parallel attempts as many `SSH` limit the number of parallel connnections and drop other connections, resulting in many of our attempts being dropped.

Sample:
`hydra -L bill.txt -P william.txt -u -f ssh://<ip>:22 -t 4`

So, similarily to how we attacked `SSH` we can perform a similar attack on `FTP` on the local machine.