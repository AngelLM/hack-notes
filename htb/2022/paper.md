# Paper

**Date**: 21/05/2022

**Difficulty**: EASY

**CTF**: [https://app.hackthebox.com/machines/Paper](https://app.hackthebox.com/machines/Paper)

***

First things first. let’s test the connection with the target machine:

<figure><img src="../../.gitbook/assets/paper0 (1).png" alt=""><figcaption></figcaption></figure>

The ttl value of 63 may indicate that the target machine is Linux.

Let’s launch a nmap scan in order to discover the open tcp ports:

<figure><img src="../../.gitbook/assets/paper1.png" alt=""><figcaption></figcaption></figure>

There are 3 ports open: 22 (ssh), 80 (http), 443 (https).

<figure><img src="../../.gitbook/assets/paper2.png" alt=""><figcaption></figcaption></figure>

Let’s see what is hosted in the http and https ports:

<figure><img src="../../.gitbook/assets/paper3.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (13).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper5.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Seems to be the same page.

<figure><img src="../../.gitbook/assets/paper7.png" alt=""><figcaption></figcaption></figure>

Wappalizer confirms the versions of apache and openssl. I’m going to search if any of this services has a vulnerability I can use:

<figure><img src="../../.gitbook/assets/paper8.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper9.png" alt=""><figcaption></figcaption></figure>

Not apparently… Let’s enumerate the directories using wfuzz:

`wfuzz -c --hc 404,403 -L -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt http://10.10.11.143/FUZZ`

<figure><img src="../../.gitbook/assets/paper10.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (2) (1).png" alt=""><figcaption></figcaption></figure>

Looks like a standard page…

Ok, no clues. Let’s go back and see what we found so far…

Taking a look to the whatweb response, there is something that looks like a domain… `office.paper` let’s add it to the /etc/hosts file and take a look to it in the web browser:

<figure><img src="../../.gitbook/assets/imagen (3) (1).png" alt=""><figcaption></figcaption></figure>

Yeah, there is a website here!

<figure><img src="../../.gitbook/assets/imagen (4) (1).png" alt=""><figcaption></figcaption></figure>

This site is using Wordpress 5.2.3

Let’s take a look to the page content…

<figure><img src="../../.gitbook/assets/paper14.png" alt=""><figcaption></figcaption></figure>

The post says that the only user in the blog is `Prisonmike`, but another user (`nick`) replied telling him that he has secret information in the blog drafts. If we gain access to the administration panel we should take a look to the drafts.

There is nothing interesting in the other 2 post available, but we can find other 2 posts if we click on `Search` button:

<figure><img src="../../.gitbook/assets/paper15.png" alt=""><figcaption></figcaption></figure>

A simple test post and another one of Nick reminding him to not write secrets in the drafts.

We didn’t found anything that could be a password for Prisonmike user, so let’s try to login with default credentials:

<figure><img src="../../.gitbook/assets/paper16.png" alt=""><figcaption></figcaption></figure>

`admin` is not a valid user, but `prisonmike` is. But we still don’t know the password.

<figure><img src="../../.gitbook/assets/paper17.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper18.png" alt=""><figcaption></figcaption></figure>

Using searchsploit I found a exploit that seems capable of view unauthenticated posts…

<figure><img src="../../.gitbook/assets/paper19.png" alt=""><figcaption></figcaption></figure>

Let’s try it!

<figure><img src="../../.gitbook/assets/paper20.png" alt=""><figcaption></figcaption></figure>

So, yeah, we have access to the draft posts contents… There is one with a “secret” url that seems interesting… Let’s add `chat.office.paper` to /ect/hosts file and visit it with the web-browser

<figure><img src="../../.gitbook/assets/imagen (5) (1).png" alt=""><figcaption></figcaption></figure>

It is a register page, let’s register a new user:

<figure><img src="../../.gitbook/assets/imagen (6) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/imagen (7) (1).png" alt=""><figcaption></figcaption></figure>

Automatically I get invited to a chat:

<figure><img src="../../.gitbook/assets/imagen (8) (1).png" alt=""><figcaption></figcaption></figure>

Let’s take a look to the chat messages:

<figure><img src="../../.gitbook/assets/paper25.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper26.png" alt=""><figcaption></figcaption></figure>

So, let’s open a private chat with Recyclops and see if we can enumerate something:

<figure><img src="../../.gitbook/assets/paper27.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper28.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper29.png" alt=""><figcaption></figcaption></figure>

Let’s see if it’s vulnerable to path traversal:

<figure><img src="../../.gitbook/assets/paper30.png" alt=""><figcaption></figcaption></figure>

Yep, it is… and we should have access to user flag this way:

<figure><img src="../../.gitbook/assets/paper31.png" alt=""><figcaption></figcaption></figure>

Not that easy… yep, it is only readable by the owner… there will a ssh key?

<figure><img src="../../.gitbook/assets/paper32.png" alt=""><figcaption></figcaption></figure>

Nope… but the .hubot\_history sounds interesting:

<figure><img src="../../.gitbook/assets/paper33.png" alt=""><figcaption></figcaption></figure>

There is a connect command? I tried to use it, but it doesn’t seems to work.

<figure><img src="../../.gitbook/assets/paper34.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper35.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper36.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper37.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper38.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper39.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper40.png" alt=""><figcaption></figcaption></figure>

woah, we found credentials: `recyclops:Queenofblad3s!23`

Let’s see if we can login as recyclops in the chat:

<figure><img src="../../.gitbook/assets/paper41.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper42.png" alt=""><figcaption></figcaption></figure>

Nope, we can’t… Recyclops is a bot made by Dwight… Will him be reusing credentials? Let’s check it via ssh:

<figure><img src="../../.gitbook/assets/paper43.png" alt=""><figcaption></figcaption></figure>

Yeah!

<figure><img src="../../.gitbook/assets/paper44.png" alt=""><figcaption></figcaption></figure>

Escalation

i.sh

<figure><img src="../../.gitbook/assets/paper45.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper46.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper47.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/paper48.png" alt=""><figcaption></figcaption></figure>
