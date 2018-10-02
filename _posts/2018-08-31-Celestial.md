---
layout: post
title: "HackTheBox Celestial"
date: 2018-08-31 
description: "Writeup of Celestial HackTheBox machine"
tag: CTF
---   

Spoiler: the third Hack the Box machine have a fun vulnerability related to Node.js that allows remote code execution and then to elevate privileges it is only needed some information gathering in the user directory.

### Information gathering

As it could not be otherwise, we start by launching an **nmap** to discover what services are available on the remote machine (because this machine was particularly slow, the nmap command only analyzes one port).

![](/images/posts/Celestial/img1.png "nmap results")

In the first request the web server responds with an invalid page and sets a new cookie. The second request includes the cookie and the web server responds with a string *Hey Dummy 2 + 2 is 22*.

![](/images/posts/Celestial/img2.png "First request")

![](/images/posts/Celestial/img3.png "Second request")

If we decode the Base64 cookie with **Burp**, we can see that is a JSON object that contains the information that is used in the response (Dummy and 2).

![](/images/posts/Celestial/img4.png "Reading the cookie")

With these information we can deduce that the server is using elements from the cookie to create the response. To verify this we can generate a malformed JSON object and check which parser are using.

![](/images/posts/Celestial/img5.png "Modify the JSON")

The error trace shows us that the web server uses the Node.js module *node-serialize*. This module is vulnerable to remote code execution using a parameter with the content *_$$ND_FUNC$$_*.

![](/images/posts/Celestial/img6.png "Application parser error")

First of all we need to know what happens if we add a non expectet argument in the JSON object. The server dosn't crash, these means that the server is parsing our new argument.

![](/images/posts/Celestial/img7.png "Add field in JSON")

![](/images/posts/Celestial/img8.png "New field test")

### RCE using JSON vulnerability

The final test to see if we can execute code on the server will be a reverse ping to our machine. With this test we will also check if the machine can connect to us or have some firewall rules. This time we put in the new field a function that generates a new child process and executes **ping** to our machine.

![](/images/posts/Celestial/img9.png "RCE test JSON")

If we send the request to the server and open some network analysis tool (**tcpdump** or **wireshark**) we will see that the remote machine sends ICMP requests to our machine.

![](/images/posts/Celestial/img10.png "RCE test")

The following [script](https://github.com/ajinabraham/Node.Js-Security-Course/blob/master/nodejsshell.py) will generate a payload that opens a reverse connection and launch */bin/sh*. The script encode the payload with the character ascii value and is loaded with eval function.

![](/images/posts/Celestial/img11.png "Create reverse shell payload")

The JSON object is the same as the ping example, we only need to put the otput of the script in the *_$$ND_FUNC$$_function(){\<code here\>}()* and this will evaluate the JavaScript payload encoded with chars.

![](/images/posts/Celestial/img12.png "Reverse shell JSON")

Voilà! We have a reverse shell with the user *sun*.

![](/images/posts/Celestial/img13.png "Reverse shell")

### Privilege escalation

Now that we have a shell with a normal user privileges, it's time to start the local information gathering to elevate privileges. Searching in the home directory of the user *sun* we can see a souspicious file *output.txt*. This file is owned by *root* but we can read the content.

![](/images/posts/Celestial/img14.png "Sun home directory")

![](/images/posts/Celestial/img15.png "File owned by root")

Searching the content of the file recursively with *rgrep* we find a pyhton script in *Documents* folder with a print of the string. We can deduce that the *root* user is executing the script and sending the result to the *output.txt* file.

![](/images/posts/Celestial/img16.png "Searching script content")

To capture the flag is as easy as modifying the script that will execute root and read the flag or do some actions to elevate privileges permanently (for example change the root line in the */etc/shadow* file with some known password).

![](/images/posts/Celestial/img17.png "Change script")

Waiting 5 minutes, the *root* user executes the python script and this reads the content of the flag.

![](/images/posts/Celestial/img18.png "Root execution")
