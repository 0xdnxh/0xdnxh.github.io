---
layout: post
title: THM Bounty Hacker writeup
categories: THM
---
## Introduction
<img src="/images/THM/BountyHacker/banner.PNG" width="800" height="150"/>

In this blog post you will find a writeup for the [Bounty Hacker](https://tryhackme.com/room/cowboyhacker) room on TryHackMe. It's a vulnerable Linux machine and since this room has a few questions to answer, you will find my answers as well as some additional information I gathered along the way.

## Enumeration
First of all, let's run nmap to get more information about the machine and discover open ports. The command that I used is ```nmap -sC -sV 10.10.37.218``` , the option -sV is used to determine service/version info, -sC to use the default script of nmap and -Pn to disable host discovery. The command outputs the following: 

<img src="/images/THM/BountyHacker/nmap_output.PNG" width="800" height="800"/>

From this output we understand that the following ports are open:
<ul>
  <li>21 : running FTP, we notice that Anonymous FTP login is allowed</li>
  <li>22 : SSH </li>
  <li>80</li>
</ul>
We also see that the OS is Linux. 

## FTP anonymous login
Let's try to log in FTP using anonymous:anonymous as credentials and look around by listing system files :

<img src="/images/THM/BountyHacker/fploginls.PNG" width="800" height="800"/>

We can see that there are two files **locks.txt** and **task.txt**, let's go ahead and download them :

<img src="/images/THM/BountyHacker/tasklocks_download.PNG" width="800" height="800"/>

