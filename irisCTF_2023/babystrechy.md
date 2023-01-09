# babystrechy (Web)

**Description**

*"More byte mean more secure
Although this is a web challenge, the script is ran directly with PHP because it doesn't need to have an HTML website attached. Run the command below to connect!"*

**Challenge code**

We were given the following code.
```
<?php
$password = exec("openssl rand -hex 64");

$stretched_password = "";
for($a = 0; $a < strlen($password); $a++) {
    for($b = 0; $b < 64; $b++)
        $stretched_password .= $password[$a];
}

echo "Fear my 4096 byte password!\n> ";

$h = password_hash($stretched_password, PASSWORD_DEFAULT);

while (FALSE !== ($line = fgets(STDIN))) {
    if(password_verify(trim($line), $h)) die(file_get_contents("flag"));
    echo "> ";
}
die("No!");

?>
```

***Break down the Code***

So what happens here?
PHP causes openssl to generate a 64 byte hexadecimal string.
Which can look like the following.

```2bf65abb57ce08fa7cd5d95ed17c44f38c643a5da9f4dd6a6c2a09064296ecddc1dc5a01fd04f1616dee636d6c3a11b2d10fe098b8ea811697c90874fb2b99d1```

The string runs through a loop in the next section which stretches our password to a 4096 byte long string.
Each character is concatenated 64 times in a row.
```
$stretched_password = "";
for($a = 0; $a < strlen($password); $a++) {
    for($b = 0; $b < 64; $b++)
        $stretched_password .= $password[$a];
}
```
After this loop we have a 4096 byte long password.
Our password which I have shortened here now looks like this.

```2222222222222222222222222222222222222222222222222222222222222222bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbffffffffffffff...```

Now our stretched password is given to the php function `password_hash` which turns our long password into a bcrypt hash.

`$h = password_hash($stretched_password, PASSWORD_DEFAULT);`

Then it goes into a loop where the function `password_verify` checks if our input matches the password the application has created before.
If it matches we get the flag.

***Solution***

Well, what is the problem?
After doing some research on the `password_hash` function, I came across the following information on https://book.hacktricks.xyz.

**These functions are typically used in PHP to generate hashes from passwords and to check if a password is correct compared to a hash.
The supported algorithms are: PASSWORD_DEFAULT and PASSWORD_BCRYPT (starts with $2y$). Note that PASSWORD_DEFAULT is often the same as PASSWORD_BCRYPT. And currently PASSWORD_BCRYPT has a size limit on the input of 72 bytes. So if you try to hash something larger than 72 bytes with this algorithm, only the first 72 bytes are used**

OK, the function uses only the first 72 bytes of our password from before!

Our super strong stretched password will be truncated and based on our demonstrative password from before only the following 72 bytes will be used.

`2222222222222222222222222222222222222222222222222222222222222222bbbbbbbb`

I used python to output a list of all 256 possible passwords.

```
hexarray = ['0','1','2','3','4','5','6','7','8','9','a','b','c','d','e','f']
out = ""
for i in hexarray:
    for j in hexarray:
        out = (i * 64) + (j * 8)
        print(out)
```
After a small copy/paste party, I finally found the right combination and got the flag.

`irisctf{truncation_silent_and_deadly}`
