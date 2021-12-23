# HyperText Transfer Protocol
HTTP is an application level protocol used to access resources over the World Wide Web. The term `hypertext` stands for links to other resources and text that can be easily interpreted by readers.

HTTP communication is based on the client-server model, where the client requests a resource from the server. The server processes the request and returns the requested resource back to the client. The default port for HTTP traffic is port 80, however, it can be changed. We enter a **Fully Qualified Domain Name**(FQDN) as a **Uniform Resource Locator** (URL) to reach the desired website.

![[Pasted image 20211222154900.png]]

Not all components are required to access a resource, however, a URL should at least contain a scheme and host to make a proper request.

### HTTP Flow
![[Pasted image 20211222155028.png]]

The first time a user enters a URL (inlanefreight.com) to the browser, it queries DNS (Domain Name System) server to resolve the domain. The DNS server looks up the IP address of the domain and returns it to the browser. 

Next the browser sends a GET request to the default HTTP port, i.e. 80, asking for the root `/` folder. Here `GET` is the request method. The type of request can vary. 

### HTTPS Overview
One of the major drawbacks of HTTP is that it transmits its data in cleartext, meaning anyone who can see the traffic would be able to see all of the unencrypted data. These drawbacks gave rise to the HTTPS (HTTP Secure) protocol. When this protocol is enabled, all communication between the client (user accessing a web application via their web browser), and the webserver that hosts the web application, is encrypted. The default port for HTTPS is `443`, which is preffered now in browsers over HTTP port 80.

### HTTPS Flow
![[Pasted image 20211222155455.png]]

Upon browsing to http://inlanefreight.com, the browser attempts to resolve the domain and redirects the user to the webserver hosting the target website. A request is sent to port 80, first which is the unecrypted protocol. The server detects this and redirects them to use HTTPS port 443 and returns a `301 Moved Permenently` response code.

Next, the client sends a *Client Hello* packet giving the information about itself. After this, the server responds with a *Server hello*, followed by a key exchange. The client verifies this key and sends one of its own. After this, an encrypted handshake is intitiated to verify the encryption and transfer are working properly.

### HTTP Request
Raw HTTP Request:

![[Pasted image 20211222155845.png]]

### HTTP Response
![[Pasted image 20211222155941.png]]

### Headers
HTTP headers provide an additional way to pass information between the client and the server. There are headers specific to requests and responses as well as general headers common to both. Headers can have one or multiple values appended after the header name and separated by a colon. There are many headers used for different purposes, which are divided into different categories.

1. General Headers
2. Entity Headers
3. Request Headers
4. Response Headers
5. Security Headers

#### 1. General Headers
Don't belong specifically to a request or a response. They are contextual and are used to describe the message rather than its contents.

- Date: holds the date and time at which the message originated. It's preferred to convert the time to the standard UTC time zone
- Connection: dictates if the current network connection should stay alive after the request finishes. Is either *closed* or *keep-alive*

#### 2. Entity Headers
Similar to general headers, entity headers can be common to both the request and response. These are used to describe content being transferred by a message. They are usually found in responses and POST or PUT requests (for example, a file upload)

- Content-Type: Describes the type of resource being transferred
- Media-Type: Describes the data being passed. PDF is `application/pdf`, charset field denotes encoding standard, such as *UTF-8*
- Boundary: Acts as a maker to separate content when there is more than one in the same message
- Content-Length: Holds the size of the entity being passed.
- Content-Encoding: Type of encoding is specified here

#### 3. Request Headers
The client sends request headers in the HTTP transaction defined in RFC 2616.

- Host: specifies the host being queried for the resource
- User-Agent: describes the client requesting resources.
- Accept: Describes what types of media the client can understand
- Cookie: Cookie value pairs in the format of `name=value`
- Referer: denotes where the current request is coming from.
- Authorization: another way for server to identify clients

#### 4. Response Headers
- Server: contains info about the HTTP server, which handled the request.
- Set-Cookie: contains the cookies needed for client identification
- WWW-Authenticate: notifies the client about the type of authentication required to access the requested resource.

#### 5. Security Headers
- Content-Security-Policy: The CSP header dictates the website's policy towards externally injected resources. This could be JavaScript code as well as script resources.
- Strict-Transport-Security: prevents the browser from accessing the website over the plaintext HTTP protocol. All comms are done over HTTPS.
- Referrer-Policy: Dictates whether a browser should include the value specified via the `Referrer` header or not.


### HTTP Methods and Codes
Methods:
- GET: most common, which requests a specific resource
- POST: used to send data to the server.
- HEAD: requests the headers that would be returned if a GET request was made to the server.
- PUT: similar to POST, as it's used to create new resources on the server.
- DELETE: lets users delete an existing resource on the webserver.
- OPTIONS: returns information about the server, such as the methods accepted by it.

Response Codes:
- 1xx: usually provides info and continues processing the request
- 2xx: positive code returned when request succeeds
- 3xx: server redirects client
- 4xx: error or improper request from client
- 5xx: some problem with HTTP server itself.

### GET Method
The most commonly used method. Its main purpose is to request a given resource and retireve it. It is possible to send data via a GET request with the help of query parameters, but his should be avoided in place of the POST method.

### POST Method
Unlike the `GET` method, `POST` places user paramters in an HTTP Request body. This has three main benefits:
- *Lack of Logging*: POST requests can handle tasks such as file uploads and user logins. Logging these fields can reuslt in the local disk filling up or passwords being stored in plaintext. When coming across a login form that accepts logins via a `GET` request, this should always be a `HIGH` finding as it will likely result in credentials in log files
- *Less Encoding Requirements*: URLs are designed to be shared, which means they need to confirm to characters that can be converted to letters. The POST request places data in the body which can accept binary data.
- *More data can be sent*: Maximum URL length varies between browsers, web servers, CDNs and even URL shorteners. Generall speaking, URL's should be kept to below 2,000 characters.

### Content-Type
This is a crucial piece to the POST request. This tells the webserver what type of content to expect. In our examples above, it was sent to `application/x-www-form-urlencoded`. This tells the server to expect something along the lines of `param1=value1&param2=value2`

Another popular one is `application/json`, which is used for JSON (JavaScript Object Notation) resources.

### PUT and DELETE Methods

#### PUT Method
`PUT` and `DELETE` methods are disallowed in web server configurations by default as they can be risky and, if misued, casue damage or result in unintended consequences. The methods supported by a server can be done with a `OPTIONS` method request. This can be useful when enumerating the list of methods accepted by the server.

#### DELETE Method
Similarly, the `DELETE` method can be used to delete an existing file.


### cURL
CURL (client URL) is a command-line tool and library which primarily suppots HTTP along with many other protocols. This makes it a good candidate for scripts as well as automation.

##### GET Request
The default HTTP requests made by cURL are GET requests.

`curl http://inlanefreight.com/`

GET request verbose
`curl http://inlanefreight.com/ -v`

Basic AUTH login (can also use -u option)
`curl http://admin:password@inlanefreight.com/ -vvv`

Basic AUTH login & follow redirections
`curl -u admin:password -L http://inlanefreight.com/`

##### POST Request

Passing data
`curl -d 'username=admin&password=password' -L http://inlanefreight.com/login.php`

This command above returns us back to the login page even after supplying valid credentials, this is because it doesn't send the auth cookie to it.

The `--cookie` or `--cookie-jar` option can be used to specify cookie usage in cURL. This can point to /dev/null or a file on disk, where the cookies will be saved

Cookie
`curl -d 'username=admin&password=password' -L --cookie-jar /dev/null http://inlanefreight.com/login.php`

Cookie File
`curl -d 'username=admin&password=password' -L --cookie-jar cookies.txt http://inlanefreight.com/login.php`

or later without supplying credentials now

`curl --cookie cookies.txt http://inlanefreight.com/admin/dashboard.php -v`

Content-Type Header
`curl -H 'Content-Type: application/json -d '{"username": "admin", "password": "password"}' --cookie-jar /dev/null -L http://inlanefreight.com/login.php`

##### PUT and DELETE Methods
`curl -X OPTIONS http://inlanefreight.com/ -vv`

File upload
`echo "curl file upload" > test.txt`
`curl -X PUT -d @test.txt http://inlanefreight.com/test.txt -vv`

Deleting
`curl -X DELETE http://inlanefreight.com/test.txt -vv`
