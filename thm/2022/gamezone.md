---
description: '#CMS, #cracking, #linux, #sqli, #SSH tunneling'
---

# GameZone

**Date**: 02/04/2022

**Difficulty**: Medium

**CTF**: [https://tryhackme.com/room/gamezone](https://tryhackme.com/room/gamezone)

***

This room will cover SQLi (exploiting this vulnerability manually and via SQLMap), cracking a users hashed password, using SSH tunnels to reveal a hidden service and using a metasploit payload to gain root privileges.

## Deploy the vulnerable machine <a href="#user-content-deploy-the-vulnerable-machine" id="user-content-deploy-the-vulnerable-machine"></a>

### Deploy the machine and access its web server <a href="#user-content-deploy-the-machine-and-access-its-web-server" id="user-content-deploy-the-machine-and-access-its-web-server"></a>

Once deployed, let’s do a quick scan with nmap:

<figure><img src="../../.gitbook/assets/Untitled (1).png" alt=""><figcaption></figcaption></figure>

And let’s see the details of each opened port:

<figure><img src="../../.gitbook/assets/Untitled 1 (1).png" alt=""><figcaption></figcaption></figure>

Let’s visit the webserver:

<figure><img src="../../.gitbook/assets/Untitled 2 (1).png" alt=""><figcaption></figcaption></figure>

### [What is the name](https://github.com/AngelLM/CTF-Writeups/blob/main/TryHackMe/2022_04_02-GameZone/readme.md#what-is-the-name-of-the-large-cartoon-avatar-holding-a-sniper-on-the-forum) [of the large cartoon avatar holding a sniper on the forum?](https://github.com/AngelLM/CTF-Writeups/blob/main/TryHackMe/2022_04_02-GameZone/readme.md#what-is-the-name-of-the-large-cartoon-avatar-holding-a-sniper-on-the-forum) <a href="#user-content-what-is-the-name-of-the-large-cartoon-avatar-holding-a-sniper-on-the-forum" id="user-content-what-is-the-name-of-the-large-cartoon-avatar-holding-a-sniper-on-the-forum"></a>

I recognized the game (Hitman) but I have no idea what’s the name of the character:

<figure><img src="../../.gitbook/assets/Untitled 3 (1).png" alt=""><figcaption></figcaption></figure>

## Obtain access via SQLi <a href="#user-content-obtain-access-via-sqli" id="user-content-obtain-access-via-sqli"></a>

In this task you will understand more about SQL (structured query language) and how you can potentially manipulate queries to communicate with the database.

SQL is a standard language for storing, editing and retrieving data in databases. A query can look like so:

`SELECT * FROM users WHERE username = :username AND password := password`

In our GameZone machine, when you attempt to login, it will take your inputted values from your username and password, then insert them directly into the query above. If the query finds data, you’ll be allowed to login otherwise it will display an error message.

Here is a potential place of vulnerability, as you can input your username as another SQL query. This will take the query write, place and execute it.

Lets use what we’ve learnt above, to manipulate the query and login without any legitimate credentials.

If we have our username as admin and our password as: **`' or 1=1 -- -`** it will insert this into the query and authenticate our session.

The SQL query that now gets executed on the web server is as follows:

`SELECT * FROM users WHERE username = admin AND password := ' or 1=1 -- -`

The extra SQL we inputted as our password has changed the above query to break the initial query and proceed (with the admin user) if 1==1, then comment the rest of the query to stop it breaking.

GameZone doesn’t have an admin user in the database, however you can still login without knowing any credentials using the inputted password data we used in the previous question.

Use `' or 1=1 -- -` as your username and leave the password blank.

### When you’ve logged in, what page do you get redirected to? <a href="#user-content-when-youve-logged-in-what-page-do-you-get-redirected-to" id="user-content-when-youve-logged-in-what-page-do-you-get-redirected-to"></a>

Let’s do it and write `' or 1=1 -- -` as the username and try to log in:

<figure><img src="../../.gitbook/assets/Untitled 4 (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Untitled 5 (1).png" alt=""><figcaption></figcaption></figure>

We log in and it redirects us to /portal.php page.

## Using SQLMap <a href="#user-content-using-sqlmap" id="user-content-using-sqlmap"></a>

SQLMap is a popular open-source, automatic SQL injection and database takeover tool. This comes pre-installed on all version of [Kali Linux](https://tryhackme.com/rooms/kali) or can be manually downloaded and installed [here](https://github.com/sqlmapproject/sqlmap).

There are many different types of SQL injection (boolean/time based, etc..) and SQLMap automates the whole process trying different techniques.

We’re going to use SQLMap to dump the entire database for GameZone.

Using the page we logged into earlier, we’re going point SQLMap to the game review search feature.

First we need to intercept a request made to the search feature using [BurpSuite](https://tryhackme.com/room/learnburp).

<figure><img src="../../.gitbook/assets/Untitled 6 (1).png" alt=""><figcaption></figcaption></figure>

Save this request into a text file.

<figure><img src="../../.gitbook/assets/Untitled 7 (1).png" alt=""><figcaption></figcaption></figure>

### In the users table, what is the hashed password? What was the username associated with the hashed password? <a href="#user-content-in-the-users-table-what-is-the-hashed-password-what-was-the-username-associated-with-th" id="user-content-in-the-users-table-what-is-the-hashed-password-what-was-the-username-associated-with-th"></a>

We can then pass this into SQLMap to use our authenticated user session. `sqlmap -r <requestText> --dbms=mysql --dump`

SQLMap will now try different methods and identify the one thats vulnerable. Eventually, it will output the database.

<figure><img src="../../.gitbook/assets/Untitled 8.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Untitled 9 (1).png" alt=""><figcaption></figcaption></figure>

### What was the other table name? <a href="#user-content-what-was-the-other-table-name" id="user-content-what-was-the-other-table-name"></a>

<figure><img src="../../.gitbook/assets/Untitled 10 (1).png" alt=""><figcaption></figcaption></figure>

## Cracking a password with JohnTheRipper <a href="#user-content-cracking-a-password-with-johntheripper" id="user-content-cracking-a-password-with-johntheripper"></a>

John the Ripper (JTR) is a fast, free and open-source password cracker. This is also pre-installed on all Kali Linux machines.

We will use this program to crack the hash we obtained earlier. JohnTheRipper is 15 years old and other programs such as HashCat are one of several other cracking programs out there.

This program works by taking a wordlist, hashing it with the specified algorithm and then comparing it to your hashed password. If both hashed passwords are the same, it means it has found it. You cannot reverse a hash, so it needs to be done by comparing hashes.

Once you have JohnTheRipper installed you can run it against your hash using the following arguments:

`john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-SHA256`

`hash.txt` – contains a list of your hashes (in your case its just 1 hash)

`--wordlist` – is the wordlist you’re using to find the dehashed value

`--format` – is the hashing algorithm used. In our case its hashed using SHA256.

### What is the de-hashed password? <a href="#user-content-what-is-the-de-hashed-password" id="user-content-what-is-the-de-hashed-password"></a>

<figure><img src="../../.gitbook/assets/Untitled 11 (1).png" alt=""><figcaption></figcaption></figure>

### Now you have a password and username. Try SSH’ing onto the machine. What is the user flag? <a href="#user-content-now-you-have-a-password-and-username-try-sshing-onto-the-machine-what-is-the-user-flag" id="user-content-now-you-have-a-password-and-username-try-sshing-onto-the-machine-what-is-the-user-flag"></a>

<figure><img src="../../.gitbook/assets/Untitled 12 (1).png" alt=""><figcaption></figcaption></figure>

## Exposing services with reverse SSH tunnels <a href="#user-content-exposing-services-with-reverse-ssh-tunnels" id="user-content-exposing-services-with-reverse-ssh-tunnels"></a>

Reverse SSH port forwarding specifies that the given port on the remote server host is to be forwarded to the given host and port on the local side.

* **L** is a local tunnel (YOU <– CLIENT). If a site was blocked, you can forward the traffic to a server you own and view it. For example, if imgur was blocked at work, you can do **ssh -L 9000:imgur.com:80** [**user@example.com**](mailto:user@example.com)**.** Going to localhost:9000 on your machine, will load imgur traffic using your other server.
* **R** is a remote tunnel (YOU –> CLIENT). You forward your traffic to the other server for others to view. Similar to the example above, but in reverse.

We will use a tool called **`ss`** to investigate sockets running on a host.

If we run `ss -tulpn` it will tell us what socket connections are running

| Argument | Description                        |
| -------- | ---------------------------------- |
| -t       | Display TCP sockets                |
| -u       | Display UDP sockets                |
| -l       | Displays only listening sockets    |
| -p       | Shows the process using the socket |
| -n       | Doesn’t resolve service names      |

### How many TCP sockets are running? <a href="#user-content-how-manytcpsockets-are-running" id="user-content-how-manytcpsockets-are-running"></a>

<figure><img src="../../.gitbook/assets/Untitled 13 (1).png" alt=""><figcaption></figcaption></figure>

We can see that a service running on port 10000 is blocked via a firewall rule from the outside (we can see this from the IPtable list). However, Using an SSH Tunnel we can expose the port to us (locally)!

From our local machine, run `ssh -L 10000:localhost:10000 <username>@<ip>`

<figure><img src="../../.gitbook/assets/Untitled 14 (1).png" alt=""><figcaption></figcaption></figure>

Once complete, in your browser type “localhost:10000” and you can access the newly-exposed webserver.

<figure><img src="../../.gitbook/assets/Untitled 15 (1).png" alt=""><figcaption></figcaption></figure>

### What is the name of the exposed CMS? <a href="#user-content-what-is-the-name-of-the-exposed-cms" id="user-content-what-is-the-name-of-the-exposed-cms"></a>

I guess it’s Webmin

### What is the CMS version? <a href="#user-content-what-is-thecmsversion" id="user-content-what-is-thecmsversion"></a>

There is nothing about the version written in the source code of the login page, so I’m going to try to login using the credentials we have.

<figure><img src="../../.gitbook/assets/Untitled 16 (1).png" alt=""><figcaption></figcaption></figure>

The credentials are valid and we enter in the webmin landpage, where we can see the CMS version.

## Privilege Escalation with Metasploit <a href="#user-content-privilege-escalation-with-metasploit" id="user-content-privilege-escalation-with-metasploit"></a>

Using the CMS dashboard version, use Metasploit to find a payload to execute against the machine.

### What is the root flag? <a href="#user-content-what-is-the-root-flag" id="user-content-what-is-the-root-flag"></a>

First of all, lets open msfconsole:

<figure><img src="../../.gitbook/assets/Untitled 17 (1).png" alt=""><figcaption></figcaption></figure>

Now, let’s search for exploits for webmin 1.58:

<figure><img src="../../.gitbook/assets/Untitled 18 (1).png" alt=""><figcaption></figcaption></figure>

There are two. The second one says will allow us to download files from the target, cool! Let’s use it:

<figure><img src="../../.gitbook/assets/Untitled 19 (1).png" alt=""><figcaption></figcaption></figure>

This way we acomplished to download the root.txt containing the root flag!
