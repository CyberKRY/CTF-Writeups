# Description

This lab demonstrates a reflected DOM vulnerability. Reflected DOM vulnerabilities occur when the server-side application processes data from a request and echoes the data in the response. A script on the page then processes the reflected data in an unsafe way, ultimately writing it to a dangerous sink.

To solve this lab, create an injection that calls the `alert()` function.

# Solution

As usual, let's enter our data and see where our input appears after writing test. We can see that our input is placed in h1, but the second question is how it gets there. We have already been given a hint: we can find the external file `/resources/js/searchResults.js`, which we can examine.

The most dangerous place is this:

```python
function search(path) {
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
            eval('var searchResultsObj = ' + this.responseText);
            displaySearchResults(searchResultsObj);
        }
    };
    xhr.open("GET", path + window.location.search);
    xhr.send();
```
```python
eval('var searchResultsObj = ' + this.responseText);
```
This line:

interprets the response text as JavaScript code

---

Let's intercept the request using BurpSuite.

If we enter our value in the search engine, it will appear in SearchTerm.

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS44.png" alt="XXS" />
</p>

Let's try to get out of it and use our payload.

```python
test"-alert()
```
<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS41.png" alt="XXS" />
</p>

After entering this, we see that our bracket has been escaped with the \ symbol.

---

Let's try to shield this sign in the same way using this payalod.

```python
test\"-alert()
```

Now we're actually out, but let's finish our quotation marks. 

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS42.png" alt="XXS" />
</p>


```python
test\"-alert()} //
```
And this is what our final query looks like

<p align="center">
  <img src="https://github.com/CyberKRY/CTF-Writeups/blob/main/PortSwigger/Cross-site%20scripting/Images/XXS43.png" alt="XXS" />
</p>
and we were able to go out and fulfill our alert


