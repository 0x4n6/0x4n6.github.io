---
published: true
title: BND Reversing Challenge Level 1 write-up
layout: post
tags: [Reversing, .NET]
categories: [CTF]
---
## Intro

Browsing the web, I stumbled upon the website of the intelligence agency of Germany (BND) where they [announced a reversing challenge](https://www.bnd.bund.de/DE/Karriere/Reversing_Challenge/Reversing_Challenge_node.html) in order to attract and recruit new potential employees. I was curious what the bright minds at BND had developed, so I had a look at the challenge and started poking at level 1. Disclaimer: I am already happily employed and not a German citizen.

The folder of the first challenge contains two files; one executable `evil.exe` and one PNG image which seems encrypted. Checking out the executable with [PE Studio](https://www.winitor.com) indicates this file is actually a .NET application. That's great, because disassembling .NET isn't the hardest thing to do. But first, let's execute the application in a VM to check out what it's trying to do or say. We're greeted with a message saying that our data is encrypted, and that we need to enter a certain code in order to decrypt it.

![Evil](https://raw.githubusercontent.com/0x4n6/0x4n6.github.io/master/_posts/images/bnd/evil1.PNG)

## Reversing

I used [ILSpy](http://www.ilspy.net) to disassemble the .NET application and found some interesting functions. The first one is called `installLinux()` and basically takes a hardcoded ascii string, replaces a lot of the characters and returns it. The second is called `callCthulhu()` and starts a TCP connection to `127.0.0.1` (localhost) at port `1337`. It then reads the first 1024 incoming bytes and compares it with the output of the `installLinux()` function. The third function `invertGravity()` seems the one where the crypto magic is happening. (I figure if you write a python script with the decoding routine, you'll get the decrypted file aswell. I chose for the fun route!) Looking at the last function `button1_Click()` reveals the execution flow of the application. It reads the inserted code and calls the `callCthulhu()` function. If `callCthulhu()` returns true, the PNG image will be decrypted.

I tried to manually reverse the given string `"The iron bank will have its due"` with all the replace statements from the `installLinux()` function, which failed. Then I realised I could just execute the function, because it returns the correct string we need. So, there it was;

>A Lannister always pays his debts 

Nice reference to Game of Thrones, BND.

![Decryption code](https://raw.githubusercontent.com/0x4n6/0x4n6.github.io/master/_posts/images/bnd/evil2.PNG)

The app compares the entered code through a TCP connection at the localhost according to the `callCthulhu()` function. To fix this, I opened a netcat listener on port 1337 and piped the code to it. 

![Decryption code](https://raw.githubusercontent.com/0x4n6/0x4n6.github.io/master/images/bnd/evil3.PNG)

Entering the code into the decryption window reveals another window for selecting the encrypted file.

![Select encrypted image](https://raw.githubusercontent.com/0x4n6/0x4n6.github.io/master/_posts/images/bnd/evil4.PNG)

There we go.

![Decryption](https://raw.githubusercontent.com/0x4n6/0x4n6.github.io/master/_posts/images/bnd/evil5.PNG)

## Loot

Resulting in a glorious MS paint version of the eiffeltower!

![Eiffeltower](https://raw.githubusercontent.com/0x4n6/0x4n6.github.io/master/_posts/images/bnd/evil6.PNG)
