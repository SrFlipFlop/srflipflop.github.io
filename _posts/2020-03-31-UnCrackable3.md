---
layout: post
title: "UnCrackable3"
date: 2020-03-31 
description: "Writeup of OWASP UnCrackable 3"
tag: Reversing
---   

Finally we are going to face the last challenge of Uncrackable available until now. This one is going to be much more complicated since we will have to play more with the native libraries and the Java Native Interface.

### UnCrackable3

When we open the application, we see that the first error appears indicating that it has detected that the environment is compromised.

![](/images/posts/UnCrackable3/img1.png "Protection mechanisms detection")

If we look at the application entry point, we can see that the dialog string is inside an if with different root checks from `RootDetection` class.

![](/images/posts/UnCrackable3/img2.png "MainActivity code")

In the `RootDetection` class we can see three methods for detecting rooted devices:
1) **Check for sensitive binaries**: Search for the `su` binary in the different path locations
2) **Check for emulation environments**: Normally the OEM install of Android use the release-keys, but for emulators it use the string "test-keys"
3) **Check for specific files**: Searches for certain files that can be found in rooted environments such as some executables or applications used to change permissions

![](/images/posts/UnCrackable3/img3.png "Root detection")

It seems that the solution to bypass these controls is easy, so we are going to use Frida to modify the responses of these functions and make the application believe that it is not in a rooted environment.

![](/images/posts/UnCrackable3/img4.png "Frida root detection bypass")

<del>In the end, not everything is as easy as it seems.</del> We can see that when we run the application with Frida, the `RootDetection` functions have been intercepted, but an error occurs and closes the application.

![](/images/posts/UnCrackable3/img5.png "Application error")

Let's analyze the application logs, to detect why the application is crashing. Just before the crash, indicated by the debugg string “beginning of crash”, we see that a verbose log appears with the information that a tampering mechanism has been detected (*UncraCkable3: Tampering detected! Terminating ...*).

![](/images/posts/UnCrackable3/img6.png "Application logs")

Searching for that same string in *jadx* we found no reference to it in the code. We have to open the application with *apktool* and cross our fingers that this string is not encrypted. Bingo! With *grep* we find the string inside the **libfoo.so** library (in the different architectures) and we also see some strings that give us clues that the library is looking for.

![](/images/posts/UnCrackable3/img7.png "Detecting error")

To investigate further, let's go down a step and use *Ghidra* to look up the string and see if we can get to the method that executes the function.

![](/images/posts/UnCrackable3/img8.png "Search string")

Following the string references, we arrive at the native function `_android_log_print`, which we can tell from its name that it is in charge of printing in the logs.

![](/images/posts/UnCrackable3/img9.png "Native print function")

We return to follow the reference of this function and we arrive at the undefined function `FUN_00013080`.

![](/images/posts/UnCrackable3/img10.png "Function")

The first thing we are going to do before decompiling and analyzing the function is to change its name. We see some interesting functions and strings that can tell us what it is looking for. The function searches the **virtual memory** of the process to see if there is a hooking tool. To do this, it reads the file `/proc/self/maps` and searches for the substrings **frida** or **xposed** with *strstr*.

![](/images/posts/UnCrackable3/img11-2.png "Renamed function")

Once we know what the application is using to detect Frida, we could simply compile another Frida version by modifying the name so that it is not reflected in virtual memory, but it is more fun to use the same tool and modify its behavior. For this we are going to intercept and attach to the *strstr* function in *libc.so*, in the case that the function parameter is frida we are going to modify the return value of the function.

![](/images/posts/UnCrackable3/img12.png "Bypass tampering script")

Since we have opened Ghidra let's see if it has detected other native functions. We find that in the `MainActivity` there is the `baz` and `init` function, and in `CodeCheck` we find the `bar` function.

![](/images/posts/UnCrackable3/img13.png "Library functions")

Returning to the `MainActivity`, we find the definition of the two native functions and how the library is loaded. We can also see a very curious function that does not affect us, but it is very interesting to understand this protection mechanism. Summarizing, `verifyLibs` checks the checksum of different libraries and the DEX file, to compare it with some static strings that it has in the resources. This mechanism allows to verify the integrity of these resources and detect modifications.

![](/images/posts/UnCrackable3/img14.png "MainActivity native functions")

Following the native functions, we see that `onCreate` executes the function just discussed and then `init` with an XOR key.

![](/images/posts/UnCrackable3/img16.png "MainActivity onCreate")

The next step is to analyze what the init function does, as we can see in Ghidra the decompiled code is not very clear, so we are going to interpret it as if it was pseudo-code.

![](/images/posts/UnCrackable3/img15.png "Native Init function")

![](/images/posts/UnCrackable3/img17-1.png "Native Init function decompiled")

Understanding the parameters that a native function receives from the Java Native Interface JNIEnv and jobject, we can see how the `FUN_00013250` function is executed and then it is simply copying the 18-bit XOR key into a section of memory.

![](/images/posts/UnCrackable3/img17-2.png "Init pseudocode")

In this function, seeing that system calls *fork*, *getpid* and *ptrace* are used, we understand that is a mechanism so that Frida cannot be injected correctly in the application process. In our case we do not need to modify this control mechanism, since the Frida agent is injected before the function is called.

![](/images/posts/UnCrackable3/img18.png "Native unknown function")

Seeing that `init` only copies the XOR key, let's go back and see what else is in the `MainActivity`. The `verify` function is in charge of checking the user's secret. This check is done with the `CodeCheck` class, which at the same time uses the native function `bar`. The result of this native function is the one that decides if the secret is correct or not.

![](/images/posts/UnCrackable3/img19.png "MainActivity code")

Decompiling the function we can see that there is an inner loop where an XOR operation is being performed. Most likely the key that we have initialized previously will be used to decrypt a secret text inside the application.

![](/images/posts/UnCrackable3/img20-1.png "Native bar function")

In the following pseudo-code we can see that the application first checks the length of the key provided by the user, in case it is correct it starts decrypting the secret and compares each value with the user input.

![](/images/posts/UnCrackable3/img20-2.png "Python script")

In the native code there is a moment where the result of doing the XOR operation is stored in a register. That is a good point to attach with Frida and read the value of the register before the comparison.

![](/images/posts/UnCrackable3/img21.png "Result sotred in the register")

To modify the library we first have to modify the `loadLibrary` function to verify that the library is the one we want to modify. Once in the correct library, we will enumerate the exported functions and when it coincides with the function `Java_sg_vantagepoint_uncrackable3_CodeCheck_bar` we will inject in the desired memory location.

![](/images/posts/UnCrackable3/img22.png "Frida script")

As we can see when executing the Frida script certain libraries appear and only when it matches the function the code is injected. Then we will see the different controls that we have bypassed and the first letter of the secret.

![](/images/posts/UnCrackable3/img23.png "Partial password")

Finally we obtain the secret: **making owasp great again**.

![](/images/posts/UnCrackable3/img24.png "Final password")
