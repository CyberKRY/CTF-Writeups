# Description

This lab contains a simple reflected cross-site scripting vulnerability in the search functionality.

To solve the lab, perform a cross-site scripting attack that calls the `alert` function.

<h4>To accomplish this task, we can use this payload. </h4>

```
<script>alert(123)</script>
```
<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS1.png" alt="XXS" />
</p>

<b>After clicking Search, the script ran and brought up a window in the browser.</b>

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS2.png" alt="XXS" width="700" />
</p>
