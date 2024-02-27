# Web - Orbital

## Description
> In order to decipher the alien communication that held the key to their location, she needed access to a decoder with advanced capabilities - a decoder that only The Orbital firm possessed. Can you get your hands on the decoder?

## Walkthrough

In this challenge, we are given the challenge code and an online instance.

The first thing I did was look at the site and came across a login portal.

![Screenshot0](./screenshots/0.png)

Now I took a look at the challenge code.

![Screenshot1](./screenshots/1.png)

These lines of code handle our login.

The comment also leads us in the direction that there is probably a __SQLi vulnerability__ here.

I intercepted a normal login attempt via burp, saved the request in a `login.req` text file and ran __sqlmap__.

__Command:__ `sqlmap -r login.req --level 4 --risk 3 --dbms=mysql --batch`

![Screenshot2](./screenshots/2.png)

SQLmap quickly confirmed a vulnerability and showed us available databases.

In the database `orbital` was a table called `user` which I dumped and got a md5-hash.

![Screenshot3](./screenshots/3.png)

The hash was quickly cracked using `hashcat`.

![Screenshot4](./screenshots/4.png)

Now I could log into the application and had access to further functions.

Interesting here was an `export` function with which we can download audio files from the web server.

![Screenshot5](./screenshots/5.png)

Intercepted in Burp, the whole thing looked like this.

![Screenshot6](./screenshots/6.png)

Now to the challenge code of this export function:

![Screenshot7](./screenshots/7.png)

The comment leads us in the right direction again and shows us a weak filter which is probably trying to prevent __Local File Inclusions__.

Any input `../` is filtered out.

A small illustration of how the filter works:

```
Input    | After passing Filter
---------+---------------------
../      | <NOTHING>
.../     | .
..//     | ./
..././   | ../
```

So our bypass for LFI is `..././` and we can use it to move through the file system and include other files.

I included the `/etc/passwd` file and our LFI is confirmed.

![Screenshot8](./screenshots/8.png)

But what about our flag?

According to the unpacked file, the flag should be in the webroot directory.

I ran the docker instance locally and saw that the flag is replaced and renamed.

![Screenshot9](./screenshots/9.png)

OK,then I'll just get the flag under `/signal_sleuth_firmware`!

![Screenshot10](./screenshots/10.png)
