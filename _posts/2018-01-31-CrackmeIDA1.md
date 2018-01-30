---
layout: post
title: "Introduction to reversing with IDA: crackme 1"
date: 2018-01-15 
description: "Writeup of the first crackme with IDA Pro"
tag: Reversing
---

I was searching some blogs or tutorials to practice reversing with IDA and remember some knowledges of assembler (especially x32 / x64 and ARM). A colleague of work recommended me to start with the new Ricardo Narvaja tutorials “Introducción al reversing con IDA Pro desde cero”. The tutorials contain different crackmes to practice the learned knowledges that I will try to solve this problems and write the solving process.

This crackme can be found at the first chapter. To solve it, first I will do a static analysis to understand how it works and then patch the code.

### Hunting the first clue

This crackme can be found at the first chapter. To solve it, first I will do a static analysis to understand how it works and then patch the code.

![](/images/posts/IdaCrackme1/img1.png "Trying some credentials")

![](/images/posts/IdaCrackme1/img2.png "Wrong credentials error message")


To start the reversing process, it’s important to find something to start searching (the spanish expression *“tirar del hilo”*). In this case the starting point will be the string displayed in the error “No luck there, mate!”.  We can use **string window** to search where the strings are referenced in the code, in this case the string “No luck there, mate!” is referenced in two different blocks.

![](/images/posts/IdaCrackme1/img3.png "Strings window")

![](/images/posts/IdaCrackme1/img4.png "Searching where are used the strings")


### Searching the important code

Analyzing the different functions and using the strings as a reference, finally it can be assumed that the following function is the responsible of the main logic of the crackme (especially the code to operate with the different views). With color red and green, I marked the wrong and good options to be easier the reverse engineering process.

![](/images/posts/IdaCrackme1/img5.png "Main function")


Using (~~another~~) the spanish expression *"Quitar el grano de la paja"* that means to separate the important things from the rest. This part of the reversing is used to search the most important part of the code to reversit later. The most important part of the code is the previous one that contain the searched strings (in red and green).  Concretely the functions **sub\_40137E** and **sub\_4013D8** have to be reversed to understand its functionality. Probably the results of these functions are used in the final comparison of EAX and EBX.

![](/images/posts/IdaCrackme1/img6.png "Code blocks that print the correct and wrong messages")


### Reversing the principal functions

This first check function (aka **check\_1**) can be explained in two different parts.
+ The left part or the inner part of the loop, contains two different conditions using the hexadecimal values **41** and **5A**. The first condition ends in the error “No luck mate” and the second condition ends in a modification of the iterated data. These two values in hexadecimal in the ASCII table are the A and Z. With this information we can suppose that the loop is used to check if the iterated data is an uppercase letter. In the case that is an lower case character or a number, the function change the value to an upper case character.
+ The right part uses the checked array of characters as an input of the function **sub\_4013C2** (**check\_2**). Then compare the EDI register (provably modified in the **check\_2** function) with the hexadecimal value **5678**, and move the boolean result to the EAX register.

![](/images/posts/IdaCrackme1/img7.png "First check function")

```python
def check_1(string):
	for i, c in enumerate(string):
		if c < 0x41:
			error()
			break
		if c > 0x5A:
			string[i] = hex(c) - 0x20
	d = check_2(string)
	a = xor(d, 0x5678)
	return a
```

The next function to analyze (**check\_2**) uses as input the checked string in the previous function (**check\_1**). The function iterates the array of characters to sum all hexadecimal values and store the result in the EDI register. This value is compared in the **check\_1** with the hexadecimal value **5678**.

![](/images/posts/IdaCrackme1/img8.png "Second check function")

```python
def check_2(string):
	d = 0x0
	for c in string:
		d += c
	return d
```

The last function to analyze is the **check\_3**. As we can see in the assembler code, the function iterates an array (probably the characters array) and performs mathematical operations to generate a second number that then compares with the hexadecimal value **1234**. The mathematical modification for each element is the following *EDI = (EDI * 0x0A) + (Character – 0x30)*.

![](/images/posts/IdaCrackme1/img9.png "Third check function")

```python
def check_3(string):
	a = b = d = 0
	for c in string:
		a = 0x0A
		b = c - 0x30
		d = (d * a) + b
	b = xor(d, 0x1234)
	return b
```

Finally the results of the checks in the **check\_1** and **check\_3** are compared and only if they are equal the program prints the correct answer. Summarizing the problem, an input string is modified using different algorithms and then is compared each result with the hexadecimal values **1234** and **5678**, if both checks are correct then the program prints the string **“Great work, mate! Now try the next CrackMe!”**.

![](/images/posts/IdaCrackme1/img10.png "Analyzed functions")


### Patching the binary

To bypass this problem it’s only necessary to bypass the last check of EAX and EBX. It can be done by change the jump equal (**JZ**) for jump not equal (**JNZ**) and then a wrong password will be correct.

![](/images/posts/IdaCrackme1/img11.png "Patch first function")

This solution can fail if is used a input name that have numbers or characters (remember that the **check\_1** function examine if the string contains uppercase letters). It is possible to bypass this control by using the **NOP** instruction and don’t let the execution arrive in the wrong code. Remember that this patch needs two **NOP** instructions to fill the **JNZ** instruction.

![](/images/posts/IdaCrackme1/img12.png "Patch second function")

![](/images/posts/IdaCrackme1/img13.png "Done ")