# urmombotnetdotnet.com 1

__Description:__

>During my databases class, my group and I decided we'd create a web app with the domain urmombotnetdotnet.com, and wrote the relevant code. At first glance, it looks pretty good! I'd say we were pretty thorough. But were we thorough enough??

>Oh... we also forgot to make the front end :)

I navigated to the page and got only a `Not Found` / `404` from the webserver.
Now, as mentioned in the description, there is no front end here.

However, we were given the source code of the application so I looked at how it works.

I looked at how the different API endnodes work and first built working `curl` commands to interact with the API.
Then I passed them through Burp to test the application.

The following code section of the application handles the endpoint `/api/register`.

```python3
@app.route('/api/register', methods=['POST'])
def post_register():
    # ensure needed parameters are present
    if (request.json is None) or ('email' not in request.json) or ('username' not in request.json) or ('password' not in request.json) or ('bitcoin_wallet' not in request.json):
        return jsonify({'message': 'Missing required parameters'}), 400

    email = request.json['email']
    username = request.json['username']
    password = request.json['password']
    bitcoin_wallet = request.json['bitcoin_wallet']

    # ensure parameters are strings
    if type(email) is not str or type(username) is not str or type(password) is not str or type(bitcoin_wallet) is not str:
        return jsonify({'message': 'Invalid parameter data'}), 400

    # ensure email is valid
    if not re.fullmatch(r'\b[A-Za-z0-9._+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,7}\b', email):
        return jsonify({'message': 'Invalid email'}), 400

    # ensure username is valid
    if len(username) < 4 or len(username) > 255:
        return jsonify({'message': 'Invalid username length'}), 400

    # ensure username isn't already taken
    cur = mysql.connection.cursor()
    cur.execute("SELECT username FROM User WHERE username=%s", (username,))
    users_found = cur.rowcount
    cur.close()
    username_taken = (users_found > 0)

    if username_taken:
        return jsonify({'message': 'Username already taken'}), 500

    # ensure password is valid
    if len(password) < 12 or len(password) > 255:
        return jsonify({'message': 'Password doesn\'t fit length requirements'}), 400

    # ensure bitcoin wallet is valid
    if not re.fullmatch(r'0x[0-9a-fA-F]+', bitcoin_wallet):
        return jsonify({'message': 'Invalid bitcoin wallet'}), 400
    # byuctf{fakeflag1}
    # insert user into database
    cur = mysql.connection.cursor()
    cur.execute("INSERT INTO User (email, username, password, blocked, bitcoin_wallet) VALUES (%s, %s, %s, %s, %s)", (email, username, sha256(password.encode()).hexdigest(), 0, bitcoin_wallet))
    mysql.connection.commit()
    user_id = cur.lastrowid
    cur.close()

    # add user as affiliate
    cur = mysql.connection.cursor()
    cur.execute("INSERT INTO Affiliates (user_id, Money_received, total_bots_added) VALUES (%s, %s, %s)", (user_id, 0, 0))
    mysql.connection.commit()
    cur.close()

    response = {"user_id": user_id}
    return jsonify(response), 200
```

I crafted the following curl command and used `--insecure -x 127.0.0.1:8080` to pass the request through Burp.

`curl -X POST -H 'Content-Type: application/json' -d '{"username":"bbbbbbbbbbbbbb","password":"bbbbbbbbbbbbbb","email":"bbbb@bbbb.com","bitcoin_wallet":"0x00"}' http://byuctf.xyz:40010/api/register --insecure -x 127.0.0.1:8080`

By simply provoking an error here, the webapp spat out an error which gave me the first flag.

Flag: `byuctf{did_you_stumble_upon_this_flag_by_accident_through_a_dup_email?}`
