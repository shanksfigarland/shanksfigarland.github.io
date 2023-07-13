---
title: Dev(Insecure)Ops - AoC3 - Day 14 - Walkthrough 
date: 2021-12-17 10:00:00 +07:00
tags: [tryhackme, aoc3, hacking, cybersecurity, walkthrough]
description: 
---

## Learning Objectives

- Understanding the CI/CD concept
- Overview of risks associated with CI/CD
- Having a basic understanding of CI/CD exploitation vectors

----

First thing I did was to get the IP and export it to make it easier, like so:

```bash
$ export ip=[targetIP]
```
I checked the website to find some clues... 'nothing' but The Grinch.

![Website](/assets/img/aoc3day14/2.png "The Grinch")

Next, I ran gobuster on the website to check for directories, found some interesting ones:

```bash
$ gobuster dir -u 10.10.65.172 -w /usr/share/seclists/Discovery/Web-Content/common.txt -t 50
```
Results:
![Gobuster](/assets/img/aoc3day14/1.png "Gobuster")

As follows I checked both *admin* and *warez*.

admin            |  warez
:-------------------------:|:-------------------------:
![Admin](/assets/img/aoc3day14/4.png "Admin") |  ![Warez](/assets/img/aoc3day14/3.png "Warez")

Nothing interesting on warez other than the *"The loot folder on this target is where I send all date I collect. I've made page that updates regularly and prints the contents of the loof folder"*, nice. So let's move on to admin.
Looks promising. Looks like it's ls'ing some files.

![ls](/assets/img/aoc3day14/5.png "ls")

Checked for the source-page and it has a file named ```/ls.html```.

Connected to SSH using:
```
username: mcskidy
password: Password1
```
Upon connecting, we find a few folders, */home/thegrinch/loot* and */home/thegrinch/scripts* being the most important.

The contents of *loot.sh* script gives us a base to understand the ```ls.html``` file from earlier.

Using ```ls -la``` to check for the files permissions, we find out that *loot.sh* has written access which we can use to change the code inside to our needs and its owned by *root*.

![permissions](/assets/img/aoc3day14/7.png "Permissions")

What if we use the contents of the file to execute a command towards ```/etc/shadow```?

I then used ```cat /etc/shadow``` and reloaded the */admin* page on my browser to check if it worked.

![File](/assets/img/aoc3day14/6.png "File")

Checking the page............ and boom. It works. 

The code            |  Admin Page with the contents of /etc/shadow
:-------------------------:|:-------------------------:
![code](/assets/img/aoc3day14/9.png "code") |  ![/etc/shadow](/assets/img/aoc3day14/8.png "/etc/shadow")

This means that any command we execute it'll be run as root.

We now change */etc/shadow* to the path of *check.sh* to read its content. 

![check.sh](/assets/img/aoc3day14/9.png "check.sh")

We wait a few minutes and reload */admin* again to check its contents.

![check.sh](/assets/img/aoc3day14/10.png "check.sh")

This script checks the existence of a file called "remindme.txt" in the */loot* folder. 
It will print the password of the grinch in a html file called */pass.html*.

![remind](/assets/img/aoc3day14/11.png "remindme")

Checking the html file we can confirm the password is *ELFSareFAST*.
Maybe we could use it for something?

![pass](/assets/img/aoc3day14/12.png "pass")

I tried to login into SSH again but now as *thegrinch*. 

Once connected, to find the flag I used the following:
```bash
$ find / -name flag.txt 2>/dev/null
```
and cat the flag. 

BOOM! Done!

![flag](/assets/img/aoc3day14/13.png "flag")









