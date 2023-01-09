# wi-the-fi (Networks)

**Description**

*"You're probably used to pcaps captured at layer 3 in promiscuous mode, but do you know what to do with a pcap captured at layer 2 in monitor mode?"*

**Solution**

In this network challenge we were given a file `BobertsonNet.cap`.

As the name and the description of the challenge suggested, this is a capture with sniffed Wi-Fi traffic.
To decrypt the traffic we first need the Wi-Fi Password.

Then let's see if we have a valid handshake in our capture file with which we can crack the password.

```
#aircrack-ng BobertsonNet.cap

      BSSID              ESSID                     Encryption

   1  A0:28:ED:C3:CC:C1  BobertsonNet              WPA (1 handshake)
```
We can see that we are given a handshake and we can start cracking the password.
```
#aircrack-ng BobertsonNet.cap -w /usr/share/wordlists/rockyou.txt

                               Aircrack-ng 1.6 

      [00:00:02] 13965/10303727 keys tested (9216.85 k/s) 

      Time left: 18 minutes, 36 seconds                          0.14%

                           KEY FOUND! [ billybob1 ]


      Master Key     : 84 F1 82 B2 91 7C D3 DD 26 B2 19 34 C0 7B 27 2F 
                       AC CF 32 B2 DF A3 56 01 9E FE 8C 7A 23 38 15 F8 

      Transient Key  : BA 3F 2A 6D D0 DB 35 F8 D4 60 FF BB EF CB B5 1A 
                       CE 60 FD 5F 58 D0 1F C4 3A EA 03 EA D4 DE 41 C9 
                       4B 0B 91 19 1F 47 2D E3 8A 26 19 85 69 89 37 F3 
                       2D 24 11 A4 C7 31 F9 D6 8F 6D 87 4F 17 87 01 9D 

      EAPOL HMAC     : 80 8E 84 12 3F EA 18 4E 2D 25 9A 19 10 54 E2 51
```
After a short time it spits out the password and we can decrypt the Wi-Fi traffic.
For this we use the tool `airdecap-ng`.   

`airdecap-ng -e BobertsonNet -p billybob1 BobertsonNet.cap`

Airdecap has now given us a new file `BobertsonNet-dec.cap` and successfully decrypted the traffic.
A simple string/grep command was enough and we get the flag.

```
#strings BobertsonNet-dec.cap | egrep -i 'iris'
irisctf{4ircr4ck_g0_brrrrrrrrrrrrrrr}
```


