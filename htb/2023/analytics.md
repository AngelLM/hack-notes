---
description: '#docker, #linux, #overlayfs'
---

# Analytics

[**Analytics**](https://app.hackthebox.com/machines/Analytics) is an **EASY** machine from the Hack The Box platform. In it we will exploit an **RCE** thanks to an outdated version of a web tool. We will also perform a **Docker breakout**, to finally obtain root permissions thanks to the exploitation of a vulnerability in the **kernel version**.

***

### Enumeration <a href="#user-content-enumeration" id="user-content-enumeration"></a>

Let’s start by scanning the open TCP ports of the target machine:

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

**nmap** reported tcp ports 22(ssh) and 80(http) to be open. Let’s scan them further:

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

**nmap** exhaustive scan reported that the service running on port 22 is OpenSSH 8.9p1. According to lauchpad, the target machine OS may be Ubuntu Jammy:

<figure><img src="../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

Regarding the http service on port 80, the service running is **nginx 1.18.0**. The scan also reported that a redirection may be being applied to http://analytical.htb, so the server may be applying virtual hosting. Before adding this domain to the **/etc/hosts** file, let’s check quickly the petition to the website using Burpsuite:

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

As we can expect, the website is applying the redirection, and there is no information here we can use. So, let’s add the domain to **/etc/hosts** file:

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

The **nmap** script **http-enum** didn’t find any common file in the server.

Let’s see what technologies are being used in the website apart from nginx:

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

The **whatweb** tool reports 2 email addresses (demo@analytical.com and due@analytical.com). It also reports that the website is using JQuery v3.0.0. This version of JQuery is outdated and is vulnerable to XSS.

Let’s see how this page looks using the browser:

<figure><img src="../../.gitbook/assets/image (11) (1).png" alt=""><figcaption></figcaption></figure>

There is a section called “Our Team” that shows photos, names and positions of workers. Let’s write down this data that may be useful to check usernames.

<figure><img src="../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

```
Jonny Smith - Chief Data Officer
Alex Kirigo - Data Engineer
Daniel Walker - Data Analyst
```

There is a contact form at the bottom on the page that doesn’t seem to work.

By hovering the top menu “Login” link we can see that it will redirect us to http://data.analytical.htb

<figure><img src="../../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

Let’s add this subdomain in the **/etc/hosts** file.

<figure><img src="../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

Now, let’s click on the link:

<figure><img src="../../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>

It redirects us to a website where there is a service called Metabase. Let’s search what is this:

<figure><img src="../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

## Exploitation <a href="#user-content-exploitation" id="user-content-exploitation"></a>

After some research, I found this interesting blog entry at MetaBase’s official webpage:

https://www.metabase.com/blog/security-incident-summary\\

In this blog is explained that there were some programming errors that made the application vulnerable in older versions of it.

<figure><img src="../../.gitbook/assets/image (17) (1).png" alt=""><figcaption></figcaption></figure>

Checking for the setup token in the website of the target machine I found it:

<figure><img src="../../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

So, maybe the version of MetaBase is outdated and it is vulnerable.

I found this script written in Python that automates the exploitation process: [https://github.com/robotmikhro/CVE-2023-38646](https://github.com/robotmikhro/CVE-2023-38646)

<figure><img src="../../.gitbook/assets/image (21) (1).png" alt=""><figcaption></figcaption></figure>

After exploiting it, we gained a reverse shell!

<figure><img src="../../.gitbook/assets/image (22) (1).png" alt=""><figcaption></figcaption></figure>

But it seems that we are inside a docker container. Let’s see how can we escape from it.

## Docker breakout <a href="#user-content-docker-breakout" id="user-content-docker-breakout"></a>

If we check the environmental variables with <kbd>env</kbd>:

<figure><img src="../../.gitbook/assets/image (23) (1).png" alt=""><figcaption></figcaption></figure>

We can find the credentials `metalytics:AnXXXXXXXXXX223#`.

Let’s try to check if they are valid to connect to the target machine via ssh:

Yes, the credentials are valid.

<figure><img src="../../.gitbook/assets/image (24) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (25) (1).png" alt=""><figcaption></figcaption></figure>

And that’s how we got the user flag

## Privilege Escalation <a href="#user-content-privilege-escalation" id="user-content-privilege-escalation"></a>

Let’s see the version of the Ubuntu and the kernel:

<figure><img src="../../.gitbook/assets/image (26) (1).png" alt=""><figcaption></figcaption></figure>

It’s an Ubuntu jammy, as we guessed in the enumeration phase. The Linux kernel is 6.2.0.

If we search for vulnerabilities of this kernel, we find this page: [https://www.wiz.io/blog/ubuntu-overlayfs-vulnerability](https://www.wiz.io/blog/ubuntu-overlayfs-vulnerability)

In that article is explained that multiple versions of the linux kernel have a vulnerability related to the OverlayFS module that can be used to perform a Privilege Escalation. Apparently a similar vulnerability was detected and fixed back in 2021, but it happened again.

<figure><img src="../../.gitbook/assets/image (27) (1).png" alt=""><figcaption></figcaption></figure>

According to the article, the version of the kernel that the target Ubuntu is using is vulnerable.

<figure><img src="../../.gitbook/assets/image (28) (1).png" alt=""><figcaption></figcaption></figure>

The article also says that the old exploits still work for this vulnerability, so I’m going to use this one I found:

https://github.com/briskets/CVE-2021-3493

So, I downloaded the .c file and compiled it in my machine. Then I shared it with the target machine using an http server:

<figure><img src="../../.gitbook/assets/image (29) (1).png" alt=""><figcaption></figcaption></figure>

Then, from the target machine, I downloaded the compiled exploit, gave it execution permissions and executed it.

<figure><img src="../../.gitbook/assets/image (30) (1).png" alt=""><figcaption></figcaption></figure>

And this way I escalated privileges to root easily and read the root flag.

## New things learned <a href="#user-content-new-things-learned" id="user-content-new-things-learned"></a>

* The **environmental variables** should be checked every time.
* It’s important to check the OS and kernel version and look for vulnerabilities.
