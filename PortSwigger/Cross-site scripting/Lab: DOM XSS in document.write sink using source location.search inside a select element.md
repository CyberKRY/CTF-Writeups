# Description
This lab contains a DOM-based cross-site scripting vulnerability in the stock checker functionality. It uses the JavaScript document.write function, which writes data out to the page. The document.write function is called with data from location.search which you can control using the website URL. The data is enclosed within a select element.

To solve this lab, perform a cross-site scripting attack that breaks out of the select element and calls the alert function.

# Solution

By clicking on any product, we can see a list at the bottom where we can check the availability of the product.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS29.png" alt="XXS" />
</p>


Let's go to the developer tools and look at the JavaScript code This code creates a drop-down list <select> with stores in London, Paris, and Milan.
It takes the storeId parameter from the page address (for example, &storeId=Paris), makes this option selected, and adds the rest below.
If there is no parameter, it simply shows all cities.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS30.png" alt="XXS" />
</p>

