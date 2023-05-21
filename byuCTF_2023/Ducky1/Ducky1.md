# byuCTF - Ducky1 (Rev)

__Description:__

>I recently got ahold of a Rubber Ducky, and have started automating ALL of my work tasks with it! You should check it out!

We received an `inject.bin` file which came from a hacking gadget called __Rubber Ducky__.

A tool that looks deceptively like a USB stick but is designed to emulate keyboard input on a PC and execute things when plugged in.

The file `inject.bin` which contains the payload is not readable for us yet.

So I found an [online resource](https://ducktoolkit.com/decode) which decodes the original payload and got the flag.

Payload/Flag:

```
DELAY
byuctf{this_was_just_an_intro_alright??}
```
