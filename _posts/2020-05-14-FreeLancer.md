---
layout: post
title: "HackTheBox FreeLancer"
date: 2020-05-14
description: "Writeup of FreeLancer HackTheBox CTF"
tag: CTF
---

Today instead of a HackTheBox machine, we are going to try a CTF. The main difference is that in the machines you face a larger environment with many possibilities and in the CTFs are usually more specific and specialized environments. For example, this CTF is focused on a web application.

![](/images/posts/FreeLancer/img1.png "Web application")

In the beginning we don't see anything strange in the application, so let's analyze the code to see if we find anything. We see that there are some HTML comments that contain the functionality **portfolio.php**.

![](/images/posts/FreeLancer/img2.png "Comments")

When we enter the new endpoint we don't find anything interesting, just a text and a default image.

![](/images/posts/FreeLancer/img3.png "Portfolio page")

We tried to inject some special characters in the ID field, but the server doesn't seem to be affected. To double check, we are going to use *sqlmap* to do a more extensive test to see if it's a blind SQL injection.

![](/images/posts/FreeLancer/img4.png "sqlmap")

It was indeed a parameter with boolean-based and time-based blind SQL injection. We are going to proceed to list the databases, to see if there is sensitive information that allows us to continue the challenge.

![](/images/posts/FreeLancer/img5.png "Extract databases")

Most probably database **freelancer** is the right one. Next we will list the tables and extract their information.

![](/images/posts/FreeLancer/img6.png "Extract tables")

![](/images/posts/FreeLancer/img7.png "Dump table")

The $2y$ field indicates that [*bcrypt*](https://en.wikipedia.org/wiki/Bcrypt) has been used (a function based on [Blowfish](https://en.wikipedia.org/wiki/Blowfish_(cipher)) encryption). To obtain the password we will use the classic *johntheripper* with the rockyou dictionary.

![](/images/posts/FreeLancer/img8.png "Bruteforce credentials")

While waiting for the pasword to be cracked, we will continue analyzing the web application. In this case we are going to launch *gobuster* to enumerate directories, since browsing the web page we haven't found many more.

![](/images/posts/FreeLancer/img9.png "Directory enumeration")

Bingo! We found a new directory that looks interesting. Testing the credentials we have found doesn't seem to be working, so let's keep listing inside the new directory.

![](/images/posts/FreeLancer/img10.png "Administrator login pannel")

While we are still listing let's use sqlmap again to see if these fields are vulnerable and we can log into the admin panel.

![](/images/posts/FreeLancer/img12.png "More enumeration")

![](/images/posts/FreeLancer/img11.png "sqlmap")

Not finding where to go from here, we will recapitulate and continue with the vulnerability we have found. With sqlmap option `--privileges` we will list the privileges of the database process.  

![](/images/posts/FreeLancer/img14.png "Database privileges")

With the privileges we have, we can exfiltrate files using SQL injection. To test if the application is in the `/var/www/html` directory, we are going to export the page index.

![](/images/posts/FreeLancer/img15.png "Extract index.php")

Next we will filter other files of the web application to see if we find sensitive information. Finally in **panel.php** we have found the CTF flag.

![](/images/posts/FreeLancer/img16.png "Extract panel.php")