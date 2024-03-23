# Reversing - BoxCutter

## Description
> You've received a supply of valuable food and medicine from a generous sponsor. There's just one problem - the box is made of solid steel! Luckily, there's a dumb automated defense robot which you may be able to trick into opening the box for you - it's programmed to only attack things with the correct label.

## Walkthrough

In this case we were given a binary called `cutter`.

If we run this binary, we only get the message "[X] Error: Box Not Found" to see.

![Screenshot0](./screenshots/0.md)

### Solution

The solution here was to use `strace` to see which system calls the binary was making and so the flag could be found.

`strace ./cutter`

![Screenshot1](./screenshots/1.md)

The binary tries to open a file with the name of our flag.


