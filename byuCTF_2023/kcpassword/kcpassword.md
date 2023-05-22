# byuCTF 2023 - kcpassword (Forensics)

__Description:__

>I'm too lazy to log into my Mac each time, so I enabled auto-logon. It's okay though because I'm sure that Apple will protect my password sufficiently...

For a MAC user who uses __auto-login password__, their passwords are stored under `/etc/kcpassword`.

The password was quickly cracked with the following tool: [decode-kcpassword.py](https://github.com/Heisenberk/decode-kcpassword/tree/master)

Flag: `byuctf{wow_Macs_really_have_it_encrypted_with_a_static_key_lol}`
