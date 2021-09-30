# THM-Rick-Morty-Write-up
Step by step guide to Rick and Morty Themed CTF by Try Hack Me



# Pickle Rick

Navigating to the IP address provided we are faced with a little backstory.

Rick forogot his password, and he needs you to log in. (Where?)
In the comments of the source we find that his username is: R1ckRul3s.

## Enumeration

Let's kick of gobuster and nmap!


Gobuster:
```
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-1.0.txt -u http://10.10.46.208

Only returns /assets.
```
Tried /rick, /pickle, /picklerick, /morty. Nothing

nmap:
```
nmap -T5 -p- 10.10.46.208
...
64754 closed ports, 779 filtered ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
...

then, follow up with -sC -sV -A

22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 42:7a:19:e6:a2:e8:64:82:2c:d8:ae:08:e1:ac:0e:15 (RSA)
|   256 3c:be:06:99:4c:2c:c9:08:b8:83:4a:f1:43:0d:55:08 (ECDSA)
|_  256 1a:91:40:a2:93:ac:22:c5:e4:e3:77:f7:e7:83:17:22 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Rick is sup4r cool
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Summarising our findings thus far:
0. Username: R1ckRul3s
1. Could the password be Rick is sup4r cool? 
2. There is the /assets/ directory storing e.g. the background image & javascript
3. There are no cookies, nor custom javascripts
4. SSH disallows password conection
5. No CVE for that particular Apache server


At this point I thought I was stuck.
Then I realised that directory discovery on a webserver is not all.
All kinds of files could lie in the root or the /assets directory!
So to find the files, I re-run gobuster with the inbuilt kali wordlist:

```
gobuster dir -w /usr/share/wordlists/dirb/big.txt -u http://10.10.46.208
```

We get two more finds!
/robots.txt
/login.php

Robots.txt contains ricks favourite phrase: Wubbalubbadubdub

## Foothold

This is a password that works with the previously found username, on the login.php website!

Logins website contains a comment, which looks very BASE64-ish!
Sending the phrase to decoder, we find it is 8 times base64 encoded "rabbit hole"
Pretty pointless, but fun. Thanks creator!

There is a command panel on the website, so naturally we list the directory and check where we're at.
No surprise that
```
$ pwd
```
Returns /var/www/html

However, 
```
$  ls
```

Returns:
* Sup3rS3cretPickl3Ingred.txt
* Assets
* Clue.txt
* Denied.php
* Login.php
* Portal.php
* Robots.txt

The first text file tells us the first secret ingredient.

Clue challenges us to discover more about the system, so we wonder to /home/rick directory.

There we find the second ingredient.

Last ingredient can be guessed to be located in the root directory.

Luckily the webserver is heavily misconfigured, as the www-data user can execute all comands as sudo, without need for password.
So we check content of the /root directory and there it is! '3rd.txt'. 
Cat-out the last ingredient! 

Now we can turn rick back into human form!

## Getting a shell
For the sake of practice, especially given that I have been studying reverse vs bind shells recently I attempted to get a shell (even though the command panel allows us for full exploitation).

Turn out there was a set of secuirty practices in place:
Netcat and Socat are disabled, so no way to get a shell that way.
PHP and Python reverse shells would drop immideitaly 
PHP bind shell wouldn't work
But finally Python bind shell did work!

Shells I tried were from:
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology and Resources/
