---
layout: post
title: "Pwnlab:init"
date: 2019-02-28 
description: "Writeup of Pwnlab:init virtual machine"
tag: CTF
---

This time we are going to solve a very complete machine where we will touch different techniques to execute code and we will even have to do reverse engineering to escalate privileges.

### Information gathering

As usual we are going to start with *nmap* to find out what services are running on the machine. We find a web application, a database that is most likely from that application and portmapper (which is a service to map the ports of network services to Remote Procedure Calls).

![](/images/posts/Pwnlab_init/img1.png "nmap scan")

While analyzing the web application manually, we are going to launch a *gobuster* to enumerate hidden directories that we may not be able to reach from the interface

![](/images/posts/Pwnlab_init/img2.png "Directory enumeration")

![](/images/posts/Pwnlab_init/img3.png "Webpage")

Browsing a bit we find an interesting login panel, but if we look at the URL we can see that the application uses the **page** parameter to serve the current page, instead of using a different path.

![](/images/posts/Pwnlab_init/img4.png "Login pannel")

Trying the same parameter with the index, we see that it renders that page. So we can use PHP filters to convert the file to **Base64** and include it in the response instead of rendering the content.

![](/images/posts/Pwnlab_init/img5.png "LFI PoC")

If we decode the result, we see that it is the home page. We can also see the part of the vulnerable PHP code that allows Local File Inclusion: `include($_GET['page'].".php")`.

![](/images/posts/Pwnlab_init/img6.png "PHP file")

In the results of the *gobuster* we see that there is a **config.php** file that may have an interesting content. We use the same trick of converting the content to Base64 and when we decode it we see the database ceredentials.

![](/images/posts/Pwnlab_init/img7.png "LFI config")

Normally databases can only be accessed by internal services, but in this case *nmap* has found the public service. Let's connect to see if there is sensitive information in the **Users** database.

![](/images/posts/Pwnlab_init/img8.png "Database credentials")

We can see in the database the users table, which contains three users. The credentials appear to be only encoded in Base64, so we can extract the information and use them later.

![](/images/posts/Pwnlab_init/img9.png "Database users")

![](/images/posts/Pwnlab_init/img10.png "Plaintext credentials")

Now that we have valid credentials, we can access the file upload functionality.

![](/images/posts/Pwnlab_init/img11.png "Upload file")

Let's test if the functionality detects if the image contains any malicious payload. Uploading an image and in Burp adding the PHP `<?php system($_GET['cmd']);?>` code as reverse shell, we see that it has not responded with any problem.

![](/images/posts/Pwnlab_init/img12.png "Upload PHP webshell")

We can check that the image has been uploaded correctly in the **uploads** directory.

![](/images/posts/Pwnlab_init/img13.png "Uploaded image")

We have to see how to execute our shell. Fortunately, we can access the application code and it becomes a white-box text. Analyzing the **index.php** file, we see that there are some suspicious comments indicating that the multilingual functionallity is not implemented. This functionallity uses the `lang` cookie to include the language file.

![](/images/posts/Pwnlab_init/img14.png "Index vulnerability")

By linking this LFI without the PHP constraint with the shell we uploaded earlier, we can create a cookie that points to image `../upload/c0fb01577de1906b91a3ec125fb829bf.jpg` and use the cmd parameter to execute code on the server.

![](/images/posts/Pwnlab_init/img15.png "Webshell")

This time we are going to upload a *Python* reverse shell that allows us to interact more comfortably on the remote machine. We just need to open an HTTP server on our machine and download the reverse shell using *wget*.

![](/images/posts/Pwnlab_init/img16.png "Upload reverse shell")

We open the port on our machine so that the reverse shell can connect and voilà, we have permanent access to escalate privileges.

![](/images/posts/Pwnlab_init/img17.png "Execute reverse shell")

## First steps in the machine

We have the web application permissions, but we see that there are the same users as in the database and we can use their credentials.

![](/images/posts/Pwnlab_init/img18.png "Change user")

We will start the enumeration with the user kane. First of all we use find to search for files that have the Set User ID (**SUID**). This permission allows us to execute the files as if we were the owner. In this case file `/home/kane/msgmike` is owned by mike and we see that it is an ELF executable.

![](/images/posts/Pwnlab_init/img19.png "List files")

We transfer the file to our machine for further analysis. Reading a part of the file with head, we can see that it is a Linux executable file.

![](/images/posts/Pwnlab_init/img20.png "Exfiltrate file")

To analyze it we will use *radare2*. Just looking at the main function we can see in the final part where the string `cat /home/mike/msg.txt` is used to execute it with the *system* command. [System](https://linux.die.net/man/3/system) executes a command specified in command by calling `/bin/sh -c command`, and returns after the command has been completed. 

![](/images/posts/Pwnlab_init/img21.png "Reverse engineering")

By running the *cat* command without the full path, we can trick the system into using a fake *cat* binary. To do this we only have to create a new file and assign it execution permissions (in our case we are going to make a bash file that executes a shell). We modify the PATH so that our fake *cat* is the first to be used and when we execute `/home/kane/msgmike`, a new shell opens with the **mike** permissions.

![](/images/posts/Pwnlab_init/img22.png "New cat")

## Last steps in the machine

Following the enumeration with the new user, we see that in his directory there is a suspicious file similar to the previous one, but this time it is called msg2root. It is conveniently an ELF file owned by root with the **SUID** permission.

![](/images/posts/Pwnlab_init/img23.png "List files and exfiltrate")

We are going to do the same to transfer it to our machine and analyze it with *radare2*. This time we see that it first prints `Message for root` and waits for the user input. Then it uses the input to create a new string and execute with system.

![](/images/posts/Pwnlab_init/img24.png "Reverse engineering")

This time we see that it uses the complete path for *echo*, so we must find another way to execute code. Simply if we connect the bash command separators (like **||**, **&&** and **;**), we can  execute the code as you can see in the `id;id` PoC. To persistently gain privileges, we just need to modify the permissions of `/bin/bash` and add the **SUID** for our user. We can then run a new shell and gain administrator privileges.

![](/images/posts/Pwnlab_init/img25.png "Modify bash")