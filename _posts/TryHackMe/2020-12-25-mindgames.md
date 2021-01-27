---
layout: post
title: Tryhackme - Mindgames
date: 2020-12-25 19:00:00 +0000
category: [Tryhackme, Medium]
tags: [thm,linux,medium,openssl,capabilities,gtfobins]
---

---
Mindgames is a medium level Tryhackme room. I had a good time doing this room and learnt a lot while doing it. I hope you'll learn something from this post too. The final objective is to get the user and root flag.

<br>
![cover_image](/assets/img/tryhackme/mindgames/1.png)

**Author** | NinjaJc01
**Description** | Just a terrible idea...

<p>Deploy the VM and lets go.</p>

## <ins>Enumeration</ins>
---
Let's start with a nmap scan.
<br>
![nmap_scan](/assets/img/tryhackme/mindgames/nmap_scan.png)
<br>
I jumped straight into the ```http service``` and found this page.
<br>
![http_homepage](/assets/img/tryhackme/mindgames/2.png)
<br>
I instantly recognized that the lines of gibberish is a ```esolang``` called ```brainf@@k```. I decoded it using ```dcode``` and got this.
<br>
![first_program](/assets/img/tryhackme/mindgames/3.png)
<br>
After decoding, I found that the language was python. I confirmed it after running the same ```brainf@@k``` code with the interpreter in the webpage.
<br>
![first_program_execution](/assets/img/tryhackme/mindgames/4.png)
<br>

I tried the same process for the second example code that is given in the webpage and that the result is the same. So, the conclusion here is that we can execute any ```python3``` code encoded with the ```esolang``` ```brainf@@k``` in the interpreter.
<br>
![second_program](/assets/img/tryhackme/mindgames/5.png)
<br>
![second_program_execution](/assets/img/tryhackme/mindgames/6.png)
<br>

## <ins>User flag</ins>
---
I grabbed the python reverse shell and encoded it in ```brainf@@k``` using ```dcode```.
<br>
![reverse_code](/assets/img/tryhackme/mindgames/7.png)
<br>
Copy and paste the encoded text into the interpreter and click on ```Run it!```. Remember to open a netcat listener.
<br>
![user_flag](/assets/img/tryhackme/mindgames/8.png)
<br>
Viola!! We got the user flag.

## <ins>Root flag</ins>
---
I transferred ```linpeas``` into the remote machine and ran it. I checked the results of linpeas and found some interesting ```linux capabilities```. 
<br>
![linux_cap](/assets/img/tryhackme/mindgames/9.png)
<br>
It seems ```Openssl``` has ```setuid``` capabilities which means ```openssl``` can change the ```user identifier (uid)``` for any user.
<br>
![cap_man_pages](/assets/img/tryhackme/mindgames/14.png)
<br>
And the ```ep``` at the end denotes that the binary has all capabilities ```permitted(p)``` and ```effective(e)``` from the start (which I referred from ```stackoverflow```). If you are interested in learning about ```linux capabilities``` you can [visit the man pages](https://man7.org/linux/man-pages/man7/capabilities.7.html).

I also got the command on how to load a library using openssl from ```gtfobins```.
<br>
![gtfo_bins](/assets/img/tryhackme/mindgames/13.png)
<br>

So, all that's left is to create a openssl engine and load it to get the root shell. Let's get going.


### <ins>Create openssl engine</ins>
---
I googled on how to create a openssl engine and the first link gave away the answer. If you didn't get to the correct place, here is the link to [openssl page](https://www.openssl.org/blog/blog/2015/10/08/engine-building-lesson-1-a-minimum-useless-engine/). I copied the first code and made some modifications to it.

Since I am not well versed in c programming, I had to google some information on how to set uid and get a shell using ```c```. I finally arrrived at this.
<br>
![openssl_engine](/assets/img/tryhackme/mindgames/18.png)
<br>


### <ins>Compiling the code</ins>
---
The command to compile the code is given in the ```openssl``` page.
<br>
<pre>gcc -fPIC -o file-name.o -c file-name.c && gcc -shared -o filename.so -lcrypto file-name.o</pre>
<br>
I didn't understand some arguments and formats used. So, I searched and arrived at these answers.

The ```-fPIC``` option is used to make the ```gcc``` compile the code into a ```Position Independent Code``` so that it can loaded as a library.

The ```-lcrypto``` is ```libcrypto``` library which is used to get a static link.

And as for .a and .so formats...
<br>
![file_formats](/assets/img/tryhackme/mindgames/16.png)
<br>


### <ins>Root Shell</ins>
---
Transfer the ```.so``` file to the remote machine and change the permissions of the file to make it ```executable```.

Finally, use the command that we got from ```gtfobins``` to get the root shell.
<br>
![root_flag](/assets/img/tryhackme/mindgames/12.png)
<br>
Banzai!! We got the root flag.
<br>

That's it folks. Happy hacking!!!
