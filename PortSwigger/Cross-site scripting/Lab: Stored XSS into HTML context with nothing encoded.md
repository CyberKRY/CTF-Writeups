# Description

This lab contains a stored cross-site scripting vulnerability in the comment functionality.

To solve this lab, submit a comment that calls the `alert` function when the blog post is viewed.

## Solution
By going to the website, we can access any post. Scrolling down the page, we see a comment submission form, which potentially allows us to carry out our attack.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS3.png" alt="XXS" width="600" />
</p>

We can use this payload:

```
<script>alert(123)</script>
```
<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS4.png" alt="XXS" width="600" />
</p>

After posting, the comment will remain on the page, and anyone who visits this page will be vulnerable to attack.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS5.png" alt="XXS" width="600" />
</p>
