# Active

**Date**: 27/06/2022

**Difficulty**: Easy

**CTF**: [https://app.hackthebox.com/machines/148](https://app.hackthebox.com/machines/148)

---

Let’s start with the classic ping to test the connection with the target machine:

<figure><img src="../../.gitbook/assets/active0.png" alt=""><figcaption></figcaption></figure>

1 packet emitted, 1 packet received. The ttl shows a value of 127 which in HTB means that we are probably against a Windows machine.

Let’s do a scan of the TCP ports to find which ones are open:

<figure><img src="../../.gitbook/assets/active1.png" alt=""><figcaption></figcaption></figure>

Wow, it shows a bunch of open TCP ports. Let’s do a further scan in these ports:

<figure><img src="../../.gitbook/assets/active2.png" alt=""><figcaption></figcaption></figure>

We have much information here. First of all we have kerberos, RPC and ldap services. We also have a DNS service in port 53 and a http service running on port 47001.

Let’s see if we can any info from the DNS service:

<figure><img src="../../.gitbook/assets/active3.png" alt=""><figcaption></figcaption></figure>

Apparently nothing… Let’s see the http service:

<figure><img src="../../.gitbook/assets/active4.png" alt=""><figcaption></figcaption></figure>

Ok, we also have the port 445 open which is usually used by SMB… Let’s try to obtain more info using crackmapexec:

<figure><img src="../../.gitbook/assets/active5.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/active6.png" alt=""><figcaption></figcaption></figure>

If we search the Build version, we can find that the target server is a Windows Server 2008 R2, SP1.

Now we know that the domain is `active.htb` let’s add it to the `/etc/hosts`.

<figure><img src="../../.gitbook/assets/active7.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/active8.png" alt=""><figcaption></figcaption></figure>

But the http service looks the same.

Let’s try to enumerate the smb:

<figure><img src="../../.gitbook/assets/active9.png" alt=""><figcaption></figcaption></figure>

We have READ permissions to the folder Replication. Let’s look inside!

<figure><img src="../../.gitbook/assets/active10.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/active11.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/active12.png" alt=""><figcaption></figcaption></figure>

Every folder at this level was empty.

<figure><img src="../../.gitbook/assets/active13.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/active14.png" alt=""><figcaption></figcaption></figure>

It seems like it may have interesting files… let’s download all the folder to navigate more quickly:

`smbget -R smb://10.129.81.48/Replication`

<figure><img src="../../.gitbook/assets/active15.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/active16.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/active17.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/active18.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/active19.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/active20.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/active21.png" alt=""><figcaption></figcaption></figure>

Maybe we have credentials here?

`active.htb\SVC_TGS : edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ`

<figure><img src="../../.gitbook/assets/active22.png" alt=""><figcaption></figcaption></figure>

Is not that simple… Let’s see if the username at least is valid using kerbrute:

<figure><img src="../../.gitbook/assets/active23.png" alt=""><figcaption></figcaption></figure>

Yes, it is. So we have a valid username but not its password I guess.

Doing some research, I found [this](https://vk9-sec.com/exploiting-gpp-sysvol-groups-xml/):

<figure><img src="../../.gitbook/assets/active24.png" alt=""><figcaption></figcaption></figure>

So the password seems to be encrypted in AES-256 and we can crack it using gpp-decrypt.

<figure><img src="../../.gitbook/assets/active25.png" alt=""><figcaption></figcaption></figure>

Let’s save this credential in a file.

<figure><img src="../../.gitbook/assets/active26.png" alt=""><figcaption></figcaption></figure>

And now let’s test it:

<figure><img src="../../.gitbook/assets/active27.png" alt=""><figcaption></figcaption></figure>

Yes, it’s valid!

<figure><img src="../../.gitbook/assets/active28.png" alt=""><figcaption></figcaption></figure>

Now, using this credentials we have access to more folders. Let’s look into `Users`:

<figure><img src="../../.gitbook/assets/active29.png" alt=""><figcaption></figcaption></figure>

Can we list the Administrator folder?

<figure><img src="../../.gitbook/assets/active30.png" alt=""><figcaption></figcaption></figure>

Nope. Let’s try with the rest:

<figure><img src="../../.gitbook/assets/active31.png" alt=""><figcaption></figcaption></figure>

Apparently the userflag is in `Users/SVC_TGS/Desktop` path. Let’s download it!

<figure><img src="../../.gitbook/assets/active32.png" alt=""><figcaption></figcaption></figure>

After enumerate the SMB I have found nothing else interesting, let’s try to do a `ldapdomaindumpH`

<figure><img src="../../.gitbook/assets/active33.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/active34.png" alt=""><figcaption></figcaption></figure>

As we saw earlier it is a Windows Server 2008 R2 SP1.

<figure><img src="../../.gitbook/assets/active35.png" alt=""><figcaption></figcaption></figure>

Apparently there are only 4 users

- SVC_TGS: we have its credentials
- krbtgt: Key Distribution Center Service Account
- Guest
- Administrator

Let’s try a Kerberoast attack:

`❯ sudo python3 /home/angellm/THM/CTF/Relevant/impacket/build/scripts-3.9/GetUserSPNs.py active.htb/SVC_TGS:GP#################18 -dc-ip 10.129.104.47 -request`

<figure><img src="../../.gitbook/assets/active36.png" alt=""><figcaption></figcaption></figure>

Let’s now try to crack the hash using hashcat:

`hashcat -m 13100 -a 0 kerberoast_result /usr/share/wordlists/rockyou.txt`

<figure><img src="../../.gitbook/assets/active37.png" alt=""><figcaption></figcaption></figure>

Cracked!

Let’s see if this credentials are correct:

<figure><img src="../../.gitbook/assets/active38.png" alt=""><figcaption></figcaption></figure>

Yeah, Pwned! Let’s go for the root flag:

<figure><img src="../../.gitbook/assets/active39.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/active40.png" alt=""><figcaption></figcaption></figure>

Done!