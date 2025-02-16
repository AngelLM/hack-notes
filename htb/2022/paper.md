# Paper

**Date**: 21/05/2022

**Difficulty**: EASY

**CTF**: [https://app.hackthebox.com/machines/Paper](https://app.hackthebox.com/machines/Paper)

***

First things first. let’s test the connection with the target machine:

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Untitled.png" alt=""><figcaption></figcaption></figure>

The ttl value of 63 may indicate that the target machine is Linux.

Let’s launch a nmap scan in order to discover the open tcp ports:

<figure><img src="../../.gitbook/assets/Untitled (1).webp" alt=""><figcaption></figcaption></figure>

There are 3 ports open: 22 (ssh), 80 (http), 443 (https).

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9037862a-3ec6-409e-a5c8-2a77e18a4098/Untitled.png)

Let’s see what is hosted in the http and https ports:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3919974a-5ff9-4d1b-9121-99a7512b0bbc/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5ad5add1-3693-4f92-8353-6ccb760e1729/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/582bad76-27b3-4c40-b626-24bd1a7218a5/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8216841a-8e41-4c0d-b7d6-8901e99cb00d/Untitled.png)

Seems to be the same page.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/81834d6d-2625-4934-b482-625236dda784/Untitled.png)

Wappalizer confirms the versions of apache and openssl. I’m going to search if any of this services has a vulnerability I can use:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6b5df2dc-5f56-4773-8824-714ef22f16a6/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/87049bf9-536f-4236-baa2-a8831d400b2b/Untitled.png)

Not apparently… Let’s enumerate the directories using wfuzz:

`wfuzz -c --hc 404,403 -L -t 200 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt <http://10.10.11.143/FUZZ`>

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/77bc9bd6-b505-4128-8239-d8689e33e558/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/75fea698-2230-4fb6-9010-6095bb138bb1/Untitled.png)

Looks like a standard page…

Ok, no clues. Let’s go back and see what we found so far…

Taking a look to the whatweb response, there is something that looks like a domain… `office.paper` let’s add it to the /etc/hosts file and take a look to it in the web browser:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c01b97be-ad23-45fd-b5f6-20b60b6450c6/Untitled.png)

Yeah, there is a website here!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b956e1f7-37ba-449c-b149-d76cedab6d3c/Untitled.png)

This site is using Wordpress 5.2.3

Let’s take a look to the page content…

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4a8b1b5c-303f-4a9d-a444-6d4ccd97b733/Untitled.png)

The post says that the only user in the blog is `Prisonmike`, but another user (`nick`) replied telling him that he has secret information in the blog drafts. If we gain access to the administration panel we should take a look to the drafts.

There is nothing interesting in the other 2 post available, but we can find other 2 posts if we click on `Search` button:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/49d8be18-9072-4501-b705-1f25d9857516/Untitled.png)

A simple test post and another one of Nick reminding him to not write secrets in the drafts.

We didn’t found anything that could be a password for Prisonmike user, so let’s try to login with default credentials:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/895a524b-30e6-46ff-bfa5-aa88ebae4523/Untitled.png)

`admin` is not a valid user, but `prisonmike` is. But we still don’t know the password.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e29ac080-0972-4672-ab76-d65ebbfeb45a/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dd890ca6-04ce-4e7f-9a27-36864f466ae5/Untitled.png)

Using searchsploit I found a exploit that seems capable of view unauthenticated posts…

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7309a401-29ff-4778-a2e7-c685532f06b6/Untitled.png)

Let’s try it!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fe44e7fd-583c-4991-ad73-bcec1f077458/Untitled.png)

So, yeah, we have access to the draft posts contents… There is one with a “secret” url that seems interesting… Let’s add `chat.office.paper` to /ect/hosts file and visit it with the web-browser

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/47d6beca-bb64-432c-90ed-f8e86b332de1/Untitled.png)

It is a register page, let’s register a new user:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/556180f7-905c-4fa8-ab8f-0835c0216dcc/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/07ffc344-1fc4-45e1-9453-17bd77f69f28/Untitled.png)

Automatically I get invited to a chat:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/99da8763-dece-41c3-8b76-f60538b160a5/Untitled.png)

Let’s take a look to the chat messages:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/901dec31-8ef0-4b29-8b7c-ac786f1d6ee4/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/22030a94-d24a-48ae-a6a5-c7bff10faaef/Untitled.png)

So, let’s open a private chat with Recyclops and see if we can enumerate something:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b4042f33-577b-420f-908d-83559b37e989/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fcfa1648-a00d-4217-b69d-86487b7ca506/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9632fd61-4439-4bc8-a1b6-7e7a86913188/Untitled.png)

Let’s see if it’s vulnerable to path traversal:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2f6c233c-6371-4f7d-a647-957c9f99e4ba/Untitled.png)

Yep, it is… and we should have access to user flag this way:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9d673504-115e-4aa5-b71b-957b789b5dac/Untitled.png)

Not that easy… yep, it is only readable by the owner… there will a ssh key?

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1fcb02df-5778-4b48-87a5-36f3c81ab3dd/Untitled.png)

Nope… but the .hubot\_history sounds interesting:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8aae124b-7e71-47a5-8cf0-95c5e313cd96/Untitled.png)

There is a connect command? I tried to use it, but it doesn’t seems to work.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a39c54d0-812c-42df-b8c7-322f645eccd5/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a0b1f381-d1cc-440a-83fe-9bc348089702/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9fccf95a-cdfd-42a4-84a5-8329e22aef15/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/52649266-8590-4aef-8fac-08839e38e413/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6c2c2571-c1fd-41e2-ad44-734ef75d8e7f/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6ae3d96e-c1d9-4149-ba32-1a5ee3b2fe96/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b11fc88d-b55f-4274-b192-65d451d93d3d/Untitled.png)

woah, we found credentials: `recyclops:Queenofblad3s!23`

Let’s see if we can login as recyclops in the chat:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/954c253c-8bfe-4675-9b36-b427064c75f1/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/152cf553-7b7a-4c30-b4f0-9953549f8db4/Untitled.png)

Nope, we can’t… Recyclops is a bot made by Dwight… Will him be reusing credentials? Let’s check it via ssh:

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/32051fa7-c437-4522-97be-0d4db76b5884/Untitled.png)

Yeah!

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea42d469-b190-40ba-8470-180d0a47ff2b/Untitled.png)

Escalation

[i.sh](http://i.sh)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7f4ca249-5413-4601-a92a-f267c37e8f08/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a950dac4-fc19-49a5-a5cc-09c15ec0aa0d/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/efbdd5fb-56e6-4da7-95a8-3d4afeaa000b/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b770e32-ef0c-4a1d-8ea1-0a485974de51/Untitled.png)
