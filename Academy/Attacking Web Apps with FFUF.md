# Attacking Web Apps with FFUF
#### Introduction
There are many tools and methods to utilize for directory and parameter fuzzing/brute-forcing. This room focuses on `ffuf` tool for web fuzzing, as it is one of the most common and reliable tools available for web fuzzing

- Fuzzing for directories
- Fuzzing for files and extensions
- Identifying hidden vhosts
- Fuzzing for PHP parameters
- Fuzzing for parameter values

#### Web Fuzzing
We will start by the basics of `ffuf` to fuzz websites for directories. The term `fuzzing` refers to a testing technique that sends various types of user input to a certain interface to study how it would react. If we were fuzzing for a buffer overflow, we would be sending long strings and incrementing their length to see if and when the binary would break.

Usually utilize pre-defined wordlists of commonly used terms for each type of test for web fuzzing to see if the webserver would accept them. This is done because web servers do not usually provide a directory of all available links and omains (unless terribly configured), and so we would have to check for various links and see which ones return pages.

#### Wordlists
To determine which pages exist, we should have a wordlist containing commonly used words for web directories and pages, very similar to a `Password Dictionary Attack`. Though this will not reveal all pages under a specific website, in general it returns the majority of pages, reaching up to 90% success rate on some websites.

#### Directory Fuzzing
The main two options are `-w` for wordlists and `-u` for URL. We can assign a keyword to a wordlist to refer to it where we want to fuzz. For ex, we can can pick our wordlist and assign the keyword `FUZZ` by adding `:FUZZ` after it.

Sample command
`ffuf -w <path-to-wordlist>:FUZZ -u http://SERVER_IP:PORT/FUZZ`

#### Page Fuzzing
Extension fuzzing can be used to see if the directories have any hidden pages. However, before we start, we must find out what types of pages the website uses, like `.html, .aspx, .php` or something else.

One of the most common ways to identify that is by finding the server type through the HTTP response headers and guessing the extension. For example, if the server is `apache` then it may be `.php` or if it was `IIS`, then it could be `.asp` or `.aspx` and so on. This method is not very practical though, so we will again utilize `ffuf` to fuzz the extension, similar to how we fuzzed for directories. Instead of placing the `FUZZ` keyword where the directory name would be, we would place it where the extension would be `.FUZZ` and use a wordlist of common extensions.

**Note**: The wordlist we chos already contains a dot (.) so we will not have to add the dot after the directory name in fuzzing.

Sample:
`ffuf -w <path-to-extensions>:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ`

We do get a couple of hits, but only `.php` gives us a response with code `200`. Now we know that `PHP` runs on this server.

#### Page Fuzzing
We will now use the same concept of keywords we've been using `ffuf`, use `.php` extensions, place our `FUZZ` keyword where the filename should be, and use the same wordlist we used for fuzzing directories.

Now that we combine them...
`ffuf -w <path-to-wordlist>:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php`

#### Recursive Fuzzing
So far, we have been fuzzing for directories, then going under these directories, and then fuzzing for files. However, if we had dozens of directories, each with their own subdirectories and files, this would take a very long time to complete. To be able to automate this, we will utilize `recursive fuzzing`

##### Recursive flags
When we scan recursively, it automatically starts another scan under any newly identified directories that may have on their pages until it has fuzzed the main website and all of its subdirectories

It's always advised to specify a `depth` to the scan to tell how far to look in the chain of directories.

In `ffuf` you can enable the recursive scanning with the `-recursion` flag, and specify the depth with `-recursion-depth` flag. And you can specify the extension with the `-e` flag with `.php` for example.

## Domain Fuzzing
#### DNS Records
Once we accessed the page under `/blog` we got a message saying `Admin panel moved to academy.htb`. If we visit the website we get 'can't connect to the server at www.academy.htb'. 

We need to add this locally to the `/etc/hosts` file because the domain is not public and it will not be able to resolve it.

`sudo sh -c 'echo SERVER_IP academy.htb >> /etc/hosts'`

#### Sub-domain Fuzzing
Use `ffuf` to identify sub-domains (i.e., `*.website.com`) for any website

A sub-domain is any website underlying another domain. For example, `https://photos.google.com` is in the `photos` sub-domain of `google.com`

In this case, we are simply checking different websites to see if they exist by checking if they have a public DNS record that would redirect us to a working server IP. So, let's run a scan and see if we get any hits. Before we can start our scan, we need two things:
- A `wordlist`
- A `target`

Luckily for us, in the `SecLists` repo, there is a specific section for sub-domain wordlists, consisting of common words usually used for sub-domains in `/SecLists/Discovery/DNS` In our case, we would be using a shorter wordlist, which is `subdomains-top1million-5000.txt`.

## Vhost Fuzzing
When it comes to fuzzing sub-domains that don't have a public DNS record or sub-domains under websites, we could not use the same method, we need to do VirtualHost fuzzing.

#### Vhosts vs. Sub-domains
The key differences between Vhosts and sub-domains is that a Vhost is basically a 'sub-domain' served on the same server and has the same IP, such that a single IP could be servering two or more different websites.

`Vhosts may or may not have public DNS records`

In many cases, many websites would actually have sub-domains that are not public and will not publish them in public DNS records, and hench if we visit them in a browser, we would fail to connect. 

To do Vhost fuzzing, without manually adding the entire wordlist to our `/etc/hosts` file locally, we can use the `-H` flag to specify a header and will use the `FUZZ` keyword within it, as follows:

Sample:
`ffuf -w <sub-domain-wordlist>:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'`

## Filtering Results
So far, we have not been using any filtering to our `ffuf`, and the results are automatically filtered by default by their HTTP code, which filters out code `404 NOT FOUND`, and keeps the rest. However, as we saw in our previous run of `ffuf`, we can get many responses with code `200`. So, in this case we have to filter the results based on another factor, which we will learn in this section.

In this case, we cannot use matching, as we don't what the response size from what other Vhosts would be. We know the response size of the incorrect results though, as seen from the test above, is `900` we can filter it out with `fs 900`.

**Note:** Don't forget once you find a Vhost to add it to your local `/etc/hosts`

## Parameter Fuzzing - GET
If we run a `ffuf` scan on `admin.academy.htb` we should find `http://admin.academy.htb:PORT/admin/admin.php`

We don't have access on this page to see the flag, such keys would usually be passed as a `parameter`, using either `GET` or a `POST` HTTP request. This section will discuss how to fuzz for such parameters until we identify a parameter that can be accepted by the page.

#### GET request fuzzing
Similarly to how we have been fuzzing various parts of the website, we will use `ffuf` to enumerate parameters. GET parameters can be passed right after the URL with the `?` symbol:

`http://admin.academy.htb:PORT/admin/admin.php?param1=key`

A good `SecLists` wordlist is in `/Discovery/Web-Content/burp-parameter-names.txt`

Again, you can filter by the size of the response to find the right one.

## Parameter Fuzzing - POST
To fuzz the `data` field with `ffuf` we can use the `-d` flag, as we saw previously in the output of `ffuf -h`. We also have to add `-X POST` to send `POST` requests.

**Tip**: In PHP, "POST" data "content-type" can only accept "application/x-www-form-urlencoded". So, we can set that in "ffuf" with the -H 'Content-Type:application/x-www-form-urlencoded'

## Value Fuzzing
After fuzzing for a working parameter, we now have to fuzz the correct value that would return the `flag` content we need.

#### Custom Wordlist
We can look for custom wordlists in the `SecLists` but we can guess the `id` parameter takes an integer value so we can make a list of numbers from 1-10000... to try them.

Simpliest way to do this is real quick in Bash:
`for i in {1..1000}; do echo $i >> ids.txt; done`

#### Value Fuzzing
Our command should be fairly similar to the `POST` command we used to fuzz for parameters, but our `FUZZ` keyword should be placed as the value of the parameter.



**Notes for practical**

three virtual hosts -> archive, faculty, test -> {}.academy.htb

archive.academy.htb -> directory **/courses**

faculty.academy.htb -> directories

