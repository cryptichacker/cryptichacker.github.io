---
layout: post
title: Tryhackme - The Server From Hell
date: 2020-12-03 19:00:00 +0000
category: [Tryhackme, Medium]
tags: [thm,linux,medium,nfs]
---

---
<p>The Server from Hell is a medium level room in Tryhackme. The final objective is to get the user and root flag.</p>
<br>
![cover_image](/assets/img/tryhackme/theserverfromhell/1.png)

**Author** | DeadPackets
**Description** | Face a server that feels as if it was configured and deployed by Satan himself. Can you escalate to root?

<p>Deploy the VM and lets go.</p>

## <ins>Enumeration</ins>
---
![description](/assets/img/tryhackme/theserverfromhell/2.png)
<br>
Let's start with the port 1337. I used ```netcat``` to get the ```banner```.
<br>
![netcat_1337](/assets/img/tryhackme/theserverfromhell/3.png)
<br>
I did a nmap scan for the ports 0-100 with the ```banner``` script but the results didn't make any sense. So, I ran ```netcat``` to grab all the banners.
<br>
<pre>for i in {0..100};do nc ip-addr $i; echo ; done</pre>
<br>
And I found this..
<br>
![image_ports](/assets/img/tryhackme/theserverfromhell/port12.png)
<br>

I grabbed the banner for the ```port 12345```.
<br>
![nfs_share](/assets/img/tryhackme/theserverfromhell/4.png)
<br>


## <ins>Flag</ins>
---
Enumerate the ```nfs share``` with showmount.
<br>
<pre>showmount -e ip-addr</pre>
<br>
![nfs_share](/assets/img/tryhackme/theserverfromhell/5.png)
<br>

I created a temporary directory to mount the nfs share in my local machine.
<br>
<pre>sudo mount -t nfs ip-addr:/home/nfs ./tmp</pre>
<br>

There is a zip file named ```backup.zip``` inside the mounted nfs share.

I used ```fcrackzip``` to crack the password.
<br>
<pre>fcrackzip -v -D -u backup.zip -p path-to-wordlist</pre>
<br>
![zip_crack](/assets/img/tryhackme/theserverfromhell/6.png)
<br>

Unzip the zip file. The inflated directory contains the answer to the first question. I also took note of the username ```hades```.
<br>
![inflated_list](/assets/img/tryhackme/theserverfromhell/7.png)
<br>

The ```hint.txt``` file shows a range. So, I tried doing a ```nmap scan``` for the given range.
<br>
<pre>nmap -sV --script=banner ip-addr -p2500-4000</pre>
<br>
On a whim, I searched for ```OpenSSH``` and it was actually there. Luckyy!
<br>
![nmap_scan](/assets/img/tryhackme/theserverfromhell/8.png)
<br>


## <ins>User flag</ins>
---
We have the ```private key``` and the ```ssh port```. Without further ado, let's login to ssh. Ahh...**Don't forget to change the permission to ```600``` for the private key**.
<br>
![ssh_login](/assets/img/tryhackme/theserverfromhell/9.png)
<br>
![ssh_noshell](/assets/img/tryhackme/theserverfromhell/10.png)
<br>

After logging in, it throws error for every command. From the error message I found that it is running ```irb```.

From ```gtfobins``` I found a way to get a shell.
<br>
![gtfobins_1](/assets/img/tryhackme/theserverfromhell/11.png)
<br>

![user_flag](/assets/img/tryhackme/theserverfromhell/12.png)
<br>

Way to go!! We got the user flag.


## <ins>Root flag</ins>
---
As a primary enumeration for ```priviledge escalation``` I used ```Linpeas```. After a quick go through of the results I found ```tar``` with ```user capabilities```.
<br>
![user_cap](/assets/img/tryhackme/theserverfromhell/14.png)
<br>

From ```gtfobins``` I found this.
<br>
![gtfobins_2](/assets/img/tryhackme/theserverfromhell/15.png)
<br>
We can use this to get the root flag directly.
<br>
![root_flag](/assets/img/tryhackme/theserverfromhell/16.png)
<br>

But, it is no fun getting only the flag. Let's also root the box.

I dumped the ```/etc/shadow``` file which contains all the passwords in encypted form.
<br>
![etc_shadow](/assets/img/tryhackme/theserverfromhell/17.png)
<br>

Copy the contents of the root hash to a seperate file and crack it using ```john```.
<br>
![crack_pass](/assets/img/tryhackme/theserverfromhell/19.png)
<br>
We got the root password.
<br>
![root_shell](/assets/img/tryhackme/theserverfromhell/20.png)
<br>

Viola!! We cleared the room.

That's it folks. Happy hacking!!!
