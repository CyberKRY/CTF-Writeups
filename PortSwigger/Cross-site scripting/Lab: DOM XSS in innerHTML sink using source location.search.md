# Description 

This lab contains a DOM-based cross-site scripting vulnerability in the search blog functionality. It uses an `innerHTML` assignment, which changes the HTML contents of a `div` element, using data from `location.search`.

To solve this lab, perform a cross-site scripting attack that calls the `alert` function.

## Solution

When we enter the website, a search form appears where we can enter any data to track where it goes. 

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS11.png" alt="XXS" width="600" />
</p>

Open Developer Tools

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS12.png" alt="XXS" width="600" />
</p>

`innerHTML` interprets a string as HTML and creates DOM nodes from it. If `location.search` contains HTML/JS (like your `<img ... onerror=...>`), it will be turned into real elements on the page.

Using this, we can use this payload to carry out an attack.

```
<img src=1 onerror=alert(1)>
```
<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS13.png" alt="XXS" width="600" />
</p>

Why `onerror` fired specifically.
The browser tries to load the image from `src="1"`. The path `1` is not a valid URL, the load fails, and the `<img>` tag’s `onerror` handler runs. Since the `onerror` attribute contains the JavaScript `alert(1)`, it gets executed. In other words, your malicious code ended up in the DOM and executed in the page’s context.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS14.png" alt="XXS" width="600" />
</p
