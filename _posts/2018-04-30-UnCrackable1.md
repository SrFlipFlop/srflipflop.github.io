---
layout: post
title: "UnCrackable1"
date: 2018-04-30 
description: "Writeup of OWASP UnCrackable 1"
tag: Reversing
---   

OWASP has created three challenges to practice reversing and evasion techniques for security controls. Each challenge adds a bit of difficulty in the security mechanisms and in reverse engineering techniques. In this first challenge we will have to bypass a root verification and some reversing to extract the secret.

### Information gathering

Starting as allways, the first thing we will do is recollect information about the application. We will use **apktool** to see the different files that the application contains and **dex2jar** to generate a jar file. The *Android Manifest* usually contains valuous information (the permissions used, configuration errors, the application components, ...), but in this case, since the application is so small, it does not contain relevant information.

![](/images/posts/UnCrackable1/img1.png "Extracting files from the APK")

![](/images/posts/UnCrackable1/img2.png "Android manifest")

It is also important to see how the application works and see if it has control mechanisms that do not allow it to run in the emulator. When the application starts showns a message of "Rooted detected!" and it closes immediately.

![](/images/posts/UnCrackable1/img3.png "Root error")

### Reading a bit of code and hooking

With a quick look of the main class of the application, we can see that in the *onCreate* method three functions seems to check if the device is rooted and then call the error that close the application.

![](/images/posts/UnCrackable1/img4.png "MainActivity")

These methods search for common root information (like files or other applications) and returns a boolean response. If some of these methdos return true, the application shows the root error message and it closes automaticaly.

![](/images/posts/UnCrackable1/img5.png "Root checks")

The first idea to do the bypass of the root controls is to hook the methods that checks if the device is compromised and return always *false*. But I don't know why these Frida hooks doesn't send any information when they are attached to early methods.

![](/images/posts/UnCrackable1/img6.png "Frida hooks for root controls")

We lost a battle but not the war! Doing some code research we can find that the error function configure the click handler using *uncrackable1.b*. This class uses *System.exit(0)* when the click is performed. We can't evade the root checks but we can hook the *onClick* function and don't perform the exit. 

![](/images/posts/UnCrackable1/img7.png "uncrackable1.b class")

![](/images/posts/UnCrackable1/img8.png "Frida hook for onClick")

Hurrah! it worked. The pop up error is still showing but when we accept the message the application doesn't close. Now we can continue with the challange and try to submnit a random text to see what the application does. We can see that the application shows another error message saying that the secret string it's not ok.

![](/images/posts/UnCrackable1/img9.png "Root error bypass")

![](/images/posts/UnCrackable1/img10.png "Incorrect string")

With the first look on the code we can imagine which function is doing the secret key check, but just to verify that is correct, we can see in the layout configuration file the function that is called when you click on the button.

![](/images/posts/UnCrackable1/img11.png "Layout configuration file")

The method *verify* gets the string and uses the method *a* from *a* class. With a quick look to this method we can se that is using two suspicious strings in another *a* method and finally compares the it's result with our string. It seems that one of these strings is an encryption key and the other the encrypted message.

![](/images/posts/UnCrackable1/img12.png "Application code that verifies the string")

Looking at the second *a* method we finally see that is using AES to decrypt the the second parameter. 

![](/images/posts/UnCrackable1/img13.png "Decrypt method")

Ok it's time to check if our hypothesis is correct. To do this we will use Frida to hook the analysed functions and execute the application.

![](/images/posts/UnCrackable1/img14.png "Frida hooks")

We can see that the result of *a* (the one who check the strings) return false and then the error message pops up. But what will happen if we change the function and always return true?

![](/images/posts/UnCrackable1/img15.png "Incorrect string eror")

![](/images/posts/UnCrackable1/img16.png "Modify function return")

Finally we can see the application success message, but there is still one last thing missing and it is to discover the application's ciphered message.

![](/images/posts/UnCrackable1/img17.png "Bypass check method")

### Bonus clip

The easyest way to obtain the secret is to hook the decrypt function and get the return value. Once we have the value in array of bytes we need to convertit to string format, we can do this by creating a new *String* Java class and pass to the constructor the byte array. The secret string that we are looking for is **I want to believe**.

![](/images/posts/UnCrackable1/img18.png "Decrypt function hook")

![](/images/posts/UnCrackable1/img19.png "Final execution with the secret string")
