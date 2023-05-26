# byuCTF 2023 - urmombotnetdotnet.com 3 (Web)

__Description:__

>During my databases class, my group and I decided we'd create a web app with the domain urmombotnetdotnet.com, and wrote the relevant code. At first glance, it looks pretty good! I'd say we were pretty thorough. But were we thorough enough??

>Oh... we also forgot to make the front end :)

The solution to the third flag was at the `/api/tickets/<ticketid>` endpoint.

The following code section is responsible for this endpoint:

```python3
@app.route('/api/tickets/<int:ticket_id>', methods=['POST'])
@token_required
def post_add_message(session_data, ticket_id):
    # get user_id from ticket_id
    cur = mysql.connection.cursor()
    cur.execute("SELECT user_id, message FROM support_tickets WHERE ticket_id=%s", (ticket_id,))
    response = cur.fetchone()
    cur.close()

    if not response and session_data['is_staff']:
        return jsonify({'message': 'Ticket not found'}), 404

    if not response or session_data['user_id'] != response[0]:
        return jsonify({'message': 'You do not have permission to access this information'}), 403

    # ensure needed parameters are present
    if (request.json is None) or ('message' not in request.json):
        return jsonify({'message': 'Missing required parameters'}), 400

    message = request.json['message']

    # ensure parameters are integers
    if type(message) is not str:
        return jsonify({'message': 'Invalid parameter data'}), 400
    # byuctf{fakeflag3}
    # insert message into database
    cur = mysql.connection.cursor()
    new_message = response[1] + "\n" + message
    cur.execute("UPDATE Support_Tickets SET message=%s WHERE ticket_id=%s", (new_message, ticket_id))
    mysql.connection.commit()
    cur.close()

    response = {"ticket_id": ticket_id, "message": message}

    return jsonify(response), 200
```

I formed here again a Curl request which I passed through Burp.

`curl -v -X POST --url "http://byuctf.xyz:40010/api/tickets/501988" -H 'Content-Type: application/json' -H 'Cookie: token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxNDQ2LCJpc19zdGFmZiI6ZmFsc2V9.ugp7Y4bndnQiLkhNQtJcngT-zbX4Xkd_fakcO9Gkwy0; HttpOnly; Path=/; SameSite=Strict' -d '{"message":"message1"}'`

Here I fuzzed the parameter for a while on different things and characters and got the flag.

Flag: `byuctf{let's_not_even_talk_about_the_newline_injection...}`
