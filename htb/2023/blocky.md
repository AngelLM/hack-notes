---
description: '#CMS, #enumeration, #exposed credentials, #linux, #reused credentials, #sudo'
---

# Blocky

[Blocky](https://app.hackthebox.com/machines/Blocky) is an **EASY** machine from the **Hack The Box** platform. In this machine we will have to enumerate a web server until we find a .jar file with **exposed credentials** inside. Using these credentials we will access to a PhpMyAdmin panel, change the password of the **WordPress** administrator account and login as it. Inside the WordPress we will launch a **Reverse Shell** by modifying a php template. Then, a **password reuse** will give us access as the main user. Finally, using **sudo privileges** we will get a bash as root.

***

## Recognition

First of all, let’s check the connection between our machine and the target one:

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

TTL value (63) may indicate that the target machine is a Linux.

Let’s scan the TCP ports to see which ones are open:

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

**nmap** reported 4 open TCP ports: 21 (ftp), 22 (ssh), 80 (http) and 25565 (minecraft). And it found a closed one: 8192(sophos).

Let’s launch an intensive scan to check which services and which versions are running on those ports:

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

There is a ProFTPD 1.3.5a, OpenSSH 7.2p2, Apache httpd 2.4.18 (that tried to redirect us to blocky.htb) and a Minecraft Server 1.11.2 (apparently being hosted in the machine 127.0.1.1)

Let’s start in order.

First of all let’s see if we can access to FTP without giving credentials or using the anonymous login:

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

No, we can’t.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

After searching for known vulnerabilities of this service, it looks like is vulnerable to RCE! (CVE-2015-3306).

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Let’s check it later!

Let’s see the next service, OpenSSH 7.2p2.

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Acording to launchpad, the OS may be an **Ubuntu Xenial.**

Let’s see it there are known vulnerabilities for this version of OpenSSH:

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Yes, it looks like there is a known vuln that may allow us to **enumerate usernames**.

Next service, Apache httpd 2.4.18. It is redirecting us to blocky.htb, so let’s add it to our /etc/hosts file and we will look at the website in a second.

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Let’s enumerate the common files and directories first using nmap’s **http-enum** script:

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

So, it looks like the website is using **WordPress** version 4.8. The scan also reported some common files.

Let’s scan it with **whatweb** also:

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Aside from the WordPress, whatweb reported that the site is using **JQuery** 1.12.4. This is an outdated version that is vulnerable to **XSS**.

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

According to searchsploit, there are also known vulnerabilities for the version 4.8 of WordPress. The most interesting one is the second, because it will allow us to see private information:

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Ok, so, there is a bunch of vulnerabilities in this system. Let’s start looking at the website using the web browser:

<figure><img src="../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Nothing very interesting in the main page. Let’s look to the wiki directory that the nmap scan reported:

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Nothing I can use.

Let’s see if there are any hidden information we can leak using the WordPress vulnerability:

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Just a predefined sample page that may be hidden. I tried with different filters, but nothing seemed to work.

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

If we go to the login page and try to log in using random credentials, an error appears saying that the username is invalid. This could be useful if we have to check valid usernames in the future.

Ok, let’s go a step back and let’s try to use the FTP vulnerability:

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Apparently the php has been written, but if we try to access to the indicated URL it redirect us to the WordPress main page. Removing the domain blocky.htb from the /etc/hosts file doesn’t help.

Let’s enumerate directories:

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Mmm… the main post mentioned something that they are developing a plugin. And there is a directory called plugins. Let’s check it:

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

Interesting 2 jar files. Let’s download them and see its content:

After unziping the BlockyCore.jar file, I found the file BlockyCore.class.

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

By printing the readable strings of the binary file I can read what it looks like a sqlUser and its password.

Let’s save it just in case: `root:8YsqfCXXXXXXXXXXe22`

The other file seems to contain a real plugin called griefprevention.

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

Inside the **me** folder there is another folder called **ryanhamshire** it may be a username or a password. Let’s note it just in case.

The gobuster scan also reported a directory called phpmyadmin. Let’s check it and let’s try to login using the credentials we just found:

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

The credentials were valid. Nice.

<figure><img src="../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

Let’s check which users are registered in the wordpress!

<figure><img src="../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

The hash of the user password is: `$P$BiVXXXXXXXXXXXXXZI4Oq0/`. Meanwhile I’m trying to crack it using John, let’s change it with a password we know and try to log in:

To do it, I’m going to write a new password and select MD5, as it is the type of hash WordPress uses.

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

Then I clicked on the **Go** button and the changes are applied.

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

Now, let’s try to log in as Notch:

<figure><img src="../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

Yeah, we’re in as Notch.

Now that we are in, let’s check if there is any info we can use. If we don’t find nothing useful we can modify the current theme/plugins code to include php code that will give us a reverse shell!

There is nothing interesting, so let’s do the reverse shell thing.

I’m going to modify the 404 template of the current theme:

<figure><img src="../../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

Instead of the default code, I’ll replace it with the [php-reverse-shell code of PentestMonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell).

<figure><img src="../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

Then, I’m going to launch a nc listener in my machine and go to http://blocky.htb/wp-content/themes/twentyseventeen/404.php

<figure><img src="../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

That’s how we obtained the Reverse Shell.

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

The IP matches with the target machine one, so we are inside it.

<figure><img src="../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

The OS is an Ubuntu Xenial, as we guessed at the beggining.

Let’s check for the users registered in the system and have a bash as a shell:

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

**notch** and **root**.

Let’s see if we can pivot to notch user by using the same credentials of the phpmyadmin `notch:8YsXXXXXXXXXe22`

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

Yes, it worked. At the /home/notch folder we found the user flag.

## Privilege escalation

Let’s see if we have sudo permissions:

<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Ehm… We can run any command as root…

<figure><img src="../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

Done.

In comparison with other EASY ranked machine this one was easier in my opinion, but it was nice to complete another one all by myself :smile:
