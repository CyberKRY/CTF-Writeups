# Description
This lab contains a DOM-based cross-site scripting vulnerability in the stock checker functionality. It uses the JavaScript document.write function, which writes data out to the page. The document.write function is called with data from location.search which you can control using the website URL. The data is enclosed within a select element.

To solve this lab, perform a cross-site scripting attack that breaks out of the select element and calls the alert function.

# Solution

By clicking on any product, we can see a list at the bottom where we can check the availability of the product.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS29.png" alt="XXS" />
</p>

Let's go to the developer tools and look at the JavaScript code. This code creates a drop-down list 
```<select>``` with stores in London, Paris, and Milan.
It takes the storeId parameter from the page address (for example, ```&storeId=Paris```), makes this option selected, and adds the rest below.
If there is no parameter, it simply shows all cities.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS30.png" alt="XXS" />
</p>

Let's try adding our own parameter

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS31.png" alt="XXS" />
</p>

After that, our parameter is entered into the table. 

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS32.png" alt="XXS" />
</p>

Now our parameter has been added to the ```<option>``` tag.

Now our task is to exit the tag and create a new attribute. 

```
https://0a15007804cf99e580f6085a009600c6.web-security-academy.net/product?productId=1&storeId=XXS</select> <img src=1 onerror=alert(123)>
```
And our payload worked. We managed to exit the <select> tag and create a new attribute that will trigger an alert.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS33.png" alt="XXS" />
</p>

And our payload worked. We managed to exit the <select> tag and create a new attribute that will trigger an alert.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS34.png" alt="XXS" />
</p>
