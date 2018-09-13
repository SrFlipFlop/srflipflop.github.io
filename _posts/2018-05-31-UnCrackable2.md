---
layout: post
title: "UnCrackable2"
date: 2018-05-31 
description: "Writeup of OWASP UnCrackable 2"
tag: Reversing
---   

The second challange of OWASP Uncrackable adds a new layer of security (NDK) to make more difficult bypass the root controls and to extract the secret password. 

### Information gathering

Like the other challange, first we need to open the APK with **apktool** and generate a *jar* file.

![](/images/posts/UnCrackable2/img1.png "Extracting files from the APK")

This second APK contains more Java code and also have a suspicious native library that provably will have the code verification and some security controls.

![](/images/posts/UnCrackable2/img2.png "Application files")

### Static analysis

In this this challenge we will read the application code first to understand how it works and when uses the native library and then create the Frida hooks to bypass the security mechanisms.

The first function that we will analize is *onCreate* from the main activity. This method is the first executed according to [Android activity lifecycle](https://developer.android.com/reference/android/app/Activity#activity-lifecycle) and normaly it calldd the security methods.

At the beginning of the *onCreate* function it is checked if the device is rooted, in the case that is true the *a* function is called. It is also checked if the application is being debugged.

![](/images/posts/UnCrackable2/img3.png "onCreate function")

The *a* method (that aperars on the *onCreate*) creates the error message using the the string parameter and then call *System.exit* once the user click the OK button.

![](/images/posts/UnCrackable2/img4.png "a method")

The *verify* function is in charge of calling the *a* method from *CodeCheck* class and create the dialog that will show the verification message.

![](/images/posts/UnCrackable2/img5.png "verify function")

*CodeCheck* class is an interface for the native library that calls the native method *bar* to check the user string.

![](/images/posts/UnCrackable2/img6.png "CodeCheck class")

### Hooking with Frida

Now that we know how the application works, it's time to get your hands dirty and start using **Frida** to bypass the root control.

![](/images/posts/UnCrackable2/img7.png "Root detected message")

We saw in the static code analysis that the application use *System.exit* to kill the application when this detects that the device is rooted. With the following hook the *exit* function is overwritten and it will only send a log message when is called.

![](/images/posts/UnCrackable2/img8.png "Frida JavaScript hook")

![](/images/posts/UnCrackable2/img9.png "Application in a rooted device")

We also know that the *verify* method and the *CodeCheck* class are the responsible to send the string to the native code and receive the check response. With **Frida** we can also follow the execution flow when a user whant to verify the password.

![](/images/posts/UnCrackable2/img10.png "Execution flow")

With the previous hooks we can see the execution flow, but what happens if we modify the Java function and return always true?

![](/images/posts/UnCrackable2/img11.png "Frida hooks")

![](/images/posts/UnCrackable2/img12.png "Application always return true")

### Bonus clip 1: hooking native functions

We just hook the Java functions to bypass the secret check, but with **Frida** it is also possible to hook the functions from the native library.

According to the Android NDK documentation ([Android NDK example](https://developer.android.com/ndk/samples/sample_hellojni)), the native functions names have to match with a patter and the Java method name. Using *strings* we can list the native functions names and some other methods used.

![](/images/posts/UnCrackable2/img13.png "library strings")

With **Frida** we can search for a native method and attach a hook that can interact before and after the function is called. With this hook we can modify, like in the Java code, the return value from the native function.

![](/images/posts/UnCrackable2/img14.png "Frida native hook")

![](/images/posts/UnCrackable2/img15.png "Bypass hooking the native function")

### Bonus clip 2: extract the secret dynamically

At this point we still don't know the correct secret that will surely be at the native library protected with some cryptographic algorithm. The following graph is the disassembly of the verification function.

![](/images/posts/UnCrackable2/img16.png "bar function graph")

The first block we can see that it does some kind of cryptographic protection using **xor** function. Then it checks if the resulting string have 23 characters (17 in hexadecimal).

![](/images/posts/UnCrackable2/img17.png "Cryptographic block")

Once the string is in clear text, the native function uses *strncmp* to check if the strings are equal. Then puts 0 or 1 in the EAX register to return the true or false.

![](/images/posts/UnCrackable2/img18.png "Comparison block")

Knowing that is used *strncmp* to check if the strings are equal, we can hook this function using **Frida** and read the incoming parameters. First we need to enumerate the *libfoo* imports to know the *strncmp* address. Then we can attach a hook to the address and read the parameters (to only print our string we can specify that the string length is 23).

![](/images/posts/UnCrackable2/img19.png "Frida hooks for strncmp")

Now we can see at the terminal the first time using a dummy string the comparison is false, but we discover that the secret is *thanks for all the fish*. The second time using the correct string all the checks return the correct answer.

![](/images/posts/UnCrackable2/img20.png "Correct string discover")

### Bonus clip 3: extract the secret statically

Another way to extract the string is reverse engineering the cryptographic block and simulate the same behavior with another language. First we need to find the key and the ciphertext in the library.

![](/images/posts/UnCrackable2/img21.png "Key and ciphertext")

Once we have the two values, we can use python to replicate the same algorithm and extract the information.

![](/images/posts/UnCrackable2/img22.png "Python script")