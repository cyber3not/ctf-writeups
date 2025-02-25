# HackTheBox - MonitorsThree

![Screenshot0](./screenshots/MonitorsThree.png)

<br>
<br>

## Table of Contents

- [Enumeration](#Enumeration)
    - [Portscan](#Portscan)
	- [Webserver](#Webserver)
- [Foothold](#Foothold)
	- [SQL Injection](#SQL-Injection)
	- [Hash Cracking](#Hash-Cracking)
	- [CVE-2024-25641](#CVE---2024---25641)
- [User](#User)
	- [www-data -> marcus](#www---data-->-marcus)
- [Privilege Escalation](#Privilege-Escalation)
	- [Local Services](#Local-Services)
	- [Duplicati](#Duplicati)
	- [Duplicati - Authentication Bypass](#Duplicati---Authentication-Bypass)
	- [root](#root)


<br>

## Enumeration

### Portscan

As usual, it starts with a port scan and service enumeration.

__Command:__ `nmap -p- -T4 -sV <IP>`

![Screenshot0](./screenshots/0.png)

Two open ports, an open SSH server and a web server came to light.

<br>

### Webserver

We need to add the domain `monitorsthree.htb` to the `/etc/hosts` file and then we can take a look at the page.

![Screenshot1](./screenshots/1.png)

On the web server is a slim website which advertises Networking Solutions and Monitoring Solutions.

I took a quick look around and read the content of the page.

At __Login__ we have a login portal probably to an admin portal

![Screenshot2](./screenshots/2.png)

I rattled off my web-numbering checklist and went in search of other interesting things.

<br>

### Subdomain

When fuzzing for subdomains, a subdomain named `cacti.monitorsthree.htb` came up.

![Screenshot3](./screenshots/3.png)

![Screenshot4](./screenshots/4.png)

<br>
<br>
<br>

## Foothold

### SQL Injection

At some point I went to the first exploitation attempts and with the password reset function an interesting error message appeared when inserting a single `'` apostrophe

![Screenshot5](./screenshots/5.png)

This looks suspiciously like a SQLi vulnerability.

So I saved the burp request to a file and fired up `sqlmap`.

![Screenshot6](./screenshots/6.png)

It didn't take long and the vulnerability was confirmed by SQLMap.

![Screenshot7](./screenshots/7.png)

So I enumerated the database using SQLmap with focus of course on access data.

![Screenshot8](./screenshots/8.png)

![Screenshot9](./screenshots/9.png)

And finally I got the user table.

![Screenshot10](./screenshots/10.png)

<br>

### Hash Cracking

I inserted the hashes into a file and cracked the hashes using hashcat and the well-known rockyou list.

![Screenshot11](./screenshots/11.png)

I was now able to log in to both admin dashboards with the password I received.

![Screenshot12](./screenshots/12.png)

![Screenshot13](./screenshots/13.png)

<br>

### CVE-2024-25641

Now that I was able to log in to the Cacti Dashboard.

There was a matching [POC](#https://github.com/cacti/cacti/security/advisories/GHSA-7cmj-g5qc-pj88) and [Metasploit Module](#https://github.com/rapid7/metasploit-framework/blob/master//modules/exploits/multi/http/cacti_package_import_rce.rb) for the exploit

I got a reverseshell as `www-data`.

![Screenshot14](./screenshots/14.png)

<br>
<br>
<br>

## User

### www-data -> marcus

So I started to look around on the system.

Under the directory `/var/www/html/cacti/include` I found access data for the MySQL database used by cacti.

![Screenshot15](./screenshots/15.png)

So I logged into the mysql database and searched for access data.

There were three users in the `user_auth` table.

![Screenshot16](./screenshots/16.png)

I put the three hashes in a file and cracked the hashes using hashcat.

The password from marcus could be recovered and I got the first flag.

![Screenshot17](./screenshots/17.png)

<br>
<br>
<br>

## Privilege Escalation

OK, now to the privilege escalation.

In the home directory of the user Marcus I first found his private SSH key, so that I could now continue working with a stable SSH session.

I checked my Linux system enumeration cheat sheet and made notes.

<br>

### Local Services

There were a few local services on the machine that I wanted to take a closer look at, so I tunneled them to me.


![Screenshot18](./screenshots/18.png)

__Command:__ `ssh -L 8200:localhost:8200 marcus@monitorsthree.htb -i id_rsa`

![Screenshot19](./screenshots/19.png)

### Duplicati

A login panel from __Duplicati__, an open source backup solution, appeared.

The first thing I tried to do was to log in with the passwords I already knew and had discovered, but this did not work.

<br>

### Duplicati - Authentication Bypass

When I was looking for vulnerabilities to Duplicati, I learned that Duplicati has an interesting vulnerability to bypass login.

In this [article](#https://medium.com/@STarXT/duplicati-bypassing-login-authentication-with-server-passphrase-024d6991e9ee) the steps to bypass are well explained.

I familiarized myself with the article and set about exploiting it.

Under `/opt/duplicati/config` is the Duplicati sqlite file which we need and which I downloaded to my Attacker machine.

![Screenshot20](./screenshots/20.png)

In the `Options` table we take the `server-passphrase` and base64 encode this.

![Screenshot21](./screenshots/21.png)

Then we place a request in Burp and make sure that we go to Intercept and get the Nonce Value.

![Screenshot22](./screenshots/22.png)

Now we use the dev console to help us and let us generate the password in JavaScript with everything we have.

```js
var saltedpwd = "59be9ef39e4bdec37d2d3682bb03d7b9abadb304c841b7a498c02bec1acad87a" ;var noncedpwd = CryptoJS.SHA256(CryptoJS.enc.Hex.parse(CryptoJS.enc . Base64 . parse("T61XB3h+VGTUUOBd3bCWE62VCJPMsbRz1A0wjLhncKk=")+ saltedpwd)).toString(CryptoJS . enc . Base64);    console . log (noncedpwd);
```

![Screenshot23](./screenshots/23.png)

We receive a valid password, which we replace with the password in the following request.

![Screenshot24](./screenshots/24.png)

And we are successfully logged in!

![Screenshot25](./screenshots/25.png)

<br>

### root

Now I was successfully logged into Duplicati and somehow I had to continue from here.

Since Duplicati is a backup solution, my first thought was that there was juicy information in some of the backups that would lead me to the root flag.

I spent a long time looking at the file structure, but that was not the way to go.

The way was to get the root flag from a backup and then restore it.

The following steps led me to the root flag.

__1. Create Backup:__

Create and execute a backup.

I specified a folder created under `/tmp` as the destination.

And as source we enter: `/source/rooot/`.

![Screenshot26](./screenshots/26.png)

![Screenshot27](./screenshots/27.png)

![Screenshot28](./screenshots/28.png)

__2. Run Backup:__

![Screenshot29](./screenshots/29.png)

![Screenshot30](./screenshots/30.png)

__3. Restore:__

![Screenshot31](./screenshots/31.png)

![Screenshot32](./screenshots/32.png)

And after some trial and error, the final flag was raised.

![Screenshot33](./screenshots/33.png)
