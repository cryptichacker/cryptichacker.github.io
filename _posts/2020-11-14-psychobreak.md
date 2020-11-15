---
layout: post
title: Psychobreak
date: 2020-11-14 19:00:00 +0000
category: [Tryhackme, Easy]
tags: [thm]
image: /assets/img/tryhackme/psychobreak/room-icon.jpeg
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

# Task 1
---
<br>
![nmap_scan](/assets/img/tryhackme/psychobreak/nmap.PNG)
<br>
<p>A simple nmap scan would give the answers to the first task. So I tried to login to the ftp service. Anonymous login is not enabled in ftp service. So let’s move on to http.</p>


# Task 2
---
<br>
![http_service](/assets/img/tryhackme/psychobreak/3.png)
<br><p>In the source page we can find the room name. Let’s go into the room.</p>

<br>
![Sadist_room](/assets/img/tryhackme/psychobreak/4.png)
<br><p>We can get the key to the locker room by clicking the given link. But wait… seconds after clicking the link the the background changes and a alert box asking for the key pops up.</p>

<br>
![second_sadist_room](/assets/img/tryhackme/psychobreak/5.png)
<br><p>After entering the key, we can see the locker room page. After looking at the link to the map, it seems we need to decode the piece of text to get the map.</p>

<br>
![Locker_room](/assets/img/tryhackme/psychobreak/6.png)
<br><p>At first I thought it was rot13 encoded but I was wrong. Go the link [vigenere-solver](https://www.guballa.de/vigenere-solver) and choose the correct variant given in the image to decode the text</p>

<br>
![decode](/assets/img/tryhackme/psychobreak/7.png)
<br><p>After decoding, enter the decoded text to get access to map. Let’s move on to the next room…</p>

<br>
![Safe_haven](/assets/img/tryhackme/psychobreak/8.png)
<br><p>In the source code of the page we can find a hint.</p>

<br>
![Safe_haven_hint](/assets/img/tryhackme/psychobreak/11.png)
<br><p>I had a hard time in this part. I tried enumerating using various methods but it took me more than a hour to finally find this.</p>


