# Description

This lab demonstrates a stored DOM vulnerability in the blog comment functionality. To solve this lab, exploit this vulnerability to call the `alert()` function.

# Solution

First, let's see where our input goes. Looking through the developer tools, we can also see the file loadCommentsWithVulnerableEscapeHtml.js.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS45.png" alt="XXS" />
</p>

We could find it when sending a comment. Of course, in real life, such a file may not exist, but since this is a training task, such a file is here. 

Let's take a look at the vulnerability in this code. 

```python
 function escapeHTML(html) {
        return html.replace('<', '&lt;').replace('>', '&gt;');
```

> <b>replace()</b> function to encode angle brackets

Based on this line, we can see that angle brackets are escaped, but there is a problem with the replace function; specifically, in this example, only the first brackets are escaped.

Let's send the text to `<h1>` 

```
<h1>KRY</h1>
```
<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS46.png" alt="XXS" />
</p>

After that, we can see that our first tag was successfully shielded, but the second one was not, which leads us to the idea of writing two tags at the beginning.

```
<h1><h1>KRY</h1>
```
And it worked. The first `<h1>` was successfully shielded, which ultimately allowed the second one to work.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS47.png" alt="XXS" />
</p>

Now we can carry out our attack. To do this, we no longer need to insert `<h1>`; we can simply insert <>.

```python
<><img src="1" onerror=alert()>
```
What ultimately works

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS48.png" alt="XXS" />
</p>
