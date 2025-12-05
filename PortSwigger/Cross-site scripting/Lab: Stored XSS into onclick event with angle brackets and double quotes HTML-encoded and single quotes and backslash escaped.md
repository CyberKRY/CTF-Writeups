# Description

This lab contains a stored cross-site scripting vulnerability in the comment functionality.

To solve this lab, submit a comment that calls the alert function when the comment author name is clicked.

# Solution

Let's leave a comment and see where our input ends up.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS70.png" alt="XXS" />
</p>

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS71.png" alt="XXS" />
</p>

Our input goes into the href and onclick attributes. Let's try to exit our onclick tag using a bracket.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS72.png" alt="XXS" />
</p>

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS73.png" alt="XXS" />
</p>

We can see that our bracket has been escaped, and even if we try to escape / slash, we will not be able to do so.

If we replace single quotes with their HTML encoding ('), we can tell the browser that this character is part of the code, which allows our JavaScript insertion to be interpreted correctly.

We use this payload

```python
https://test4.com?&apos;-alert(1)-&apos;
```

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS74.png" alt="XXS" />
</p>

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS75.png" alt="XXS" />
</p>
After we click on our name, our attack will be triggered.

