# Description

This lab has a simple reflected XSS vulnerability. The site is blocking common tags but misses some SVG tags and events.

To solve the lab, perform a cross-site scripting attack that calls the `alert()` function.

# Solution

By entering different tags into the search, we can see that most of them are blocked. Let's find tags that we can use.

First, let's intercept the request with Burp Suite. Then, let's send it to Intruder. And let's copy the tag dictionary from the [Cross-site scripting cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet).


Let's launch the attack and see that we managed to detect two tags: `<svg>` and `<animatetransform>`.


<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS61.png" alt="XXS" />
</p>

Then we start searching for attributes in the same way, copy the dictionary from the cheat sheet, and begin the attack.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS62.png" alt="XXS" />
</p>

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS63.png" alt="XXS" />
</p>

After the attack, we can see that we managed to detect one attribute, `onbegin`.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS64.png" alt="XXS" />
</p>

Now let's use this payload data.

```python
<svg><animatetransform onbegin='alert(1)'>
```
---

And we successfully complete the task

