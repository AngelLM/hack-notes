# Forest

**Date**: 30/06/2022

**Difficulty**: Easy

**CTF**: [https://app.hackthebox.com/machines/Forest](https://app.hackthebox.com/machines/Forest)

---

Let’s start testing the connection with the target machine by sending a ping:

<figure><img src="../../.gitbook/assets/forest0.png" alt=""><figcaption></figcaption></figure>

The ttl confirms that we are against a Windows Machine. Let’s move to the nmap scan to see if there are any TCP port open:

<figure><img src="../../.gitbook/assets/forest1.png" alt=""><figcaption></figcaption></figure>

There are many ports open! Let’s do a detailed scan to these ports:

<figure><img src="../../.gitbook/assets/forest2.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/forest3.png" alt=""><figcaption></figcaption></figure>

Let’s start with the SMB (port 445). I’ll use crackmapexec to gather more info:

<figure><img src="../../.gitbook/assets/forest4.png" alt=""><figcaption></figcaption></figure>

Let’s see if we can see something inside the SMB without credentials:

<figure><img src="../../.gitbook/assets/forest5.png" alt=""><figcaption></figcaption></figure>

Not with smbmap, let’s try it with smbclient

<figure><img src="../../.gitbook/assets/forest6.png" alt=""><figcaption></figcaption></figure>

Ok, apparently we cannot access to the SMB… let’s try to find some usernames using kerbrute:

<figure><img src="../../.gitbook/assets/forest7.png" alt=""><figcaption></figcaption></figure>

We found some valid usernames! 

Let’s create a list of valid users:

<figure><img src="../../.gitbook/assets/forest8.png" alt=""><figcaption></figcaption></figure>

Any of them would be AS-Rep Roastable?

<figure><img src="../../.gitbook/assets/forest9.png" alt=""><figcaption></figcaption></figure>

None of the discovered user has the `UF_DONT_REQUIRE_PREAUTH` set, so no AS-Rrep Roast available.

Maybe is not the best idea, but let’s try to obtain a password of a discovered user using Kerbrute:

```bash
#!/bin/bash

File="validusers"
Lines=$(cat $File)
for Line in $Lines
do
	/opt/kerbrute/kerbrute bruteuser --dc 10.10.10.161 -d htb.local -t 200 /usr/share/seclists/Passwords/xato-net-10-million-passwords-10000.txt $Line
done
```

or do it with a one liner:

`cat validusers | while read LINE; do /opt/kerbrute/kerbrute bruteuser --dc 10.10.10.161 -d htb.local -t 200 /usr/share/seclists/Passwords/xato-net-10-million-passwords-10000.txt $LINE; done`

<figure><img src="../../.gitbook/assets/forest10.png" alt=""><figcaption></figcaption></figure>

But nah, no password has been discovered using this usernames and the password dictionary…

Let’s take a look to the ldap.

First of all, let’s see if it allows anonymous binds. To do so I can use `ldapsearch` tool:

`ldapsearch -H ldap://10.10.10.161:389 -x -b "dc=htb,dc=local”`

<figure><img src="../../.gitbook/assets/forest11.png" alt=""><figcaption></figcaption></figure>

The -x flag is used to specify anonymous authentication, while the -b flag denotes the base dn to start from. We were able to query the domain without credentials, which means null bind is enabled.
Now we can use `windapsearch` to obtain more info from the domain:

`/home/angellm/repos/windapsearch/windapsearch.py -d htb.local --dc-ip 10.10.10.161 -U`

- `-U` : Enumerate all users, i.e. objects with objectCategory set to user.

<figure><img src="../../.gitbook/assets/forest12.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/forest13.png" alt=""><figcaption></figcaption></figure>

These users are the ones that we previously had… let’s try to obtain even more info using the flag `--custom "objectClass=*"` in order to obtain all the objects in the domain

<figure><img src="../../.gitbook/assets/forest14.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/forest15.png" alt=""><figcaption></figcaption></figure>

The object `svc-alfresco` catches my attention. Let’s google it:

[Set up authentication and sync](https://docs.alfresco.com/content-services/7.0/admin/auth-sync/)

<figure><img src="../../.gitbook/assets/forest16.png" alt=""><figcaption></figcaption></figure>

So… this account seems to not require Kerberos preauthentication so… maybe we can get a valid TGT form it via AS-REP Roast:

<figure><img src="../../.gitbook/assets/forest17.png" alt=""><figcaption></figcaption></figure>

Yeah, we got a NTLM hash of the user svc-alfresco. Let’s try to crack it using john:

<figure><img src="../../.gitbook/assets/forest18.png" alt=""><figcaption></figcaption></figure>

Yeah, we have a password. Let’s see if it is valid using crackmapexec:

<figure><img src="../../.gitbook/assets/forest19.png" alt=""><figcaption></figcaption></figure>

Yes, its a valid credential.

As we have a valid credential and there is a winrm service active in the port 47001, maybe we can try to gain access to target machine using `evilwinrm`:

<figure><img src="../../.gitbook/assets/forest20.png" alt=""><figcaption></figcaption></figure>

Yeah, we obtained a PowerShell. Let’s look for the user flag:

<figure><img src="../../.gitbook/assets/forest21.png" alt=""><figcaption></figcaption></figure>

Nice.

Now is time to escalate privileges. Let’s see which privileges this user has:

<figure><img src="../../.gitbook/assets/forest22.png" alt=""><figcaption></figcaption></figure>

And let’s see also the groups this account is in

<figure><img src="../../.gitbook/assets/forest23.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/forest24.png" alt=""><figcaption></figcaption></figure>

I see nothing to use. But let’s use Bloodhound to see it more clearly:

<figure><img src="../../.gitbook/assets/forest25.png" alt=""><figcaption></figcaption></figure>

When imported into BloodHound it I searched SVC-ALFRESCO user and marked as OWNED.

<figure><img src="../../.gitbook/assets/forest26.png" alt=""><figcaption></figcaption></figure>

Double clicking on this user I see that it’s included in 9 groups:

<figure><img src="../../.gitbook/assets/forest27.png" alt=""><figcaption></figcaption></figure>

So, I click on the number 9 to display them:

<figure><img src="../../.gitbook/assets/forest28.png" alt=""><figcaption></figcaption></figure>

Mmmm… That group called ACCOUNT OPERATORS looks interesting. Apparently, members of this group are allowed create and modify users and add them to non-protected groups. So maybe we can use that.

Let’s go to the Analysis “Shortest Paths to High Value Targets”

<figure><img src="../../.gitbook/assets/forest29.png" alt=""><figcaption></figcaption></figure>

Is a little bit messy. One of the paths shows that the Exchange Windows Permissions group has WriteDacl

privileges on the Domain. The WriteDACL privilege gives a user the ability to add ACLs to an

object. This means that we can add a user to this group and give them DCSync privileges.

<figure><img src="../../.gitbook/assets/forest30.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/forest31.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/forest32.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/forest33.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/forest34.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/forest35.png" alt=""><figcaption></figcaption></figure>

And now we can use secrets-dump to see the hashes!

<figure><img src="../../.gitbook/assets/forest36.png" alt=""><figcaption></figcaption></figure>

We got the hash of the administrator account.

We can perform a pass the hash attack to log in as the administrator:

<figure><img src="../../.gitbook/assets/forest37.png" alt=""><figcaption></figcaption></figure>

Yeah, we are logged as Administrator. Now is time to search the root flag:

<figure><img src="../../.gitbook/assets/forest38.png" alt=""><figcaption></figcaption></figure>