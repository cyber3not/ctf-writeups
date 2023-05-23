# byuCTF 2023 - Leet 2 (Jail)

This challenge was a continuation of the challenge __Leet 1__.

The code of this pyJail was here the following:

__leet2.py__


```python
import re

FLAG = open('flag.txt').read()

inp = input('> ')

if re.search(r'[123456789]', inp) or re.search(r'\(', inp) or eval(inp) != 1337:
    print('Nope')
else:
    print(FLAG)
```

The condition was a little bit different to get the flag.

Let's look at the condition again:

`if re.search(r'[123456789]', inp) or re.search(r'\(', inp) or eval(inp) != 1337:`

As in the previous challenge, we still cannot enter any numbers and we have to arrive at a value of `1337`.

Additionally this condition forbids us to use an open `(` bracket.

So the trick with our built-in function from before doesn't work here anymore and we have to come up with something new.

## Solution

So I played around locally with a python interpreter and thought about how we can get the value `1337.

The following showed me that we can calculate with the value `True` and use it.

```
>>> True + True + True
3
```

First I tried a very long payload using 1337 times `True +` in a row.
But this only brought me into traceback errors.

So I looked and thought about what else python provides us with.

I found out that the __bitwise operators__ are useful and formed a working bypass with `<<` and `^`.

Bypass: `True << True << True << True << True << True << True << True << True << True << True ^ True << True << True << True << True << True << True << True << True ^ True << True ^ True << True << True << True << True << True ^ True ^ True << True << True << True << True ^ True << True ^ True << True << True << True`

Flag: `byuctf{aaaaaaand_more_simple_bypasses_:)}`
