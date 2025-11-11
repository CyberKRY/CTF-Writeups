# Description

This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality where angle brackets are encoded. The reflection occurs inside a JavaScript string. To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the alert function.

# Solution
Once on the page, we have a search field where we enter “test” to check where our input is entered.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS26.png" alt="XXS" />
</p>

As we can see, our input goes into the JavaScript attribute. Our task is to exit this attribute by entering our payload and carrying out an attack.

```
test'; alert(1); //
```

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS27.png" alt="XXS" />
</p>

After sending, our request will look like this

```
var SearchTerms = 'test'; alert(1); //';
```
<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS28.png" alt="XXS" />
</p>
