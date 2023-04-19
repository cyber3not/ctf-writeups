# HackTheBox - Blunder

We start with a simple portscan as always.

`nmap -sS -T4 -sV <IP>`

![Screenshot0](./screenshots/0.png)

We see an FTP server, but its port is `closed` and an open web server on port `80`.

When we visit the site we see a blog page set up with a few facts about different things.

![Screenshot1](./screenshots/1.png)

I started to enumerate the page manually and to run the usual tools like `FFUF`.

Under the path `/admin` we can see a login panel to `Bludit` and a quick search for it tells us that this is a CMS.

![Screenshot2](./screenshots/2.png)

I found out a more exact version using the Firefox developer console in the `Network` tab.

![Screenshot3](./screenshots/3.png)

This version seems to have a `RCE` vulnerability and furthermore I found an `Auth Brute Force Bypass`.
I used an Auth-bypass script and first started to find a valid access with the typical `admin` user which did not work for a long time.

When I looked at the results of `FUFF`, I saw an interesting file on the webserver called `todo.txt` whose content was as follows:

![Screenshot4](./screenshots/4.png)

Ok, lo and behold we found a user named `fergus`.
I started to brute the user `fergus` with some most used passwords.
But that didn`t work either and I was a bit stuck.

Later I got the idea to create a custom bruteforce list with the tool `cewl` with all used words on the page.

And finally it worked:

![Screenshot5](./screenshots/5.png)

With this access data I could now proceed to the next exploit and got a reverse shell as user `www-data`.

![Screenshot6](./screenshots/6.png)

## PrivEsc 1: www-data -> hugo

I started to enumerate the system.
It took a little while until I found the essentials.

Interesting was that under the path `/var/www` were two folders of different bludit versions.

![Screenshot7](./screenshots/7.png)

I took a closer look at these folders and on the newer version under the path `/var/www/bludit-3.10.0a/bl-content/databases` was a file `users.php` which contains settings for a used user named `Hugo` which is also found on our Linux system.

![Screenshot8](./screenshots/8.png)

I tried to crack the password with the `rockyou.txt` wordlist which failed.
With the switch `-g` which generates random rules I started again and the password was cracked.

![Screenshot9](./screenshots/9.png)

I logged in with the password as user `hugo` and got the user flag.

![Screenshot10](./screenshots/10.png)

## PrivEsc 2: hugo -> root

The Privelege Escalation was done quickly, unlike the previous section.
One of the first things I looked at was `sudo`.

`sudo -l` shows us the following.

![Screenshot11](./screenshots/11.png)

Well, we can run `/bin/bash` as any user we want, just not as `root` which we want to get.

I looked at the sudo version with `sudo -V`.

![Screenshot12](./screenshots/12.png)

I looked to see if this version was vulnerable and found a simple exploit: [CVE-2019-14287](https://www.exploit-db.com/exploits/47502).

`sudo -u#-1 /bin/bash` does the job and I got the flag.

![Screenshot13](./screenshots/13.png)
