# Sau

[**au**](https://app.hackthebox.com/machines/Sau) is an **EASY** machine from the **Hack The Box** platform. In this machine we will exploit 2 known issues of 2 different web tools that will give us access to the target machine. After that we will escalate our privileges by taking advantage of a binary that we will be able to execute with sudo permissions.

***

## Recognition

First of all, let’s test the connection with the target machine:

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

We sent 1 packet and received 1 packet. The connection with the target machine is ok.

The **ttl** value (63) may indicate that we are against a Linux machine.

Let’s scan all the target machine’s TCP ports to see which ones are open:

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

**Nmap** has reported 2 open ports (**22** and **55555**) and 2 ports that may be filtered (**80** and **8338**).

Let’s do an intensive scan of the open ones to try to get the name and version of the services running on those ports:

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

The scan reports that the service running on port 22 is OpenSSH 8.2p1

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

According to launchpad it may be an Ubuntu Focal.

On the other hand, the service running on the port 55555 couldn’t be identified by nmap, but by reading the responses it looks like HTTP.\
Let’s check with **whatweb** if there is a website in that port and if it is, what technologies are being used.

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

The whatweb tool reported some information, so it may be a website there. It’s using **HTML5** and **JQuery 3.2.1**. Jquery versions before 3.5.0 are vulnerable to **XSS**. The website is also running **Bootstrap 3.3.7** which has some [known vulnerabilities](https://security.snyk.io/package/npm/bootstrap-sass/3.3.7).

Let’s check it out using the web browser:\


<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Let’s check what happen if we click on **Create**

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Ok, let’s click on **Open Basket**

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

Ok, let’s go back to the main page:

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

Now, at the right of the page we have a link to our recently created basket.

At the bottom of the page we can see that the site is using something called **request-baskets 1.2.1**.

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

If we click on the link, it redirects us to a github repository. It’ll be useful in case we have to get into the code.

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

After checking the latest releases I have not found nothing particularly interesting, in the latest one (1.2.3) the JQuery has been updated to the latest version.

Let’s check if there are any known vulnerabilities for this version of request-baskets:

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

Bingo! Apparently there is a SSRF vulnerability in this particular version. Let’s look at the code:

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

i don’t fully understand the script, but as it is an SSRF vuln and it ask for the **URL** of the request-baskets and it also ask for a **TARGET** url I guess that it will allow us to make requests to services hosted in ports we don’t have access to.

Anyway, let’s search for the **CVE-2023-27163** to understand better how it works. In [this github repo](https://github.com/entr0pie/CVE-2023-27163) is explained how it works and how to use it.

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

## Exploitation

Let’s try it. If we check our first scan of TCP ports, there were 2 filtered ports that we can try to check now with this vuln.

Let’s start with the port 80:

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

If we now visit the created basket:

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

We can see a web page, powered by something called **Maltrail v0.53**. The 8338 ports seems to redirect us to the same page.

Let’s see what is this and if there are any known vulnerabilities for this version:

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

There is a vulnerability for this version that apparently leads to a RCE. Let’s check it:

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

In [this github repo](https://github.com/spookier/Maltrail-v0.53-Exploit) is better explained:

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

## Exploitation 2

Let’s try it:

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

We got a reverse shell! The IP machine matches with the target machine one, so we are inside!

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

We went to the home folder of the current user to see the user flag.

## Privilege Escalation

Let’s check which users are registered in the system and have a bash as a shell:

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

Only the current user and root. Let’s check now everything that can give us a clue.

**Environmental variables**:

<figure><img src="../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

**SUID binaries**:

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

**sudoers**:

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

Interesting, we can execute as sudo `/usr/bin/systemctl status trail.service`

According to [GTFOBins](https://gtfobins.github.io/gtfobins/systemctl/) we can try using this to escalate our privileges:

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Let’s try the option c first, since that seems to be the easiest one:

<figure><img src="../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

Woah, we escalated our privileges to root! Let’s go to see the root flag:

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

And machine done.

I have enjoyed this one, because it was quite simple and I was able to do it all without looking for help hahaha. I found interesting how the exploits work, it fascinates me that people find this kind of bugs, I would like to discover one someday!
