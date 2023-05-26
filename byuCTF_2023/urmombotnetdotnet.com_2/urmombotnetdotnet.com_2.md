# byuCTF 2023 - urmombotnetdotnet.com 2 (Web)

__Description:__

>During my databases class, my group and I decided we'd create a web app with the domain urmombotnetdotnet.com, and wrote the relevant code. At first glance, it looks pretty good! I'd say we were pretty thorough. But were we thorough enough??

>Oh... we also forgot to make the front end :)

The solution to the second flag was at the `/api/tickets` endpoint.

The following code section is responsible for this endpoint:

```python3
@app.route('/api/tickets', methods=['POST'])
@token_required
def post_create_ticket(session_data):
    # ensure needed parameters are present
    if (request.json is None) or ('description' not in request.json):
        return jsonify({'message': 'Missing required parameters'}), 400

    user_id = session_data["user_id"]
    description = request.json['description']
    timestamp = datetime.utcnow().isoformat()

    # ensure parameters are integers
    if type(description) is not str:
        return jsonify({'message': 'Invalid parameter data'}), 400
    # byuctf{fakeflag2}
    # insert ticket into database
    cur = mysql.connection.cursor()
    cur.execute("INSERT INTO Support_Tickets (description, messages, time_stamp, user_id) VALUES (%s, %s, %s, %s)", (description, "", timestamp, user_id))
    mysql.connection.commit()
    ticket_id = cur.lastrowid
    cur.close()

    response = {"ticket_id": ticket_id, "description": description, "time_stamp": timestamp}

    return jsonify(response), 200
```

First I had to get a valid cookie value to be able to work with this endnode which is why I logged into the endpoint `/api/login` first.

After that I formed a curl command to create a ticket and passed it through Burp

`curl -v -X POST --url "http://byuctf.xyz:40010/api/tickets" -H 'Content-Type: application/json' -H 'Cookie: token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjo2LCJpc19zdGFmZiI6ZmFsc2V9.zy3Grtln9EqtyUktZKgyMJ20LAQoxU4AvjeydZ6iZ3g; HttpOnly; Path=/; SameSite=Strict' -d '{"description":"test-desc"}'  --insecure -x 127.0.0.1:8080`

![Screenshot0](./screenshots/0.png)

The solution here was to provoke an error via a very long input string in the `description` parameter.

The application returned an error here and the second flag.

![Screenshot1](./screenshots/1.png)

Flag: `byuctf{oof_remember_to_check_length_limit}`

