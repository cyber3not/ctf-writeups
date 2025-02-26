# HackTheBox - Cicada

![Screenshot0](./screenshots/Cicada.png)

<br>
<br>

## Table of Contents

- [Enumeration](#Enumeration)
    - [Portscan](#Portscan)
	- [SMB-Share HR](#SMB---Share-HR)
- [Foothold](#Foothold)
	- [RID Cycling](#RID-Cycling)
	- [Password Spraying](#Password-Spraying)
	- [Credentiailed Enumeration](#Credentiailed-Enumeration)
	- [Password in Description](#Password-in-Description)
	- [SMB-Share DEV](#SMB---Share-DEV)
- [User]
	- [Hardcoded Credentials](#Hardcoded-Credentials)
- [Privilege Escalation](#Privilege-Escalation)
	- [SeBackupPrivileges](#SeBackupPrivileges)
	- [root](#root)

<br>
-----------------------------
<br>

## Enumeration

### Portscan

As always, it starts with a standard port scan.

__Command:__ `nmap -p- -T4 -sV <IP>`

![Screenshot0](./screenshots/0.png)

We see all kinds of ports open and can easily recognize that we are dealing with a domain controller.

I ran nmap again with the `-A` switch to get more information from the NSE scripts.

For example, the FQDN of the domain controller we need.

![Screenshot1](./screenshots/1.png)

<br>

### SMB-Share HR

With windows, SMB is often the first thing I look at when it is open.

Using `Guest` or any other random username we can see shares.

![Screenshot2](./screenshots/2.png)

The DEV share looks interesting.

Let's take a look at what's there using the spider module from netexec.

__Command:__ `netexec smb cicada.htb -u 'Guest' -p '' --shares -M spider_plus`

![Screenshot3](./screenshots/3.png)

So I downloaded the file and took a look at the note.

![Screenshot4](./screenshots/4.png)

And we have a default password for new employee accounts.

<br>
<br>
<br>

## Foothold

### RID Cycling


Now I already had a password but no username.

So this is where RID Cycling comes into play, for which I used `netexec`.

__Command:__ `netexec smb cicada.htb -u 'Guest' -p '' --rid-brute`

![Screenshot5](./screenshots/5.png)

<br>

### Password Spraying

I created a small userfile and with a short PassworSpray I got valid credentials.

![Screenshot6](./screenshots/6.png)

<br>

### Credentiailed Enumeration

Now that we have valid access data, we have new possibilities.

A new enumeration cycle began.

I looked at group memberships, since LDAP enumeration was now also open to us, I looked at the accounts and structures graphically using Bloodhound.

<br>

### Password in Description

At some point I came across a password in the description field of the user `david.orelious` who keeps his password there in case he forgets it.

Bad idea...

![Screenshot7](./screenshots/7.png)

<br>

### SMB-Share DEV

New user, new luck!

With these credentials, we can now access a share that we could not access before.

![Screenshot8](./screenshots/8.png)

There is an interesting script on the share.

![Screenshot9](./screenshots/9.png)

So let's get it and take a look at the contents.

![Screenshot10](./screenshots/10.png)

<br>
<br>
<br>

## User

### Hardcoded Credentials

The backup script obtained contained access data from the account `emily.oscars`.

![Screenshot11](./screenshots/11.png)

User `emily.oscars` is in the group `Remote Management Users`, which means that we can log in using WinRm.

I logged in with `evil-winrm` and got the first user flag.

![Screenshot12](./screenshots/12.png)

<br>
<br>
<br>

## Privilege Escalation

### SeBackupPrivileges

In comparison to the user part, the privilege Escalation was done quickly.

A view at the output `whoami /all` shows us that the account has `SeBackupPrivileges`.

This gives us the possibility to read all files on the file system.

I fetched the **SAM** and **SYSTEM** file from Windows and then used Impacket to read the hashes.

![Screenshot13](./screenshots/13.png)

![Screenshot14](./screenshots/14.png)

<br>

### root

Using **Pass the hash** I logged in with `evil-winrm` and got the final root flag from the administrator.

![Screenshot15](./screenshots/15.png)
