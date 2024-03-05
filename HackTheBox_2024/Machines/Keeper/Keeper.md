# HackTheBox - Keeper

I started with the usual port scan.

`nmap -p- -T4 -sV -sC <IP>`

![Screenshot0](./screenshots/0.png)

I entered the IP in the browser to see what is on the webserver.

![Screenshot1](./screenshots/1.png)

And we get a sub-/domain to an __IT support ticket system__.

![Screenshot2](./screenshots/2.png)

I briefly tried some __Password Guessing__ and later found out that the working credentials `root:password` were just the __default credentials__ from the installation.

![Screenshot3](./screenshots/3.png)

Now I was logged in and first clicked through the menus and looked at everything.

Under the menu `Admin` -> `Users` I found users.

![Screenshot4](./screenshots/4.png)

And with the user __lnorgaard__ I found access data for this user.

![Screenshot5](./screenshots/5.png)

I logged in with the found password as that user via SSH and got the first flag.

![Screenshot6](./screenshots/6.png)

## lnorgaard -> root

An interesting ZIP file immediately appeared which I first downloaded and unpacked.

![Screenshot7](./screenshots/7.png)

And a short file command check tells us exactly what files we are dealing with.

![Screenshot8](./screenshots/8.png)

Ok, the first thing I tried was to extract the hash with `keepass2john` and try to crack it.

However, this ran for a long time without success.

Well, the second `.dmp` crash file will probably also be there for a reason.

After some searching I learned of a vulnerability that affects certain Keepass versions: [CVE-2023-32784](https://nvd.nist.gov/vuln/detail/CVE-2023-32784).

So I moved the crash dump file to my Windows VM and used the following [POC](https://github.com/vdohney/keepass-password-dumper).

![Screenshot9](./screenshots/9.png)

![Screenshot92](./screenshots/92.png)

Okay, I tried to open the Keepass file, but `dgrød med fløde` still didn't work.

A simple google search for it and the google "Did you mean XYZ" suggestion then brought me to the full password and meaning.

![Screenshot10](./screenshots/10.png)

So the password was kinda a sweet dessert and the `R` was missing at the beginning.

And now i was able to open the Keepass File.

![Screenshot11](./screenshots/11.png)

The `PuTTY-User-Key-File-3` was interesting.

I transformed this into an OpenSSH compatible format.

![Screenshot12](./screenshots/12.png)

And logged in via SSH as root and got the final flag.

![Screenshot13](./screenshots/13.png)
