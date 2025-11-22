# Description

This lab contains a reflected XSS vulnerability in the search functionality but uses a web application firewall (WAF) to protect against common XSS vectors.

To solve the lab, perform a cross-site scripting attack that bypasses the WAF and calls the print() function.

> Your solution must not require any user interaction. Manually causing print() to be called in your own browser will not solve the lab.

# Solution

Let's go to the website and test the standard payload by entering `<script>alert(123)</script>`. Nothing happened; this tag is blocked. 

Let's try entering  `<>`. After entering it, we can see that these brackets are not escaped and a query is executed.

Our next step will be to find the working tag. Let's use the [cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) from PortSwigger and BurpSuite.

Let's intercept the request using BurpSuite and perform a brute force attack using Intruder. Copy the tags from the cheat sheet and paste them into the dictionary. Then start the attack.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS49.png" alt="XXS" />
</p>


<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS50.png" alt="XXS" />
</p>

We were able to successfully detect several tags. Let's select the `body` tag.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS51.png" alt="XXS" />
</p>

Let's try inserting the following payload

```python
<body onload=alert(1)>
```
<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS52.png" alt="XXS" />
</p>

The tag worked successfully, but now there is a problem with the attribute. This attribute is blocked by the firewall and returns an error. Let's find a working attribute in the same way.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS53.png" alt="XXS" />
</p>

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS54.png" alt="XXS" />
</p>

After waiting a little while, we managed to find a couple of attributes, but let's take a look at what the task description says.

> Your solution must not require any user interaction.

This means that most attributes become unusable for us because they require some action from the user.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS55.png" alt="XXS" />
</p>

But the `onresize` tag may suit us.

The `onresize` event occurs when the browser window has been resized.

Let's try our payload in action 

```
<body onresize=print(1)>
```

The payload worked, but the window confirming XXS did not appear.

If we make the FireFox tab larger or smaller, the attack will work, but this is not what we expect, so we use the exploit button at the top.

Let's use this payload

```python
<iframe src="https://ID.web-security-academy.net/?search=search=%22%3E%3Cbody%20onresize=print()%3E"onload=this.style.width='100px'>
```

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS56.png" alt="XXS" />
</p>

Since the page is controlled by the attack, we can use `<iframe>` and the onload attribute in it. 
