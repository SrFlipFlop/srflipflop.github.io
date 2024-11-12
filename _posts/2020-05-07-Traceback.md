---
layout: post
title: "HackTheBox Traceback"
date: 2020-05-07 
description: "Writeup of Traceback HackTheBox machine"
tag: CTF
---

Spoiler: this HTB machine has a very curious scenario where we will find that another attacker has already gained access. We will have to use the artifacts we find to gain access and escalate privileges using some of the attacker's configuration flaws.

### Information gathering

As always, we will start enumerating the remote services using *nmap*. We could se that there are only two services: SSH server, <del>normally used to open a persistent session once we gain initial access</del>, and a web application.

![](/images/posts/Traceback/img1.png "nmap scan")

We start by taking a look at the web application and when we see nothing we use the browser to view the HTML code. The attacker has left a hidden message in a comment that allows us to follow the rabbit.

![](/images/posts/Traceback/img2.png "Web defacement")

Searching for a web shells repository on GitHub, we found [TheBinitGhimire/Web-Shells](https://github.com/TheBinitGhimire/Web-Shells) and using some bash magic we created a dictionary with all the files. Once we have the dictionary, we can search the directories using *gobuster* to see if the shell used by the attacker appears.

![](/images/posts/Traceback/img3.png "Directory enumeration")

We find a login panel and looking for the credentials for **smevk.php** in the same repository we see that the default credentials are: `admin:admin`.

![](/images/posts/Traceback/img4.png "Web shell login page")

### Gaining foothold

We have gained access to a powerful web shell that allows us to continue enumerating the machine with **webadmin** permissions.

![](/images/posts/Traceback/img5.png "Web shell pannel")

To achieve some persistence, in case the attacker decides to delete his trace or the administrators delete the web shell, we will add our public key in `/home/webadmin/.ssh/authorized_keys` to authenticate via SSH.

![](/images/posts/Traceback/img6.png "SSH authorized keys")

### Analyzing the remote machine

Once connected via SSH as **webadmin**, we start to investigate and see a note from the user **sysadmin** saying that he has left a tool to practice with Lua.

![](/images/posts/Traceback/img7.png "SSH connection")

Looking for that tool of the note, we see that when listing the *sudo* options we find an executable called *luvit*. This tool allows us to run Lua scripts with **sysadmin** permissions.

![](/images/posts/Traceback/img8.png "Sudo listing")

We can execute commands directly with `sudo -u sysadmin /home/sysadmin/luvit -e os.execute("/bin/bash")`, but we are going to do the same operation as before modifying the **authorized_keys** file to access more comfortably by SSH. The following Lua script adds our public key to the **sysadmin** user. 

![](/images/posts/Traceback/img9.png "Lua script")

### Final privilege escalation

Now with a new user, we have to start the enumeration again in case we see another privilege escalation vector for **root**.

![](/images/posts/Traceback/img10.png "SSH connection")

After a long time searching for a way to escalate privileges, we see that there are some files owned by root, which we have permissions to modify them as we are in the same group. Listing the content we see that this is the message that is displayed when there is a new SSH connection.

![](/images/posts/Traceback/img12.png "List directory")

We only need to modify the file to execute commands with elevated privileges. To make the example we are going to create a bash script that is going to be executed when we enter the machine.

![](/images/posts/Traceback/img13.png "Modify header")

Bingo! In the same header we see the result of the script we have created. To gain persistent privileges, we could use other techniques such as modifying the sudoers file or playing with the SSH keys again.

![](/images/posts/Traceback/img14.png "New SSH connection")