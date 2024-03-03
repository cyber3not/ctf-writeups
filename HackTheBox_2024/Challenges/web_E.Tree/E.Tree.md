# Web - E.Tree

## Description
> After many years where humans work under the aliens commands, they have been gradually given access to some of their management applications. Can you hack this alien Employ Directory web app and contribute to the greater human rebellion?

## Walkthrough

For this Challenge, we were given the Challenge code and an online instance.

First, as always, I took a look at the online instance.

![Screenshot0](./screenshots/0.png)

We see a page with a search field where we can enter names.

Under `/leaderboard` we have a "Hall of fame".

![Screenshot1](./screenshots/1.png)

Let's take a name from the Hall of fame and see what the application tells us when we enter the name.

![Screenshot2](./screenshots/2.png)

If we simply enter some gibberish, we get a negative output and we are told that this staff does not exist.

![Screenshot3](./screenshots/3.png)

So far so good, let's move on to the challenge code.

__Challenge Code in tree-like format:__

```
├── Dockerfile
├── build-docker.sh
├── challenge
│   ├── application
│   │   ├── app.py
│   │   ├── blueprints
│   │   │   └── routes.py
│   │   ├── config.py
│   │   ├── static
│   │   │   ├── css
│   │   │   │   └── main.css
│   │   │   ├── favicon.png
│   │   │   └── js
│   │   │       └── main.js
│   │   ├── templates
│   │   │   ├── index.html
│   │   │   └── leaderboard.html
│   │   └── util.py
│   ├── military.xml
│   ├── requirements.txt
│   └── run.py
└── config
    └── supervisord.conf
```

The __util.py__ file was of particular interest.

![Screenshot4](./screenshots/4.png)

Here I already became aware of the comment "who cares about parameterization" and the import `from lxml import etree` shows us what kind of data processing we are dealing with here.

Now I took a look at the file `military.xml`.

And here is a small section of it, and here is also our flag in two parts.

![Screenshot5](./screenshots/5.png)

## Exploitation

Now I started to try the exploitation and used __Burp__.

Here is an example of a valid search.

![Screenshot6](./screenshots/6.png)

I first fuzzed the field with the __Burp Intruder__ on all printable ascii characters to detect error messages and other anomalies.

![Screenshot7](./screenshots/7.png)

The only character that causes an HTTP-500 error was a single quote `'` and shows us an XPATH error.

![Screenshot8](./screenshots/8.png)

The resources on [Hacktricks](https://book.hacktricks.xyz/pentesting-web/xpath-injection) were helpful for the progress of the exploitation.

The first thing I found out with this injection type was that we can work with boolean values.

With `' or '1'='1` we get a POSITIVE response:

![Screenshot9](./screenshots/9.png)

And with `' or '1'='2` we get a NEGATIVE response.

![Screenshot10](./screenshots/10.png)

So far so good, we already have valid queries and see that we can work with boolean responses.

So I played around a bit and looked for a way to extract data / our flag.

I found this query useful: `'or //selfDestructCode[contains(., '')] or'`

With this we can check if there is a certain string in the `<selfDestructCode>` element where our flag is stored.

And I already found a proof of concept:

The string 'HTB{' gives us a positive feedback:

![Screenshot11](./screenshots/11.png)

While `DONT_EXIST` gives us a negative response.

![Screenshot12](./screenshots/12.png)

Okay great!

All I had to do now was to brute the flag char by char using the Burp intruder and a printable Ascii list.

So I bruted the flag char by char and looked for the positive responses.


![Screenshot13](./screenshots/13.png)

![Screenshot14](./screenshots/14.png)

