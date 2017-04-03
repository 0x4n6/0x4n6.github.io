---
title: Securityfest CTF 2016 Rev100 [Flux] write-up
tags: [CTF, Reversing, Python]
categories: [CTF]
---
## Intro
Last week I had a couple of hours to kill and noticed there was a [CTF going on](http://securityfest.ctf.rocks). I decided to focus myself on the reversing challenges and this led to solving 'Flux', which was worth 100 points.
 
>Flux - Rev (100) This license tool came with some software we recovered from a battle. Please figure it out for us.
 
## Reversing
 
After unpacking the tarball I noticed the challenge file had a `.pyc` extension and after a quick verify with `file` I was rather sure. It's compiled Python! yay

>$ file flux.pyc  
flux.pyc: python 2.7 byte-compiled


Let's check `strings` for a quick look into the program.

>$ strings flux.pyc  
mCWc  
Enter license: s  
Wrong license!i  
Correct license!N(  
flagt  
raw_inputt  
licenset  
lent  
exitt  
ranget  
ord(  
wat.pyt  
><module>


Running the program will only reveal a prompt which asks for a license. And it's always wrong.

>$ python flux.pyc  
Enter license:  AAAAAAAAAAAAAAAAAAAAA  
>Wrong license!


It was a while ago I dealt with Python bytecode, but after a quick google search I stumbled upon `uncompyle6`. A tool to decompile the Python bytecode and reveal the original source code.

>$ uncompyle6 flux.pyc  

```python
Python 2.7 (decompiled from Python 2.7)  
Embedded file name: wat.py  
Compiled at: 2016-05-23 16:54:01

flag = [67, 78, 70, 70, 127, 117, 95, 115, 64, 70, 100, 120, 83, 100, 96, 80, 99, 65, 38, 112, 39, 104]
print 'Enter license: ',
license = raw_input()
if len(license) != len(flag):
    print 'Wrong license!'
    exit(0)
for x in range(len(flag)):
    if flag[x] != ord(license[x]) ^ x:
        print 'Wrong license!'
        exit(0)
print 'Correct license!'
```

Let's figure out what this program wants. The flag list is filled with numbers which represent ascii characters in decimal notation. It then prompts for a license and checks if it has the same length as the flag and then exits. 
In the `for` loop is where the magic happens. It loops from 0-21 (the length of the flag list) and checks if every value of the flag and license is the same if XOR'd against the loop variable. 


## Script it up

I then wrote the following script to solve this challenge.

```python
#!/usr/bin/python

flag = [67, 78, 70, 70, 127, 117, 95, 115, 64, 70, 100, 120, 83, 100, 96, 80, 99, 65, 38, 112, 39, 104]

license = ''.join(chr(i) for i in flag)

chars = []
for x in range(len(flag)):
	c = ord(license[x]) ^ x
	chars.append(c)

print ''.join(chr(i) for i in chars)
```

Running this will give us the flag!

`pYtHOns_in_sP4c3`
