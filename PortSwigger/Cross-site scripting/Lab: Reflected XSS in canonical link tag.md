# Description

This lab reflects user input in a canonical link tag and escapes angle brackets.

To solve the lab, perform a cross-site scripting attack on the home page that injects an attribute that calls the alert function.

To assist with your exploit, you can assume that the simulated user will press the following key combinations:

```python
ALT+SHIFT+X

CTRL+ALT+X

Alt+X
```

> Please note that the intended solution to this lab is only possible in Chrome.


Since we don't have a search field, we can create our own attribute in the link tag.
Let's see what happens.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS65.png" alt="XXS" />
</p>

Our attribute is enclosed in href, so we need to exit it using a bracket. 

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS66.png" alt="XXS" />
</p>

we have exited href now we can use our onclick attribute 

```python
'onclick=alert(1)
```

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS67.png" alt="XXS" />
</p>

Since our attribute is enclosed in quotation marks, we can try adding a quotation mark at the beginning of the alert.

```python
'onclick='alert(1)
```

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS68.png" alt="XXS" />
</p>

Well, since the onclick attribute itself only works with visible objects, we need to use accesskey to activate the URL. Let's set the value x as written in the task.

```python
'accesskey='x'onclick='alert(123)
```

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS69.png" alt="XXS" />
</p>

And now, when we press Alt x, the attack will work.
