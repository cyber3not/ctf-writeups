# byuCTF 2023 - Leet 1 (Jail)

Here we had a pyJail task to solve.

The following code was given:

__leet1.py__

```python
import re

FLAG = open('flag.txt').read()

inp = input('> ')

if re.search(r'\d', inp) or eval(inp) != 1337:
    print('Nope')
    print(inp)
else:
    print(FLAG)
```

Let's break down the code a bit.

The condition ```python if re.search(r'\d', inp) or eval(inp) != 1337:``` checks if there is a number in our input string and if the result of `eval(inp)` is not 1337.

So we have to find a way to make an input that comes to the value `1337` without entering any numbers.

## Solution

I used the `ord()` function which returns the charcode of a letter in the form of an integer value and formed a calculation which ends up with 1337.

Payload: `ord("z")+ord("z")+ord("z")+ord("z")+ord("z")+ord("z")+ord("z")+ord("z")+ord("z")+ord("z")+ord("u")`

Flag: `byuctf{simple_bypasses!}`
