# Forensics - Persistence

## Description
> We're noticing some strange connections from a critical PC that can't be replaced. We've run an AV scan to delete the malicious files and rebooted the box, but the connections get re-established. We've taken a backup of some critical system files, can you help us figure out what's going on?

<br>
<br>

## Walkthrough

In this challenge we are given a file named `query`.

First of all, I wanted to know exactly what kind of file we are dealing with and used the `file` command.

![Screenshot0](./screenshots/0.png)

So we are dealing here with a __MS Windows Registry file__.

Now I looked for a tool to parse and display its contents.

The tool __regfexport__ proved to be helpful.

![Screenshot1](./screenshots/1.png)

Now we have a text file in which we can see all registry keys and values.

Under the registry entry __Windows Update__ there was a value for an executable file under the path `C:\Windows\System32\SFRCezFfQzRuX2t3M3J5XzRMUjE5aDd9.exe`.

The name of the exe file is our flag in base64 encoded form.

__Flag:__

![Screenshot2](./screenshots/2.png)
