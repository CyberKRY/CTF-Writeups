# Write-up: Impossible Challenge (TryHackMe)
## 🧩Challenge Overview
In this challenge, we’re given a file named Impossible_1587053656818.zip and a suspicious encoded string. Our task is to analyze the provided data, decode it correctly, and uncover the flag. The challenge requires a bit of creativity, some encoding tricks, and even steganography knowledge.

##  Step 1: Download and Analyze the File
We start by downloading the provided ZIP file:
Impossible_1587053656818.zip
Also, we are given a strange string:
```python
qo qt q` r6 ro su pn s_ rn r6 p6 s_ q2 ps qq rs rp ps rt r4 pu pt qn r4 rq pt q` so pu ps r4 sq pu ps q2 su rn on oq o_ pu ps ou r5 pu pt r4 sr rp qt pu rs q2 qt r4 r4 ro su pq o5
```
At first glance, this looks like it might be encoded multiple times.

## Step 2: Multi-layered Decoding
We use [CyberChef](https://gchq.github.io/CyberChef/) to decode the string using a sequence of operations:
* ROT13
* then ROT47
* then From Hex
* then From Base64

However, the output doesn’t seem helpful at this stage.

## Step 3: Unicode Steganography
Looking at the description of the challenge, we notice the word <b>Hmm</b> — this seems ordinary, but don’t type it manually. You must copy the word directly from the challenge description (it contains zero-width characters).

Now go to this tool:
[Unicode Steganography Tool](https://330k.github.io/misc_tools/unicode_steganography.html)

Paste the copied <b>Hmm</b> into both:
* Original Text
* Steganography Text

And decode.

This reveals a hidden message:
```python
password is ********
```
## Step 4: Extract the ZIP Archive
Now that we have the password, extract the archive using:
```python
7z e Impossible_1587053656818.zip
```
After successful extraction, you’ll receive the flag:
```python
You have solved the Impossible Challenge! Here is your flag THM{**********}
```
# Conclusion
This challenge was a clever test of attention to detail and knowledge of obscure encoding and steganography techniques. It’s a great reminder to always inspect everything, even something as simple as a word in the description. Don't trust your eyes — trust the bytes!
