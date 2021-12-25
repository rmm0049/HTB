### Introduction
Code deobfuscation is an important skill to learn if we want to be skilled in code analysis and reverse engineering. It's common to come across obfuscated code that wants to hide certain functionailities.

Topics:
- Locating JavaScript code
- Intro to Code Obfuscation
- How to Deobfuscate JavaScript code
- How to decode encoded messages
- Basic Code Analysis
- Sending basic HTTP requests

### Source Code
Most websits nowadays utilize JavaScript to perform their functions. While `HTML` is used to determine the website's main fields and parameters, and `CSS` is used to determine its design, `JavaScript` is used to perform any functions necessary to run the website.

### What is obfuscation
It is a technique used to make a script more difficult to read by humans but allows it function the same from a technical point of view, though might be slower. Code written in many languages are published and executed without being compiled in `interpreted` languages, such as `Python`, `PHP`, and `JavaScript`. While Python and PHP usually reside on the server-side and hence are hidden from end-users, JavaScript is usually used within brwosers on the client-side, which is why it's often obfuscated.

### Use Cases
Many reasons to do this, to prevent code from being reused or copied without developers permission, or dealing with authentication or encryption. However, it is not recommended to deal with authentication or encryption on the client side becasue it more prone to being attacked. The most common reason though for obfuscation is for malicious reasons.

### Basic Obfuscation
It is not usually done manually, as there are many tools for various languages to do automated code obfuscation.

### Running JavaScript code
Take the line of JavaScript:
```scheme
console.log('HTB JavaScript Deobfuscation Module');
```

### Minifying JavaScript Code
A common way of reducing the readability of a snippet of JS code while keeping it fully functional is JS minification. It usually means having the entire code in a single (often long) line.

### Advanced Obfuscation
This section uses a couple of tools that should completely obfuscate the code and hide any remanence of its original functionality.

### Deobfuscate
Tools like **JSNice** will help deobfuscate after beautifying the code.


### Code Analysis
Now that we have deobfuscated the code, we can start going through it

```scheme
'use strict';
function generateSerial() {
  ...SNIP...
  var xhr = new XMLHttpRequest;
  var url = "/serial.php";
  xhr.open("POST", url, true);
  xhr.send(null);
};
```

We see that the `secret.js` file contains only one function, `generateSerial`

##### HTTP requests
Let us look at each line of the `generateSerial` function.

The function starts by defining a variable, *xhr* which creates an object of *XMLHttpRequest*. This is a JavaScript function that handles web requests.

The second variable defined is *url* which contains a URL to */serial.php* which should be on the same domain, as no domain is specified.

Next, we see that the xhr.open is used with "POST" and URL. We see that is opens the HTTP request defined 'GET' or 'POST' to the URL, and then the next line xhr.send would send the request.

### Decoding
Often things are double or triple encoded after being obfuscated. The three most common ways are
- base64
- hex
- rot13

##### base64
Is usually used to reduce the use of special characters, as any characters encoded in base64 would be represented in alphanumeric in addition to the + and / only.

##### Hex
Hexadecimal format and can be spotted when A-F is used and numbers.

##### rot13
Another common way is the caesar cipher, which shifts each letter by a fixed number. There isn't a specific command in Linux to do rot13 encoding however it's fairly easy to do the character shifting.

`echo https://www.hackthebox.eu/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'`

to decode

`echo uggcf://jjj.unpxgurobk.rh/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'`

