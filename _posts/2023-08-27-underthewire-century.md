---
layout: post
title:  "Under The Wire - Century Game"
date:   2023-08-27
desc: "PowerShell Console Training - Novice - Under The Wire Plataform"
keywords: "Windows,PowerShell,Easy"
categories: [WINDOWS]
tags: [Windows,Training,Easy]
icon: icon-htb
---


**[Under the Wire](https://underthewire.tech/){:target="_blank"}** is a project that started in 2015, and is the product of two people in the cybersecurity field. It is a platform that seeks to develop technical skills, but in a Windows environment, which gives us Linux lovers to get out of our comfort zone and expand our skills, we must not forget that many, if not all, audits must be performed in organizations that employ Microsoft's OS. To log into each game an **SSH** connection to the server must be established, so I am going to download the [Putty](https://www.putty.org/){:target="_blank"} client on my machine and proceed to install it.
<br/><br/>

<img src="{{ site.img_path }}/underthewirecentury/UTH_century_01.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">
<br/><br/>

We set up the connection in `Putty` with the Under The Wire game server and save the session so that we don't do it repeatedly, every time we have to connect or reach a new level in each game.
<br/><br/>

<img src="{{ site.img_path }}/underthewirecentury/UTH_century_02.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/><br/>


## Level 0
----------

<img src="{{ site.img_path }}/underthewirecentury/UTH_century_06.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

We must register in the **[Under The Wire Slack channel](https://underthewire.slack.com/)** to obtain the initial credentials for each game, in the **#starthere** channel.
I'm going to start with the initial one, `Century` and then go up the difficulty. For the initial level we just need to get the connection to the server, here we go!
<br/><br/>

<img src="{{ site.img_path }}/underthewirecentury/UTH_century_04.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

We connect and I can continue with the next level. As these are initial levels I will try to reach different objectives and publish them in a single post, when the complexity or difficulty of the game increases I will create a new post.
<br/><br/>

<img src="{{ site.img_path }}/underthewirecentury/UTH_century_03.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

I can connect now again with `Putty` and the credentials **`century1`** to play in the next level.
<br/><br/>

<img src="{{ site.img_path }}/underthewirecentury/UTH_century_05.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

> Tips: To copy from **Windows** and paste into **PuTTY**, highlight the text in Windows, press *"Ctrl-C"* select the PuTTY window, and press the right mouse button to paste. To copy from PuTTy and paste into Windows, highlight the information in PuTTY and press *"Ctrl-V"* in the Windows application to paste it.
<br/><br/>

## Level 1
----------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_08.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

In this level I need find a way to get the `build version of the Powershell instance` I am using. After trying several commands and trying to interpret if they are the results I need, I use:

```Powershell
Get-ComputerInfo			# :(

[System.Environment]::OSVersion
# 10.0.14393.0

$PSVersionTable
# BuildVersion		10.0.14393.5127					# :)
```

And I get the information I need to connect with the credentials **`century2`** and advance to level 2.

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_07.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>


## Level 2
----------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_09.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>


To find the password to access as `century3`, I must look for the name of the cmdlet that performs the wget-like function inside PowerShell first.

> **cmdlet (command-let)**: is a small, lightweight command that is used in the Windows PowerShell environment. A cmdlet typically exists as a small script that is intended to perform a single specific function such as coping files and changing directories. A cmdlet and its relevant parameters can be entered in a PowerShell command line for immediate execution or included as part of a longer PowerShell script that can be executed as desired

After using several commands with different filters and using the cdmldet `Get-Help` to find a command that performs the wget-like task, I find what I need and also see in the directory the file with its corresponding name to complete the game.

```powershell
Get-Command
Get-Command -CommandType cmdlet
Get-Command -CommandType cmdlet *wget*
Get-Command -CommandType cmdlet *web*
Get-Command -CommandType cmdlet Get-*

Get-Command *web*
Get-Help Get-WebFilePath
Get-Help Get-WebRequest
Get-Help Invoke-WebRequest

dir -Force
type 443
```

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_10.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">

I can now access the next level with the password and username **`century3`**
<br/><br/>

## Level 3
----------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_11.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

To advance level, I must obtain the `total number of the Desktop folder`, for this I investigate a little with `Get-Help` and we can use several commands and use `|` to obtain the expected result. The `.count` operator can be used together with the `Measure-Object` Cmdlet to count objects in PowerShell.

```
Get-Help Get-ChildItem
# The Get-ChildItem cmdlet gets the items in one or more specified locations. If the item is a container, it gets the items inside the container, known as child items. You can use the Recurse parameter to get items in all child containers.

Get-Help Measure-Object
# The Measure-Object cmdlet performs calculations on the property values of objects. It can count objects and calculate the minimum, maximum, sum, and average of the numeric values. For text objects, it can count and calculate the number of lines, words, and characters.
```

It is possible to be polishing the command but I believe that the last one is the most appropriate for what they are asking me.

```powershell
(Get-ChildItem | Measure-Object).Count
(Get-ChildItem -Directory | Measure-Object).Count

(Get-ChildItem -Recurse | Measure-Object).Count
(Get-ChildItem -Recurse -File | Measure-Object).Count
(Get-ChildItem -Recurse -Directory | Measure-Object).Count

(Get-ChildItem -File | Measure-Object).Count
```

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_12.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">

I already have the credentials for the next level **`century4`**!
<br/><br/>

## Level 4
----------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_13.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

I am asked to access a directory and read the name of the file that is stored in it, as the directory name has blank spaces we must use single or double commas to access it.

```powershell
cd '.\Can you open me'
dir

cd ".\Can you open me"
dir
type 49125
```

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_14.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

I can now level up with the credential **`century5`**

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_15.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

## Level 5
----------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_16.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

The goal now is to get the `short name of the domain`, if I search the internet for some concepts and then use the `Get-Help` cmdlet I can try several commands to get what I need.

> Tips: A fully **qualified domain name (FQDN)** contains both a host name and a domain name. For a landing page, the fully qualified domain name usually represents the full URL or a major portion of the top-level address.
In looking at a fully qualified domain name, the host name typically comes before the domain name. The host name represents the network or system used to deliver a user to a certain address or location. The domain name represents the site or project that the user is accessing.

```powershell
Get-Help gwmi
# The Get-WmiObject cmdlet gets instances of Windows Management Instrumentation (WMI) classes or information about the available WMI classes. To specify a remote computer, use the ComputerName parameter. If the List parameter is specified, the cmdlet gets information about the WMI classes that are available in a specified namespace. If the Query parameter is specified, the cmdlet runs a WMI query language (WQL) statement.
```

```powershell
Get-WmiObject
Win32_NTDomain
Get-WmiObject -Class Win32_NTDomain
Get-WmiObject Win32_ComputerSystem

(gwmi WIN32_ComputerSystem).Domain
(gwmi WIN32_NTDomain).DomainName
(Get-ADDomain).NetBIOSName
```

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_17.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

Now that I have the necessary data, I go to the next level with the user **`century6`**

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_18.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

## Level 6
----------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_19.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

For this level, I have already found a command for level three, which by adjusting it just a little bit I get what I need, the number of folders in the Desktop folder of user `century6`.

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_20.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

I can now proceed to the next level, credentials: **`century7`**!

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_21.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

# Level 7
---------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_22.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

To know the content of the file, I am going to assume the one whose name begins with ***Readme***, I can make use of the Cmdlet `Get-ChildItem` and search recursively for the file in question.

```powershell
Get-ChildItem readme* -Recurse
```

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_23.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

Once I find and see the contents of the file, I continue to level up the game with the user: **`century8`**!

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_24.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

## Level 8
----------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_25.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

To get the number of unique entries in a file I can use the `Get-Unique` Cmdlet.

```powershell
Get-Help Get-Unique
# The Get-Unique cmdlet compares each item in a sorted list to the next item, eliminates duplicates, and returns only one instance of each item. The list must be sorted for the cmdlet to work properly. Get-Unique is case-sensitive. As a result, strings that differ only in character casing are considered to be unique.
```

To get the number of unique entries in a file I can use the `Get-Unique` Cmdlet. Adjusting a little the command I need, and with the help of `.Count`, I get what I was looking for.

```powershell
type .\unique.txt
type .\unique.txt | Get-Unique
(type .\unique.txt | Get-Unique).Count
```

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_26.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

I continue advancing in these first steps in PowerShell, credentials **`century9`**!

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_27.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

## Level 9
----------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_28.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

Now I need to get the word 161 from a file. As a first attempt, I try to divide the string in different lines using the space as divisor, but it is unsuccessful at first, since it continues being a string.

```powershell
(type .\Word_File.txt).Replace(" ","`r`n") | Select-Object -Index 160
type .\Word_File.txt | Select-Object -Index 0
(type .\Word_File.txt).Replace(" ","`r`n")[160]
type .\Word_File.txt | Select-Object -Index 0
(type .\Word_File.txt).Replace(" ","`r`n")
(type .\Word_File.txt).Replace(" ","`r`n").Count
```

But if I transform the string into an array of words, it is much easier to get the word, which in the array is in position 160, position zero corresponds to the first word in the array.

```powershell
(Get-Content .\Word_File.txt).GetType()
(Get-Content .\Word_File.txt)[9]


(Get-Content .\Word_File.txt).Split(" ")[160]
```

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_29.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

I continue to advance and I am understanding the power of PowerShell as the levels become more complex. Credentials **`century10`**!

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_30.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

## Level 10
-----------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_31.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

I am now asked for some precise words of the description of a service, as a first instance I look for a command to find the service in question, and I also find the name of it:

```powershell
Get-Service "Windows Update"
#	Name: wuauserv
```

I try some commands using `Get-Service`, but I don't get what I want.

```powershell
Get-Service "Windows Update" | Get-Member
Get-Service "Windows Update" | Format-List *
```

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_32.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

But if I use `Get-WmiObject`, in addition to using `|` and some commands I saw in previous levels, wow, I get what I want.

```powershell
Get-WmiObject win32_service | ?{$_.Name -like 'wuauserv'} | select
Get-WmiObject win32_service | ?{$_.Name -like 'wuauserv'} | select Description

(Get-WmiObject win32_service | ?{$_.Name -like 'wuauserv'} | select Description | Format-Table -HideTableHeaders | Out-String).Split(" ")[9]
```

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_33.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_34.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

The game is getting interesting, I advance to the next level thanks to the credential **`century11`**

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_35.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>


## Level 11
-----------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_36.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

Now I have to find the name of a file hidden somewhere random. I am looking for help with `Get-Help` for the Cmdlet `Get-ChildItem` (dir). I see that there are some interesting parameters to try and not drive me crazy searching manually.

```powershell
Get-Help Get-ChildItem

Get-ChildItem Hidden
Get-ChildItem -Attributes Hidden,Archive
Get-ChildItem -Attributes Hidden,Archive -Recurse
```

I find what I need with the following commands, but I also hide the error messages returned by the output.

```powershell
Get-ChildItem -Attributes Hidden,Archive -Recurse -ErrorAction SilentlyContinue
dir -Hidden -File -Recurse -ErrorAction SilentlyContinue
clearcGet-ChildItem -Attributes Hidden,Archive -Recurse -ErrorAction SilentlyContinue -Filter secret_sauce
```

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_37.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

Now I can continue with the **`century12`** credential, but I am left with a bitter taste, I could not exclude from the search those folders that were not in scope, I will polish this command better.

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_38.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

## Level 12
-----------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_39.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

I am already at level 12 and now we must obtain information from the Domain Controller, first I will use the Cmdlet `Get-Help` to search for some command related to the DC and I find an interesting one, I see its function and its parameters.

```powershell
Get-Command *DomainController*
get-help Get-ADDomainController

Get-ADDomainController
Get-ADDomainController -Discover
Get-ADDomainController -Identity "UTW"
```

I use some commands, but the most interesting thing I find is the DC Intedity Name. I am now looking for a command that can help me to get information, but from an AD Computer

```powershell
Get-Help *Computer*
Get-Help Get-ADComputer
```

Now I can search for what I need, and with everything I have learned so far, I create an oneliner and find the information I need to go to the next level.

```powershell
Get-ADComputer -Identity UTW -Properties *
Get-ADComputer -Identity UTW -Properties * | Select Description | Format-Table -HideTableHeaders
```

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_40.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

I continue with the next step, **`century13`**

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_41.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>


## Level 13
-----------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_42.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

I am asked to obtain the number of words in a file, with the Cmdlet Get-Content and Object-Measure I can access this information.

```powershell
type .\countmywords
Get-Content .\countmywords
Get-Content .\countmywords | Measure-Object -Line
Get-Content .\countmywords | Measure-Object -Character
Get-Content .\countmywords | Measure-Object -Word
```

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_43.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

Now if I can access the last level with **`century14`**

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_44.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

## Level 14
-----------

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_45.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

With all that I have learned, I think I can put together an oneline and get the last thing I am asked for, the number of times a word is repeated in a file.

```powershell
Get-Content .\countpolos
(Get-Content .\countpolos).Split(" ")
(Get-Content .\countpolos).Split(" ") | Where-Object {$_ -like "polo"}

((Get-Content .\countpolos).Split(" ") | Where-Object {$_ -like "polo"}).Count
```

<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_46.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>
<img src="{{ site.img_path }}/underthewirecentury/UTH_century_47.png" width="100%" style="margin: 0 auto;display: block; max-width: 900px;">  
<br/>

> **First [UTW](https://underthewire.tech/century) game finished, and it's the first one! Very good way to start with PowerShell**

## Resources
------------

[How to Count Objects in PowerShell](https://www.itechguides.com/powershell-count/){:target="_blank"}

[Using PowerShell to Convert String To Array](https://shellgeek.com/powershell-to-convert-string-to-array/){:target="_blank"}
