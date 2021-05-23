---
layout: post
title: THM Blueprint writeup
categories: THM
---
## Introduction
<img src="/images/THM/Blueprint/banner.PNG" width="200" height="200"/>
In this blog post you will find a writeup for the [Blueprint](https://tryhackme.com/room/blueprint) room on TryHackMe. It's a Windows machine running a vulnerable version of osCommerce, an online store solution. Since this room isn't guided like the other ones, you will find my 

## Enumeration
First we fire up nmap to get more info on the machine and find out open ports. The command that I used is ```nmap -sV -sC -Pn 10.10.7.76``` , the option -sV is used to determine service/version info, -sC to use the default script of nmap and -Pn to disable host discovery. The command outputs the following:

<img src="/images/THM/Blueprint/nmap_output.PNG" width="800" height="800"/>

From this output we understand that the following interesting ports are open:
<ul>
  <li>80</li>
  <li>8080</li>
  <li>443</li>
</ul>

Navigating to http://10.10.7.76 , we get a Server error:

<img src="/images/THM/Blueprint/server_error.PNG" width="600" height="100"/>

However, navigating to https://10.10.7.76 or 10.10.7.76:8080 we get the following page:

<img src="/images/THM/Blueprint/oscommerce_page.PNG" width="500" height="150"/>

Interesting! So now we know there's an **osCommerce version 2.3.4** application running. Navigating inside the directory, we find two folders: catalog/ and docs/

<img src="/images/THM/Blueprint/indexof_oscommerce.PNG" width="500" height="175"/>

### Fuzzing
Let's try to find which directories exist using dirb

<img src="/images/THM/Blueprint/dirboutput.PNG" width="500" height="150"/>

Nothing really interesting. Let' go back to focusing on finding an exploit related to the version.

## Exploitation
Trying to find out if there are any exploit for osCommerce 2.3.4 using searchsploit, we find some results that could potentially be helpful :

<img src="/images/THM/Blueprint/searchsploit_oscommerce.PNG" width="500" height="150"/>

The last one **php/webapps/44374.py** looks interesting, it's a Remote code execution, we can find the complete code [here](https://www.exploit-db.com/exploits/44374) :

