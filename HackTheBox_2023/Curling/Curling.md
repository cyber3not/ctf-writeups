# HackTheBox - Curling

As always, it starts with a port scan.

![Screenshot0](./screenshots/0.png)

We see an open web server and SSH server.

So I took a look at the web server.

![Screenshot1](./screenshots/1.png)

The layout looked familiar to me as Joomla.

A click on __Wappalyzer__ addon confirms this.

![Screenshot2](./screenshots/2.png)

I looked at the page and made my basic enumerations.

Looking in the source code for __HTTP-Comments__ I found something interesting.

![Screenshot3](./screenshots/3.png)

I looked onto `/secrets.txt` and got a __base 64__ encoded string which I decoded.

![Screenshot4](./screenshots/4.png)

Now I tried to log in with the password under guessed usernames like `admin` but failed.

So I went back to the Joomla content and got a username called `Floris`.

![Screenshot5](./screenshots/5.png)

Now I successfully logged in with the two things I found and was in the Joomla CMS.

![Screenshot6](./screenshots/6.png)

I just had a quick look to see if there were any other users in the CMS, but there were not.

I was logged in as __Floris__ as __Super User__.


In the `Control Panel` -> `Install Extensions` I installed a [webshell plugin](https://github.com/p0dalirius/Joomla-webshell-plugin/tree/master) from which I then established a reverse shell.


## www-data -> floris

In the `home` directory of user `floris` I found an interesting file called `password_backup`.

![Screenshot7](./screenshots/7.png)

Looks like a hashdump of `xxd`.

I fetched the file and had a quick look at how to reverse this hashdump, which is very easy with the `-r` switch.

![Screenshot8](./screenshots/8.png)

I used the `file` command to display the file type.

![Screenshot9](./screenshots/9.png)

The file was easy to extract on kali and I got a `password.txt` file which contained the password of the user `floris`.

I logged in via __SSH__ and got the first flag.

![Screenshot10](./screenshots/10.png)

## floris -> root

In the home directory of floris I noticed an interesting folder which is now relevant for the final Privilege Escalation.

![Screenshot11](./screenshots/11.png)

I quickly used the __pspy64__ tool to see if any regular operations were taking place.

![Screenshot12](./screenshots/12.png)

And yes!

We have an interesting `curl` command that does something with the input and output file.

The whole thing is executed as __UID 0__, so __root__.

But what happens here?

The `-K` switch reads in a config file that curl works with and with the `-o` switch the output ends up in the `report` file.

Which now also explains the previous content of the two files.

Curl retrieves the url `127.0.0.1` and outputs it to the report file.

</br>
</br>

__Root Flag:__

Curl is a really comprehensive tool and accepts more protocols than just HTTP.

Furthermore, we can also read files with `file://`.

So I got the root flag Quick & Lazy by changing the url parameter in the input config as follows.

![Screenshot13](./screenshots/13.png)

And got it from the output in the `report` file.

![Screenshot14](./screenshots/14.png)

