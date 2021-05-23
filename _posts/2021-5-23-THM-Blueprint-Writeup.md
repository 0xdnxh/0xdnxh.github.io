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

<img src="/images/THM/Blueprint/nmap_output.PNG" width="400" height="400"/>

From this output we understand that the following interesting ports are open:
<ul>
  <li>80</li>
  <li>8080</li>
  <li>443</li>
  <li></li>
</ul>

Navigating to http://10.10.7.76 , we get a Server error:
<img src="/images/THM/Blueprint/servor_error.PNG" width="200" height="200"/>

However, navigating to https://10.10.7.76 or 10.10.7.76:8080 we get the following page:
<img src="/images/THM/Blueprint/oscommerce_page.PNG" width="400" height="400"/>

Interesting! Navigating inside the folder, we find two folders: catalog/ and docs/
<img src="/images/THM/Blueprint/indexof_oscommerce.PNG" width="400" height="400"/>