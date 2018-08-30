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

For this case, first we will read the application code to understand how it works and when uses the native library.

![](/images/posts/UnCrackable2/img3.png "TBC")

![](/images/posts/UnCrackable2/img4.png "TBC")

![](/images/posts/UnCrackable2/img5.png "TBC")

### Hooking with Frida

![](/images/posts/UnCrackable2/img6.png "TBC")

![](/images/posts/UnCrackable2/img7.png "TBC")

![](/images/posts/UnCrackable2/img8.png "TBC")

![](/images/posts/UnCrackable2/img9.png "TBC")

![](/images/posts/UnCrackable2/img10.png "TBC")

![](/images/posts/UnCrackable2/img11.png "TBC")

![](/images/posts/UnCrackable2/img12.png "TBC")

![](/images/posts/UnCrackable2/img13.png "TBC")

### Bonus clip 1

![](/images/posts/UnCrackable2/img14.png "TBC")

![](/images/posts/UnCrackable2/img15.png "TBC")

![](/images/posts/UnCrackable2/img16.png "TBC")

![](/images/posts/UnCrackable2/img17.png "TBC")

![](/images/posts/UnCrackable2/img18.png "TBC")

![](/images/posts/UnCrackable2/img19.png "TBC")

![](/images/posts/UnCrackable2/img20.png "TBC")

![](/images/posts/UnCrackable2/img21.png "TBC")

![](/images/posts/UnCrackable2/img22.png "TBC")

![](/images/posts/UnCrackable2/img23.png "TBC")

![](/images/posts/UnCrackable2/img24.png "TBC")
