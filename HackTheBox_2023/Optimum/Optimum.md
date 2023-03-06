## HackTheBox - Optimum

# User

I start with the usual portscan to see which ports are open to us.

`nmap -sS -T4 -sV <IP>`

![Screenshot0](./screenshots/0.png)

We see only one open web server on port 80.
A quick research on this service and version quickly lets us know that this service is vulnerable.

Metasploit provides us with an exploit.

![Screenshot1](./screenshots/1.png)

The exploit worked fine, I got reverse shell and fetched the user flag.

![Screenshot2](./screenshots/2.png)

# System

The privilege escalation was done with metasploit's `multi/recon/local_exploit_suggester` module just as quickly.
The exploit suggester modul suggested two possible exploits to me.

![Screenshot3](./screenshots/3.png)

The second exploit worked, I got a session as `NT AUTHORITY\SYSTEM` and fetched the root flag.

![Screenshot4](./screenshots/4.png)
