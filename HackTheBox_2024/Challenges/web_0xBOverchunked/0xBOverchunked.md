# Web - 0xBOverchunked

## Description
> Are you able to retrieve the 6th character from the database?

## Walkthrough

We have a web challenge where the source code is also available to us.

I went to the website first and we have a simple input field and we can enter numbers and get information about different gameboy characters.

![Sreenshot0](./screenshots/0.png)

Well...at least up to number 5.

For number 6 we get the information that we are not allowed to see it.

![Sreenshot1](./screenshots/1.png)

Because that's where our flag is, as we can see from the sql file `init.sql`.

![Sreenshot2](./screenshots/2.png)

I first played around with the input field and tried it out.

When we try SQL Injection we get the following output

![Sreenshot3](./screenshots/3.png)

Well OK, let's take a look at the challenge code.

![Sreenshot4](./screenshots/4.png)

The highlighted area is of particular importance.

If a HTTP header field called `Transfer-Encoding:` with the value `chunked` is set in the query, a function called `unsafequery` is used.

Let's go to the code of these functions.

In the file `Cursor.php` we see our `unsafequery` function and the safe one.

![Sreenshot5](./screenshots/5.png)

Our insecure function is vulnerable to SQL injection, the other one is not because it uses prepared statements.

It is also important to note here that the function is written in such a way that the status code `500` is returned for a non-valid MYSQL query.

### Solution

Ok, so all we need to do is add a HTTP header field `Transfer-Encoding: chunked` and then we can perform the SQL injection.

With `curl` I manually confirmed the SQL gap first.

__Valid:__

![Sreenshot6](./screenshots/6.png)

__Error:__

![Sreenshot7](./screenshots/7.png)

I then performed the exploitation with `sqlmap` and used the `--header` flag.

`sqlmap --dbms sqlite -u "http://<IP>/Controllers/Handlers/SearchHandler.php" --method=POST --data "search=1" -p search --header "Transfer-Encoding: chunked"`

![Sreenshot8](./screenshots/8.png)

The vulnerability was confirmed and I dumped the database/flag.

![Sreenshot9](./screenshots/9.png)
