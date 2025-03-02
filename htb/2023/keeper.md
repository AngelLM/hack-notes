---
description: '#default credentials, #exposed credentials, #keepass, #linux'
---

# Keeper

[Keeper](https://app.hackthebox.com/machines/Keeper) is an **EASY** machine from the Hack The Box platform. On this machine we will access a web tool control panel using the default credentials. Once inside, we will find credentials that will allow us to connect to the victim machine using SSH. Finally, we will take advantage of a memory dump of Keepass to obtain the Keepass master password, thus revealing an RSA-Key that we will use to connect as the root user.

***

## Enumeration <a href="#user-content-enumeration" id="user-content-enumeration"></a>

First of all let’s check with TCP ports are open in the target machine using **nmap**:

`nmap -p- --open -sS --min-rate 5000 -n -Pn -vvv 10.10.11.227 -oG allPorts`

<figure><img src="../../.gitbook/assets/Pasted image 20231013184126.png" alt=""><figcaption></figcaption></figure>

There are 2 TCP ports open: ssh (22) and http (80).

Let’s do an exhaustive scan on this ports using **nmap** again:

`nmap -p22,80 -sCV 10.10.11.227 -oN targeted`

<figure><img src="../../.gitbook/assets/Pasted image 20231013184329.png" alt=""><figcaption></figcaption></figure>

This scan reports the version of the OpenSSH service. If we search it in launchpad, the target machine appears to be an Ubuntu Jammy:

<figure><img src="../../.gitbook/assets/Pasted image 20231013184544.png" alt=""><figcaption></figcaption></figure>

On the other hand, the http service is a **nginx 1.18.0**.

`nmap --script=http-enum -p80 10.10.11.227 -oN webContent`

Let’s run the nmap http-enum script to do a quick enumeration of common files and directories:

<figure><img src="../../.gitbook/assets/Pasted image 20231013184833.png" alt=""><figcaption></figcaption></figure>

Nothing found, let’s use **whatweb** to obtain more info about the applications and tools this website is using:

`whatweb http://10.10.11.227`

<figure><img src="../../.gitbook/assets/Pasted image 20231013184955.png" alt=""><figcaption></figcaption></figure>

Nothing appart from the nginx version.

Let’s take a look to the website using the browser:

<figure><img src="../../.gitbook/assets/Pasted image 20231013185128.png" alt=""><figcaption></figcaption></figure>

I see, the server may be using virtual hosting, so we need to add the domain `keeper.htb` and `tickets.keeper.htb` subdomain to the `/etc/hosts` file:

<figure><img src="../../.gitbook/assets/Pasted image 20231013185313.png" alt=""><figcaption></figcaption></figure>

After doing it, the keeper.htb page shows the same message, so let’s check the tickets.keeper.htb subdomain:

<figure><img src="../../.gitbook/assets/imagen (2) (1).png" alt=""><figcaption></figcaption></figure>

Apparently the site is using an application called **Request Tracker** made by **BEST PRACTICAL** which is a real product.

A quick search shows that the default user for this application is **root** and its password is **password**. Let’s check if it works:

<figure><img src="../../.gitbook/assets/imagen (3) (1).png" alt=""><figcaption></figcaption></figure>

Yes, it worked.



We can see info about users at Admin>Users menu.

There is a user called **lnorgaard**

When we click in it’s name, more info is displayed about this user. It says that the initial password of this user has been set to `WXXXXXXX3!`

At User Summary we can see that this user has requested a ticket about “Issue with Keepass Client on Windows”

<figure><img src="../../.gitbook/assets/Pasted image 20231013195508.png" alt=""><figcaption></figcaption></figure>

The history of the ticket shows a conversation between the **root** user and **Inorgaard**. If we gain access as Inorgaard we should take a look to the crash dump file he says he saved into his home folder.

Let’s try to log via ssh as the user lnorgaard.

<figure><img src="../../.gitbook/assets/Pasted image 20231013200857.png" alt=""><figcaption></figcaption></figure>

Woah, we’re in.

<figure><img src="../../.gitbook/assets/Pasted image 20231013202858.png" alt=""><figcaption></figcaption></figure>

Inside the user folder we found the user flag and some interesting files mentioned previously in the ticket conversation.

## Privilege Escalation

Lets download them:

<figure><img src="../../.gitbook/assets/Pasted image 20231013203630.png" alt=""><figcaption></figcaption></figure>

After decompressing the ZIP file it appears that the content is the same we downloaded.

<figure><img src="../../.gitbook/assets/Pasted image 20231013203816.png" alt=""><figcaption></figcaption></figure>

So, we have a memory dump of KeePass (dmp file) and a KeePass database (kdbx file).

KeePass is _a free open source password manager_. Passwords can be stored in an encrypted database, which can be unlocked with one master key. Old versions had a vulnerability that allowed to extract most of the characters of the master key from a memory dump.

[KeePwn](https://github.com/Orange-Cyberdefense/KeePwn) is a tool that automatizes the extraction of the characters of the master key from the memory dump.

<figure><img src="../../.gitbook/assets/Pasted image 20231013205644.png" alt=""><figcaption></figcaption></figure>

The tool successfully found the master password! `rødgrød med fløde`

As we have the master password, we can see the content of the KeePass database we downloaded. To do so, we need a client for KeePass but I found this WebClient that works with KeePass files ([https://app.keeweb.info/](https://app.keeweb.info/)):

<figure><img src="../../.gitbook/assets/imagen (1) (1).png" alt=""><figcaption></figcaption></figure>

So, here we have a rsa-key for the user **root**.

And a password.

According to [this website](https://www.baeldung.com/linux/ssh-key-types-convert-ppk) this is a Putty ID-RSA key, and we cannot use it to access via OpenSSH, but there is a way to convert it in order to use it with OpenSSH:

First step is to create a ppk file with the Putty RSA key content:

<figure><img src="../../.gitbook/assets/Pasted image 20231013211803.png" alt=""><figcaption></figcaption></figure>

Next step is convert it by using this command: `puttygen id_rsa.ppk -O private-openssh -o id_rsa.pub`

<figure><img src="../../.gitbook/assets/Pasted image 20231013212236.png" alt=""><figcaption></figcaption></figure>

The private key is extracted. If we also wanted to extract the public key, this is the command we should use: `puttygen id_rsa.ppk -O public-openssh -o id_rsa.pub`

Now, with the private key, let’s try to connect via OpenSSH to the target machine as the **root** user:

<figure><img src="../../.gitbook/assets/Pasted image 20231013212632.png" alt=""><figcaption></figcaption></figure>

We gained a root shell and found the root flag.

## New things learned <a href="#user-content-new-things-learned" id="user-content-new-things-learned"></a>

* How to extract the master password from **KeePass** dump.
* Convert a Putty ID\_RSA key into an OpenSSH usable one.
