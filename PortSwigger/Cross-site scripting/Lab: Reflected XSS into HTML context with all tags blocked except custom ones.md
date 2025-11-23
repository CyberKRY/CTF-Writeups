# Description 

This lab blocks all HTML tags except custom ones.

To solve the lab, perform a cross-site scripting attack that injects a custom tag and automatically alerts document.cookie.

# Solution

In the task description, we see that we need to use custom tags, such as `<exploit>`. If we enter h1 or any other HTML tag, it will be blocked by the firewall.

To solve this problem, we will use this payload.

```python
<exploit onfocus='alert(document.cookie)' id='x' tabindex='1'>
```

The `onfocu` event fires when an element receives focus, making it the currently active element ready for user interaction. This typically happens when a user clicks, tabs to, or touches the element.

`alert(document.cookie)` Calls up a cookie warning window

The `id` attribute serves as a singular identifier for an element in a document, facilitating its selection via CSS or JavaScript.

The `tabindex='1'` attribute dictates the sequence for keyboard focus when tabbing through page elements

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS59.png" alt="XXS" />
</p>

If we click on the bracket, a window will appear and the attack will actually work, but it still requires user interaction, so let's add `#x` to the end of the link.

```python
https://ID.web-security-academy.net/?search=%3Cexploit+onfocus%3D%27alert%28document.cookie%29%27+id%3D%27x%27+tabindex%3D%271%27%3E>#x
```

And it works. Now let's move on to the exploit section and insert this payload, and then send it. 

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS60.png" alt="XXS" />
</p>
