# Description

This lab contains a stored cross-site scripting vulnerability in the comment functionality. To solve this lab, submit a comment that calls the alert function when the comment author name is clicked.

# Solution

When entering any message, we can write a comment. Let's send the data to see where it is entered. 

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS22.png" alt="XXS" />
</p>

By entering the developer tools and finding our input, we can see that the website we specified is included in the href parameter. Now that the comment has been sent, we can access this website by clicking on the user's nickname.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS23.png" alt="XXS" />
</p>

And this prompts us to attack this point by changing the site to our payload. 

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS24.png" alt="XXS" />
</p>

Now, if we click on the user's nickname, XXS will work.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS25.png" alt="XXS" />
</p>
