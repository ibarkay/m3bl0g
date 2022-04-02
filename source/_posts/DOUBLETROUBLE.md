---
title: DOUBLETROUBLE
date: 2021-09-15 08:52:52
tags:
---

![alt](dubletrubleNmap.png)

<!-- more -->

Nmap scan for open ports:
![alt](dubletrubleNmap.png)
DirBuster (gobuster) for path's:
![alt](dubletrubleGobuster.png)
We got the qdpm version:
![alt](dubletrubleQdpmVersion.png)
Lets check for available exploit's:
![alt](dubletrubleQdpmExploits.png)
After checking the RCE's exploits we have to get a user in the qdpm, we haven't got one so lets check whats on /secret :
![alt](dubletrubleSecret.png)
Nothing special here (expats maybe a user name , but we need the full email and pass...) . so lets check if there is something hiding in the picture with steg tools:
![alt](dubletrubleStegCreds.png)
We got credz:
![alt](dubletrubleCredz2.png)
So lets Exploit :
![alt](dubletrubleExploit.png)
We got RCE:
![alt](dubletrubleRCE.png)
And a reverse shell:
![alt](dubletrubleRverseShell.png)
Checking for sudo rights we can see we can use awk as root , so we just start it as sudo and call for a shell :) :
![alt](dubletrubleROOT.png)
Nice EZPZ machine.
