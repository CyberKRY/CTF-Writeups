# Description

This lab contains a DOM-based cross-site scripting vulnerability in a AngularJS expression within the search functionality.

AngularJS is a popular JavaScript library, which scans the contents of HTML nodes containing the `ng-app` attribute (also known as an AngularJS directive). When a directive is added to the HTML code, you can execute JavaScript expressions within double curly braces. This technique is useful when angle brackets are being encoded.

To solve this lab, perform a cross-site scripting attack that executes an AngularJS expression and calls the `alert` function.

# Solution

`AngularJS` is an outdated JavaScript framework for creating dynamic single-page applications (SPAs). It actively uses two-way data binding, templates, and directives, which makes it vulnerable to incorrect filtering of user input.

Let's go to the lab and enter our input data to verify the input. 

If we look at the Developer Tools, we can see that our body is included in ng-app, which prompts us to try whether we can use ``{{}}`` to embed our payload.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS36.png" alt="XXS" />
</p>

Using {{ 2 + 2 }}, we can see that ng-app is running, and we can try to create our function.

If we try to find angularjs methods scope, we will find methods that can call a function.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS37.png" alt="XXS" />
</p>

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS38.png" alt="XXS" />
</p>

Using this payload, we will execute our alert.

```
{{ $emit.constructor('alert()')() }}
```
<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS39.png" alt="XXS" />
</p>

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS40.png" alt="XXS" />
</p>
