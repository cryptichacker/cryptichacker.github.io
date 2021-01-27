---
layout: post
title: Tryhackme - ColddBox - Easy
date: 2021-01-25 19:00:00 +0000
category: [Tryhackme, Easy]
tags: [thm,linux,easy,boot2root,gtfobins,reverse-shell]
---

---
ColddBox - Easy is a beginner level room in Tryhackme. The final objective is to get the user and root flag.

<br>
![cover_image](/assets/img/tryhackme/coldboxeasy/1.png)

**Author** | C0ldd
**Description** | An easy level machine with multiple ways to escalate privileges.


Deploy the VM and lets go.

## <ins>Enumeration</ins>
---
Let's start with the usual nmap scan.
<br>
![nmap_scan](/assets/img/tryhackme/coldboxeasy/nmap_scan.png)
<br>
The results from the nmap scan showed that the ports ```80/http``` and ```4512/ssh```. It is unusual for ```ssh``` to be working in the port ```4512``` is what I thought...but it's fine, let's do this.
 

The first thing I noticed after checking the ```http service``` is that it runs ```Wordpress```. Moreover, the username can be found in the introduction comment.
<br>
![wordpress_coldbox](/assets/img/tryhackme/coldboxeasy/2.png)
<br>
I tried common credentials in the page ```/wp-admin``` but my effort was of no use. So, I used ```gobuster``` to find the hidden directories.
<br>
<pre>gobuster dir -u http://machine-i-addr/ -w /usr/share/dirb/wordlists/common.txt</pre>
<br>
![gobuster_result](/assets/img/tryhackme/coldboxeasy/3.png)
<br>
After visiting the ```/hidden``` dir in the webpage, I found this message.
<br>
![hidden_dir](/assets/img/tryhackme/coldboxeasy/4.png)
<br>
This further backed up the fact that ```c0ldd``` is the username.

I used ```wpscan``` to bruteforce the password for that username using the ```rockyou.txt``` wordlist.
<br>
<pre>wpscan -U c0ldd -P path-to-rockyou.txt --url http://machine-ip-addr/</pre>
<br>
By the way, you can also use ```hydra``` to bruteforce the password.
<br>
![wpscan_bruteforce](/assets/img/tryhackme/coldboxeasy/5.png)
<br>


## <ins>Reverse shell</ins>
---
A common way to get the reverse shell from when the webpage uses ```Wordpress``` is to edit the template file. So, I edited the ```twenty fifteen/404.php``` file with the contents of ```php reverse shell file```. Remember to change the ```ip``` and ```port``` fields in the file.
<br>
![php_reverse_shell](/assets/img/tryhackme/coldboxeasy/6.png)
<br>
Click on update file in the page. Before accessing the page remember to open a netcat listener in your local machine. Then go to...
<br>
<pre>http://machine-ip-addr/?p=404.php</pre>
<br>
The reverse shell successfully popped up. Stabilize the shell using ```python pty```.
<br>
![reverse_shell](/assets/img/tryhackme/coldboxeasy/7.png)
<br>


## <ins>User flag</ins>
---
I searched around a bit and found that the ```wp-config.php``` file contains the password for the used ```c0ldd```. It can be found in the ```/var/www/``` directory.
<br>
![user_pass](/assets/img/tryhackme/coldboxeasy/8.png)
<br>
We can change the user to ```c0ldd``` and get the user flag.
<br>
![user_flag](/assets/img/tryhackme/coldboxeasy/9.png)
<br>
We got the user flag...let's also get the root flag.

## <ins>Root flag</ins>
---
After messing around for a while to find a way for priviledge escalation, I finally ended up with using ```sudo -l```.
<br>
![priv_esc](/assets/img/tryhackme/coldboxeasy/10.png)
<br>
I searched for ```ftp``` in ```gtfobins``` and found the commands to spawn the root shell.
<br>
<pre>
sudo ftp
!/bin/bash
</pre>
<br>
Going into the ```/root``` directory we can get the ```root flag```.
<br>
![root_flag](/assets/img/tryhackme/coldboxeasy/12.png)
<br>
I tried and found other ways for priviledge escalation but for this post this should do. I also insist you to try out other ways for priviledge escalation. Good luck!!!

That's it folks. Happy hacking!!!
