---
layout: post
title: "HackTheBox Valentine"
date: 2018-08-15 
description: "Writeup of Valentine HackTheBox machine"
tag: CTF
---   

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Maecenas ut efficitur nisi. Nullam non risus a orci ultrices lacinia ac a purus. Etiam in imperdiet lectus. Interdum et malesuada fames ac ante ipsum primis in faucibus. Cras ultrices tellus quis sapien mollis, in luctus tortor tempor. Suspendisse potenti. Curabitur sit amet porta magna. Duis dignissim risus at scelerisque malesuada. Integer venenatis quis augue eget pellentesque. Phasellus iaculis sed lacus quis efficitur.

### Information gathering

We can not start in any other way than using **nmap** to discover what services are public on the machine. Our old friend show us that the machine is only running a SSH service and 2 web services.

![](/images/posts/Valentine/img1.png "nmap results")

Both of the web services have the same content, an image that contains the first clue. The web is only a landing page, we need to search more attack vectors. With *gobuste* we discover 3 more resources that can leet us to get the first shell.

![](/images/posts/Valentine/img2.png "Web image") 

![](/images/posts/Valentine/img3.png "Resources enumeration")

The *encode* and *decode* functionallites, as it's names says, only transform to Base64 the user input.  

![](/images/posts/Valentine/img4.png "Encode functionallity")

![](/images/posts/Valentine/img5.png "Decode functionallity")

The */dev/* directory contains 2 files, one contains a todo list and the other a suspicious encoded or ciphered text. 

![](/images/posts/Valentine/img6.png "/dev/ directory")

![](/images/posts/Valentine/img7.png "key")

![](/images/posts/Valentine/img8.png "Todo list")

The second file is an SSH key in hexadecimal format, but we don't know the users to loggin. Meanwhile we are looking for more clues, we can run **hydra** or other script to brute force the SSH user.

![](/images/posts/Valentine/img9.png "RSA key")

The secret clue in the web image, is the heart at the right that is the logo of Heartbleed. This vulnerability consist in sending a crafte packet used in the SSL protocol, then the server will respond with random information from the memory. Our goal is to caputre some sensitive information that allow us to enter in the machine.

![](/images/posts/Valentine/img10.png "Heartbleed explotation")

The python script creates the special packed and sends a lot of traffic, to increase the provability that apears sensitive information. After executing the script and looking for information a few times, we finally found a string in Base64 that contains the message *heartbleedbelievethehype*.

![](/images/posts/Valentine/img11.png "Memory leak")

![](/images/posts/Valentine/img12.png "Base64 string")

With a lot of patience and luck, finally we obtained the first shell using the SSH service. The username for the SSH RSA key is the substring *hype* from the Base64 string that we found using Heartbleed. To bruteforce different users it can be done with [*Paramiko*](http://www.paramiko.org/) library and all the possible usernames (also combination of these string) that we have found during the information gathering.

![](/images/posts/Valentine/img13.png "SSH connection")

> Note: These machine have different ways to elevate privileges, but probably the author patched the DirtyCow exploitation and now is only possible the path that he designed. This way of elevate privileges is more interesing than a simple exploit.

Once we have the shell and discardted the elevation with some OS exploit, it's time to enumerate some files to find another path for the privilege escalation. With **find** tool we find the *dev_sess* file in */.dev/* directory wich our user have read and write privileges. 

![](/images/posts/Valentine/img14.png "Search files")

Searching more information about this file, we find that is a inter-process communication socket (IPC). With **tmux** we can attach to this socket and finally we get root privieleges.

![](/images/posts/Valentine/img15.png "Socket file")

![](/images/posts/Valentine/img16.png "Tmux with root privileges")
