---
layout: post
title: "UnCrackable1"
date: 2018-04-30 
description: "Writeup of OWASP UnCrackable 1"
tag: CTF
---   

TODO: intro

### Information gathering

Starting as allways, the first thing we will do is recollect information about the application. We will use **apktool** to see the different files that the application contains and **dex2jar** to generate a jar file. The *Android Manifest* usually contains valuous information (the permissions used, configuration errors, the application components, ...), but in this case, since the application is so small, it does not contain relevant information.

![](/images/posts/UnCrackable1/img1.png "TBC")

![](/images/posts/UnCrackable1/img2.png "TBC")

It is also important to see how the application works and see if it has control mechanisms that do not allow it to run in the emulator. When the application starts showns a message of "Rooted detected!" and it closes immediately.

![](/images/posts/UnCrackable1/img3.png "TBC")



![](/images/posts/UnCrackable1/img4.png "TBC")

![](/images/posts/UnCrackable1/img5.png "TBC")

![](/images/posts/UnCrackable1/img6.png "TBC")

![](/images/posts/UnCrackable1/img7.png "TBC")

![](/images/posts/UnCrackable1/img8.png "TBC")

![](/images/posts/UnCrackable1/img9.png "TBC")

![](/images/posts/UnCrackable1/img10.png "TBC")

![](/images/posts/UnCrackable1/img11.png "TBC")

![](/images/posts/UnCrackable1/img12.png "TBC")

![](/images/posts/UnCrackable1/img13.png "TBC")

![](/images/posts/UnCrackable1/img14.png "TBC")

![](/images/posts/UnCrackable1/img15.png "TBC")

![](/images/posts/UnCrackable1/img16.png "TBC")

![](/images/posts/UnCrackable1/img17.png "TBC")

![](/images/posts/UnCrackable1/img18.png "TBC")

![](/images/posts/UnCrackable1/img19.png "TBC")
