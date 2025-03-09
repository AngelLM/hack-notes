# Daily Bugle

**Date**: 04/04/2022

**Difficulty**: Hard

**CTF**: [https://tryhackme.com/room/dailybugle](https://tryhackme.com/room/dailybugle)

---

# Deploy

## Access the web server, who robbed the bank?

First of all, a quick scan:

<figure><img src="../../.gitbook/assets/dailybugle0.png" alt=""><figcaption></figcaption></figure>

Ping tell us that it will be a linux machine (ttl=63)

<figure><img src="../../.gitbook/assets/dailybugle1.png" alt=""><figcaption></figcaption></figure>

A quick nmap scan discover that ports 22, 80 and 3306 are open.

<figure><img src="../../.gitbook/assets/dailybugle2.png" alt=""><figcaption></figcaption></figure>

A more detailed nmap scan shows us the version of the services.

For the http service it also discovered that the site uses Joomla, and that robots.txt file exists and has several directories configured as disallowed entries, we may check them all later.

For now, let’s visit the page:

<figure><img src="../../.gitbook/assets/dailybugle3.png" alt=""><figcaption></figcaption></figure>

# Obtain user and root

## What is the Joomla version?

<aside>
💡 Instead of using SQLMap, why not use a python script!

</aside>

As I have never used SQLMap before, I’m going to do it with a python script first and then I’ll find how to do it with SQLMap:

### Python Script

Doing a bit of research I found [Juumla Github repository](https://github.com/oppsec/juumla). This tool is a python script that discovers the version of a Joomla site.

<figure><img src="../../.gitbook/assets/dailybugle4.png" alt=""><figcaption></figcaption></figure>

So let’s clone the repository in our local machine and follow the instructions:

<figure><img src="../../.gitbook/assets/dailybugle5.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/dailybugle6.png" alt=""><figcaption></figcaption></figure>

Pretty fast!

### SQLMap

After some research I didn’t find a way to do it with SQLMap... Maybe that phrase was referring to the next task?

### Other

There is other way to obtain the Joomla version according to [this site](https://www.itoctopus.com/how-to-quickly-know-the-version-of-any-joomla-website). As it says, is as easy as navigate to this url: [`http://10.10.121.47/administrator/manifests/files/joomla.xml`](http://10.10.121.47/administrator/manifests/files/joomla.xml) and a config file appears where you can easy see the Joomla version:

<figure><img src="../../.gitbook/assets/dailybugle7.png" alt=""><figcaption></figcaption></figure>

## What is Jonah's cracked password?

Looking for any exploit for this Joomla version, the results are aligned, there is a SQL Injection vulnerability in this version we can exploit.

<figure><img src="../../.gitbook/assets/dailybugle8.png" alt=""><figcaption></figcaption></figure>

So, let’s try with the [python script of stefanlucas](https://github.com/stefanlucas/Exploit-Joomla) first:

<figure><img src="../../.gitbook/assets/dailybugle9.png" alt=""><figcaption></figcaption></figure>

Nice, now we have to crack the password. To do it we should know the hash format... Let’s discover it:

<figure><img src="../../.gitbook/assets/dailybugle10.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/dailybugle11.png" alt=""><figcaption></figcaption></figure>

A user in hashcat forum says is bcrypt... It has an avatar of a cat, I should trust on him.

So, let’s try to crack it using John the Ripper, fasttrack wordlist and bcrypt hash format:

<figure><img src="../../.gitbook/assets/dailybugle12.png" alt=""><figcaption></figcaption></figure>

After 5 minutes we get a cracked password!

## What is the user flag?

In the nmap scan we did before, it discovered robots.txt file, which had some directories we have taken note. Let’s try to visit the ones that look like an administrator page:

<figure><img src="../../.gitbook/assets/dailybugle13.png" alt=""><figcaption></figcaption></figure>

At /administrator we find this login. Let’s try to login as jonah:

<figure><img src="../../.gitbook/assets/dailybugle14.png" alt=""><figcaption></figcaption></figure>

We are in. Let’s look for the flag...

After some navigation it doesn’t look like there is a flag inside joomla itself. I think I can upload a php reverse shell and try to catch it on my local machine:

<figure><img src="../../.gitbook/assets/dailybugle15.png" alt=""><figcaption></figcaption></figure>

It doesn’t work. Apparently I can’t upload .php files there.

<figure><img src="../../.gitbook/assets/dailybugle16.png" alt=""><figcaption></figcaption></figure>

But... I can modify this hahaha

<figure><img src="../../.gitbook/assets/dailybugle17.png" alt=""><figcaption></figcaption></figure>

Let’s try again:

<figure><img src="../../.gitbook/assets/dailybugle18.png" alt=""><figcaption></figcaption></figure>

Nah, there is no way.

Just for the sake of trying, let’s see if we can login via SSH using this credentials:

<figure><img src="../../.gitbook/assets/dailybugle19.png" alt=""><figcaption></figcaption></figure>

Nope, okay. Let’s try again to open a reverse shell. I’m going to search how to upload php files in Joomla.

After some searching, apparently there is no way to upload php files, buuuut, there are php files you can edit: the templates. So let’s try to use them to inject our code there:

<figure><img src="../../.gitbook/assets/dailybugle20.png" alt=""><figcaption></figcaption></figure>

I’ll backup all the index.php code to undo it if neccesary, and replace all the cod with the php-reverse-shell.php one:

<figure><img src="../../.gitbook/assets/dailybugle21.png" alt=""><figcaption></figcaption></figure>

It will work if I click on preview? Nope. Let’s save it.

<figure><img src="../../.gitbook/assets/dailybugle22.png" alt=""><figcaption></figcaption></figure>

And now let’s click on preview, And Instantly we get the reverse shell!

<figure><img src="../../.gitbook/assets/dailybugle23.png" alt=""><figcaption></figcaption></figure>

I’m going to stabilize the shell:

<figure><img src="../../.gitbook/assets/dailybugle24.png" alt=""><figcaption></figcaption></figure>

Now, let’s look for the flag:

<figure><img src="../../.gitbook/assets/dailybugle25.png" alt=""><figcaption></figcaption></figure>

hmmm... we have not access to jjameson folder. Let’s see what can I do with this user:

<figure><img src="../../.gitbook/assets/dailybugle26.png" alt=""><figcaption></figcaption></figure>

I have no permissions to read /etc/shadow and there is nothing being executed on crontab.

<figure><img src="../../.gitbook/assets/dailybugle27.png" alt=""><figcaption></figcaption></figure>

No interesting SUID binaries, no interesting capabilities binaries, no NFS...

Ok, we know the other user is jjameson. Maybe we can start an hydra attack? I tried for 30 min with no success, so I guess is not the correct way.

Let’s see if there is any vulnerability we can exploit in the apache version 2.4.6. After some research I didn’t find nothing exploitable for privesc.

What about the OpenSSH version 7.4? Nothing neither...

And Maria DB? Nothing? uhm...

Ok, let’s try another thing. Let’s investigate if there is something we can use in the /var/www/html folder which contains the files hosted in the httpd service:

<figure><img src="../../.gitbook/assets/dailybugle28.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/dailybugle29.png" alt=""><figcaption></figcaption></figure>

This is interesting, there are credentials of a user “root”. This credentials seems to be for a database service. Maria DB maybe? Let’s check:

`mysql -h <hostname> -u <username> -p <databasename>`

<figure><img src="../../.gitbook/assets/dailybugle30.png" alt=""><figcaption></figcaption></figure>

Looks like we cannot access to the database from our local machine. Let’s try with the target machine:

<figure><img src="../../.gitbook/assets/dailybugle31.png" alt=""><figcaption></figcaption></figure>

Yeah, we can from target machine. Let’s see what’s inside this DB:

<figure><img src="../../.gitbook/assets/dailybugle32.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/dailybugle33.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/dailybugle34.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/dailybugle35.png" alt=""><figcaption></figcaption></figure>

Mmmh... we got this before when we exploited the joomla vulnerability. Maybe we are in the incorrect DATABASE.

<figure><img src="../../.gitbook/assets/dailybugle36.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/dailybugle37.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/dailybugle38.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/dailybugle39.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/dailybugle40.png" alt=""><figcaption></figcaption></figure>

I have found nothing interesting through the database. Let’s try this password in the root user of the target machine just in case:

<figure><img src="../../.gitbook/assets/dailybugle41.png" alt=""><figcaption></figcaption></figure>

Nope. Let’s try as the password of the jjameson user, just to discard everything:

<figure><img src="../../.gitbook/assets/dailybugle42.png" alt=""><figcaption></figcaption></figure>

OMG... it is. Let’s go for the user flag:

<figure><img src="../../.gitbook/assets/dailybugle43.png" alt=""><figcaption></figcaption></figure>

## What is the root flag?

Ok, root flag would be probably located into /root folder. This user has no access there, so it’s time for privesc.

Let’s see if we can execute anything using sudo:

<figure><img src="../../.gitbook/assets/dailybugle44.png" alt=""><figcaption></figcaption></figure>

We can use yum as sudo... Let’s check it at GTFOBins:

<figure><img src="../../.gitbook/assets/dailybugle45.png" alt=""><figcaption></figcaption></figure>

We can escalate privileges using yum!

<figure><img src="../../.gitbook/assets/dailybugle46.png" alt=""><figcaption></figcaption></figure>

First way is not possible, as the target system doesn’t recognize fpm as a command. Let’s try with the second option:

<figure><img src="../../.gitbook/assets/dailybugle47.png" alt=""><figcaption></figcaption></figure>

This second one worked like a charm (even if I don't fully understand how it works). Now let’s go for the root flag!

<figure><img src="../../.gitbook/assets/dailybugle48.png" alt=""><figcaption></figcaption></figure>

And challenge finished!