# HackTheBox - BoardLight

![Screenshot0](./screenshots/BoardLight.png)

<br>
<br>

## Table of Contents

- [Enumeration](#Enumeration)
    - [Portscan](#Portscan)
	- [Webserver](#Webserver)
	- [CVE-2023-30253](#CVE---2023---30253)
- [Foothold](#Foothold)
	- [Credential Harvesting](#Credential-Harvesting)
- [User](#User)
	- [Password Reuse](#Password-Reuse)
- [Privilege Escalation](#Privilege-Escalation)
	- [SUID](#SUID)
	- [CVE-2022-37706](#CVE---2022---37706)
	- [root](#root)
	
---------------------------------------
<br>
<br>

## Enumeration

### Portscan

It starts with the usual port scan.

__Command:__ `nmap -p- -T4 -sV <IP>`

![Screenshot0](./screenshots/0.png)

The number of open ports is manageable, we have one open SSH port and one web server.

<br>

### Webserver

I moved on to the web server. 

![Screenshot1](./screenshots/1.png)

From here I enumerated the webserver.

I entered the found domain name in the `/etc/hosts` file.

![Screenshot2](./screenshots/2.png)

When fuzzing for subdomains I got the following result

__Dommand:__ `gobuster vhost -w subdomains-top1million-110000.txt -u board.htb --append-domain`
`
![Screenshot3](./screenshots/3.png)

<br>

### CVE-2023-30253

I also entered the subdomain in the `/etc/hosts` file and looked at what was behind it.

![Screenshot4](./screenshots/4.png)

A login interface of __Dolibarr__ is visible and the version couldn't be faster to find.

A quick Google search showed me that this version is vulnerable and I found an exploit on [Github](https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30

However, the exploit requires credentials....

![Screenshot5](./screenshots/5.png)

...which were found quite quickly.

Default credentials `admin:admin` were still set and working.

![Screenshot6](./screenshots/6.png)

<br>
<br>
<br>

## Foothold

I started the exploit, a netcat listener and got the desired reverse shell.

![Screenshot7](./screenshots/7.png)

![Screenshot8](./screenshots/8.png)

<br>

### Credential Harvesting

Ok, so we are now operating under the account `www-data`.

The first thing I looked for was the config file of the web application and the database.

Under the path `/html/crm.board.htb/htdocs/conf` there were some interesting config files.

![Screenshot9](./screenshots/9.png)

And in the file `conf.php` I found a password for the MySQl database.

![Screenshot10](./screenshots/10.png)

<br>
<br>
<br>

## User

### Password Reuse

The password was not only valid for the MySQL database but also for the user larissa.

I logged in via SSH and got the user flag.

![Screenshot11](./screenshots/11.png)

<br>
<br>
<br>

## Privilege Escalation

### SUID

I enumerated the system and found the following interesting binaries while searching for SUID binaries.

![Screenshot12](./screenshots/12.png)

<br>

### CVE-2022-37706

I made myself familiar with these files and quickly found an [exploit](https://www.exploit-db.com/exploits/51180).

The exploit failed completely automatically and without any further action.

![Screenshot13](./screenshots/13.png)

<br>

### root

So I took a closer look at the exploit, went through the steps manually and got my root shell.

![Screenshot13](./screenshots/13.png)
