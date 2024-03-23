# Reversing - PackedAway

## Description
> To escape the arena's latest trap, you'll need to get into a secure vault - and quick! There's a password prompt waiting for you in front of the door however - can you unpack the password quick and get to safety?

<br>
<br>

## Walkthrough

In this challenge there was a binary called `packed`.

First I used the `strings` command and I noticed the following.

![Screenshot0](./screenshots/0.png)

Apparently this is an `UPX` packed binary.

With the help of `upx-ucl` we can unpack the binary and get the original code.

<br>

__Command:__ `upx-ucl -d packed`

![Screenshot1](./screenshots/1.png)

Now I could run 'strings' again and got the flag.

<br>

__Command:__ `strings packed | egrep -i HTB`

![Screenshot2](./screenshots/2.png)

