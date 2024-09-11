---
layout: post
title: "IdaCrackme 1"
date: 2018-01-16
description: "Writeup of OWASP IdaCrackme 1"
tag: Reversing
---

The first post of reversing, will be a challenge that I have found in a course to learn reversing with IDA. It is a typical crackme where a pop-up appears asking for some credentials that we must know.

This challenge could be applied in real life, for example in a video game or software that asks for a license key. In the <del>morally questionable</del> case that we want to use it without paying for the key, we could use some technique to bypass the check.

![](/images/posts/IdaCrackme1/img1.png "Crackme pop-up")

Testing some random credentials, we can see that the application responds with the following error message: *"No luck there, mate!”*.

![](/images/posts/IdaCrackme1/img2.png "Wrong credentials error")

### First steps

The first clue we have to analyze the application is the message that appears in the previous error. With IDA we can analyze the strings that appear in the executable and search for the message *No luck there, mate!*. As we can see, apart from the string that we have seen, there are also other strings for a correct execution.

![](/images/posts/IdaCrackme1/img3.png "Binary strings")

Looking at the cross references (*xrefs*) of the error message, we can see that two functions (**sub_401362** and **sub_40137E**) that use this string. Searching the functions we have found in the application diagram we see that there is a section where it's decided whether to display the error message or the correct one.

![](/images/posts/IdaCrackme1/img4.png "Error string xrefs")

![](/images/posts/IdaCrackme1/img5.png "Application diagram")

### Analyzing the application

Analyzing in more depth that section we see that the function **sub_401362** (that we have seen previously with the *xrefs*) is executed when the check fails and the function **sub_40134D** is executed when it has been successful. Previously we see that there are two functions that their results could determinate the final message (**sub_40137E** and **sub_4013D8**).

![](/images/posts/IdaCrackme1/img6.png "Application code")

Let's start looking at the functions in order of execution to see if we can know at what point the credentials are checked. The first function will be **sub_40137E** (renamed to **check_1**) where we see a loop in which one of the conditions shows the error message. Briefly analyzing the loop, we see that a string is being iterated and checked to be between two values (possibly between A and Z). 

Following the execution flow once the loop is finished, we see another function **sub_4013C2** where it may be checking or modifying the credentials.

![](/images/posts/IdaCrackme1/img7.png "check_1 function")

The **sub_4013C2** (**check_2**) function is another loop that iterates over the register *EBX* and adds the content of *EDI*.

![](/images/posts/IdaCrackme1/img8.png "check_2 function")

The next function we are going to analyze seems to be further modifying another variable (either the stored credential or the one we have provided). It is another loop where certain mathematical operations are applied to the *EDI* register and once out of the loop is performed an **XOR** operation with **1234**.

![](/images/posts/IdaCrackme1/img9.png "check_3 function")

Returning again to the application diagram, we can now understand more about the **check_1** and **check_2** functions and how the results of these functions are subsequently compared in the *EAX* and *EBX* registers respectively.

![](/images/posts/IdaCrackme1/img10.png "Code renamed")

### Pathcing the application

Different techniques can be used to solve this challenge, from reproducing the algorithm used to process the credentials to modifying the execution flow. In this case, and to follow the [KISS principle](https://en.wikipedia.org/wiki/KISS_principle), we are going to modify the executable so that it always addresses the correct message. It's only necessary to change the final condition from *JZ* to *JNZ* and make any incorrect credentials be treated as correct.

![](/images/posts/IdaCrackme1/img11.png "Modify the executable")

![](/images/posts/IdaCrackme1/img13.png "Correct execution")