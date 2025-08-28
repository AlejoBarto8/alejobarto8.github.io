---
layout: post
title:  "Mirai Writeup - Hack The Box"
date:   2025-08-27
desc: ""
keywords: "HTB,eJPT,Linux,Raspberry Default Credentials Abuse,Sudo Group Abuse,Connected External Device Recovery File,Easy"
categories: [HTB]
tags: [HTB,eJPT,Linux,Raspberry Default Credentials Abuse,Sudo Group Abuse,Connected External Device Recovery File,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I start a new **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab, but this time I choose a lab rated with an **Easy** complexity by the community, because the last ones I did demanded me a lot of time and energy, so I want to control the **Cognitive Load** in order not to overdemand myself and damage my **Learning strategy**. Although the box does not have much difficulty, I did have to invest time in researching technologies that I did not have the opportunity to work with until I faced this lab. I'm going to access my **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** account to spawn the machine and start my writeup.

<br /><br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I check by sending a trace to the machine with `ping` to corroborate that I already have connectivity with the machine and then with `whichSystem.py` (**[hack4u](https://hack4u.io/){:target="_blank"}** tool) I measure with great proximity the **Operating System** of the machine (thanks to the **TTL** value). Now I can start the main phase of every **Pentest** or **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab, the **Reconnaissance**, using the `nmap` tool to list the open ports that I will have to audit and engage. Then with custom `nmap` scripts I leak as much information as possible about the services, and their versions, on each port exposed out of the box. I get a lot of interesting information about ports and services that I don't find very often, but also with all the collected information I can investigate with a search engine the **codename** of the machine and in case I find different ones, it is a sign that containers are being deployed (**this is the case**).

```bash
ping -c 1 10.10.10.48
whichSystem.py 10.10.10.48
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.48 -oG allPorts
nmap -sCV -p22,53,80,1373,32400,32469 10.10.10.48 -oN targeted
cat targeted
#       --> OpenSSH 6.7p1 Debian 5+deb8u3
#       --> lighttpd 1.4.35
```

<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since I have the **DNS** protocol available I can use the `dig` tool to leak information such as **subdomains** or even perform an **[AXFR](https://beaglesecurity.com/blog/vulnerability/dns-zone-transfer.html){:target="_blank"}** (**DNS zone transfers**) attack but I don't get the expected results. My next target is the **HTTP** protocol, with two web applications accessible on port **80** and **32400**, which always represents the largest attack surface because of the possibility to interact with them. With `whatweb` from my shell and with **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** from the browser I manage to disclose the technology stack behind the web applications, but for some reason I don't have access to the port **80** service.

```bash
dig @10.10.10.48
dig @10.10.10.48 ns
dig @10.10.10.48 mx
dig @10.10.10.48 axfr

cat targeted | grep http
whatweb http://10.10.10.48 http://10.10.10.48:32400

# http://10.10.10.48/
# http://10.10.10.48:32400/web/index.html
```

<br/>
<img src="{{ site.img_path }}/mirai_writeup/Mirai_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I search with `searchsploit` for some vulnerability to the **[Plex Media Server](https://support.plex.tv/articles/200288286-what-is-plex/){:target="_blank"}** service that `nmap` had leaked me, but I don't have the version. Before continuing the **Reconnaissance** phase I'm going to add to my **hosts** file the domain **miriai.htb**, maybe that is the problem why I'm not being able to access the service on port **80**. I'm lucky and I can use `whatweb` again to disclose information about the technology stack behind the application, which happens to be **[Pi-hole](https://docs.pi-hole.net/){:target="_blank"}** and I can also access from my browser where I can find the version of it. With `searchsploit` I find an exploit that would allow me to execute commands remotely (**RCE**), but I prefer to analyze the code and I find a path that redirects me to the authentication panel.

> **[Plex](https://support.plex.tv/articles/200288286-what-is-plex/){:target="_blank"}**: gives you one place to find and access all the media that matters to you. From personal media on your own server, to free and on-demand Movies & Shows or live TV, to streaming music, you can enjoy it all in one app, on any device.

> The **[Pi-hole](https://docs.pi-hole.net/){:target="_blank"}** is a **DNS sinkhole** that protects your devices from unwanted content, without installing any client-side software.

```bash
searchsploit plex

nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping -c 1 mirai.htb
whatweb http://mirai.htb

# http://mirai.htb/
# please ask the owner of the Pi-hole in your network to have it whitelisted

searchsploit pi-hole 3
searchsploit -x python/webapps/48727.py
# response = session.post(url + "/admin/index.php"...

# http://10.10.10.176/admin/
```

<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I look for the default credentials to access the **Pi-hole** Dashboard and they are not the correct ones, but in the article I found it informs me that it is usually the one used for **[SSH connection to the Raspberry Pi OS](https://discourse.pi-hole.net/t/password-for-pre-configured-pi-hole/13629){:target="_blank"}**. It is indeed the username (**pi**) and password (**raspberry**) that allows me to connect to the machine to perform the first enumeration commands in addition to accessing the first flag. The most important thing I can find is that my user belongs to the **sudo** group, so I can execute all the commands impersonating the user with maximum privileges. I try to see the content of the last flag but I get a message informing me that there is a backup of the flag on the **USB stick**. With `df` I enumerate the file system where I find the **USB stick** and with `strings` I can access the content of the last flag. Machine finally engaged.

> **Raspberry Pi OS password**: The **ssh** login password is not the same as the **Pi-Hole** login password, unless you set it up this way. As installed from a new Raspbian image, the default password for user **pi** is **raspberry**.

```bash
# http://mirai.htb/admin/index.php?login
# pi:raspberry

ssh pi@10.10.10.48
whoami
hostname -I

export TERM=xterm
uname -a
lsb_release -a
id
groups
# sudo
sudo su

cat root.txt
# I lost my original root.txt! I think I may have a backup on my USB stick...

man df
# df - report file system space usage

df -h
# /dev/sdb        8.7M   93K  7.9M   2% /media/usbstick

man strings
# strings - print the sequences of printable characters in files

strings /dev/sdb
```

<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine that I succeed in **engaging**, which does not represent much complexity but that I always take advantage of to learn new vulnerabilities and technologies. As I am very new in this field, for me everything is unknown and I'm passionate about learning new techniques to engage more and more complex labs. The **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform is my favorite because it allows me to **grow professionally** and **have fun** at the **same time** with excellent machines, whose configuration is made by professionals who share their knowledge. Now I will continue my learning with the next lab without forgetting to kill the **Mirai**.

<br /><br />
<img src="{{ site.img_path }}/mirai_writeup/Mirai_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
