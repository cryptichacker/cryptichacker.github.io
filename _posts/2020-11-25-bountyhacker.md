---
layout: post
title: Tryhackme - Bounty Hacker
date: 2020-11-25 19:00:00 +0000
category: [Tryhackme, Easy]
tags: [thm,linux,easy,tar,gtfobins]
---

---
<p>Bounty Hacker is a beginner level room in Tryhackme. The objective is to get the user and the root flag.</p>
<br>
![cover_image](/assets/img/tryhackme/bountyhacker/1.png)

**Author** | Sevuhl
**Description** | You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker!.

<p>Deploy the VM and lets go.</p>

## <ins>Enumeration</ins>
---
Let's start with a nmap scan.
<br>
![nmap_scan](/assets/img/tryhackme/bountyhacker/nmap_scan.png)
<br>

From the nmap scan we can see that ftp Anonymous login is enabled. I checked the http service and couldn't find anything useful. So, Login to the ftp server.
<br>
![ftp_server](/assets/img/tryhackme/bountyhacker/2.png)
<br>

Found two text files. Download it using the command ```get file-name```.

After reading the files I concluded that ```locks.txt``` is a wordlist and the username is ```lin```.
<br>
![task.txt](/assets/img/tryhackme/bountyhacker/3.png)
<br>


## <ins> User flag</ins>
---
Let's use what we found to bruteforce ```ssh``` using ```hydra```.
<br>
<pre>hydra -l lin -P locks.txt ip-addr ssh</pre>
<br>
![hydra](/assets/img/tryhackme/bountyhacker/4.png)
<br>

With the username and password in hand, let's login to ssh.
<br>
![user_flag](/assets/img/tryhackme/bountyhacker/5.png)
<br>
Boom!! We got the user flag.


## <ins>Root flag</ins>
---
I used ```sudo -l``` to see if there are any files that can be run as root.
<br>
![suod_check](/assets/img/tryhackme/bountyhacker/6.png)
<br>

I searched ```gtfobins``` for tar and got this.
<br>
![gtfo_bins](/assets/img/tryhackme/bountyhacker/7.png)
<br>

<pre>sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh</pre>
<br>

Run the command to get the root shell. The root flag is there in the ```/root``` dir.
<br>
![root_flag](/assets/img/tryhackme/bountyhacker/8.png)

Thats it folks. Happy hacking!!!
