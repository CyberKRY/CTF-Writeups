# Description 

This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript document.write function, which writes data out to the page. The document.write function is called with data from location.search, which you can control using the website URL.

To solve this lab, perform a cross-site scripting attack that calls the alert function.

## Solution

We need to perform an XXS attack via the search field. 

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS6.png" alt="XXS" />
</p>

I entered the word “test” so we can check where our input is being entered. 

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS7.png" alt="XXS" />
</p>

To do this, we need to open the Developer Tools. To do this, we need to hover over test, right-click, and select inspect. 

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS8.png" alt="XXS" />
</p>

As you can see, our input fits into img and the SearchTerms parameter. Let's try to bypass the filters and launch an attack.

To do this, we will try this payload

```
"><svg onload=alert(1)>
```

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS9.png" alt="XXS" />
</p>

And our DOM XXS attack was successful, and we get the output in the browser.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS10.png" alt="XXS" />
</p>
