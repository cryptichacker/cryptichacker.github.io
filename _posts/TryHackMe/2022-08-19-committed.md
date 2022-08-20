---
layout: post
title: Tryhackme - Committed
date: 2022-08-19 19:00:00 +0000
category: [Tryhackme, Easy]
tags: [thm,git,easy]
---

---
Committed is a beginner level room in Tryhackme. If you know a little bit of **Git**, this room will be a piece of cake for you. 
<br>

![cover_image](/assets/img/tryhackme/committed/7.png)

**Author** | tryhackme and cmnatic
**Description** | One of our developers accidentally committed some sensitive code to our GitHub repository. Well, at least, that is what they told us...


Deploy the VM and let's Capture the flag.

## <ins>Task Files</ins>
---
Read the description and download the task files after the machine starts.
<br>

![task_files](/assets/img/tryhackme/committed/1.png)
<br>

I transferred the **committed.zip** file from the remote machine to my local machine since I am comfortable using my own machine.

---
## <ins>Flag</ins>
---
I unzipped the file and found that it was a git repository. 
<br>

<pre>unzip committed.zip</pre>

![unziped_files](/assets/img/tryhackme/committed/2.png)

I only found two files named main.py and the usual Markdown file of a git repository. The first thing I did was using the commit id to look through the previous versions of the repo using the following commands
<br>
<pre>
git log                 # To get all the commit id
git show [commit_id]    # To get the file changes for the respective commit id
</pre>

![commit_id](/assets/img/tryhackme/committed/4.png)
<br>

I couldn't find anything useful after looking at the previous commits. So, I decided to check if there is any other branch and yes....it was there.
<br>


To look for local branches in an git repository use:
<br>
<pre>git branch</pre>
And to change to another branch use:
<br>
<pre>git checkout [branch_name]</pre>
In this case, we'll be using
<br>
<pre>git checkout dbint</pre>

![branch_change](/assets/img/tryhackme/committed/5.png)
<br>

After looking at the log using the ```git log``` command I found an interesting git commit message. I got the flag after using the ```git show [commit_id]``` command
<br>

![flag](/assets/img/tryhackme/committed/6.png)

That's it for this post. Happy Hacking!!
