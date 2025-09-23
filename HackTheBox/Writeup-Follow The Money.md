# Follow The Money – Write-up

## Short description:

This challenge focuses on cryptocurrency forensics: tracing transactions, clustering wallets, identifying mixers, and performing KYC/OSINT analysis to uncover a suspect’s identity and location.

## FLAG 1 – Primary Funding Source Exchange

**Task**:
Analyze transaction clusters to identify the primary funding source exchange.

**Approach**:

We scanned the suspect wallet address

```python
bc1q7x4a9m2k8j5p3r9s1t6u8v2w7x4z9a1b2c3d4e5
```

using tools such as ```Chainalysis```, ```Crystal```, ```Arkham```.

**Finding**:

The cluster clearly showed deposits from Binance.

**Answer**:

```python
Binance
```
## FLAG 2 – Total Victim Payment Amount

**Task**:

Calculate the total BTC amount of seven victim payments (T1-CHARLIE through T7-CHARLIE) in the Evidence Terminal.

**Approach**:

In the Evidence Terminal under Transaction Analysis, we added the seven values with a calculator.

**Answer**:

```python
0.01891004
```
## FLAG 3 – Obfuscation Method Detection

**Task**:

Identify the mixing protocol used for the transaction hash:

```python
q7r8s9t0u1v2345678901234567890123456789012
```

**Approach**:

Using OXT Advanced, under the Clustering → Output Clustering section we found:

12% mixing

Wallet ID: ```Wasabi_Mixing_Pool```

**Answer**:

```python
Wasabi
```
## FLAG 4 – Digital Identity Attribution

**Task**:

Determine the email address associated with the primary suspect through KYC analysis.

**Approach**:

KYC scan of the address returned:

```python
Email: j****.crypto.****@proton.me
Name: James M****** C***
Registration: 2024-02-15
KYC Level: Level 2
```

Given the name pattern and experimenting with the hidden characters, we deduced:

**Answer**:

```python
james.crypto.2024@proton.me
```
## FLAG 5 – Real-World Identity Attribution

**Task**:

Determine the suspect’s real name.

**Approach**:

Using Crystal Analytics “Name Verification,” we found:

Name Verification: ```James Mitchell Chen``` (92% match)


**Answer**:

```python
James Mitchell Chen
```
## FLAG 6 – Geographic Attribution

**Task**:

Determine precise operational coordinates.

**Approach**:

Crystal Analytics showed:

San Francisco Bay Area (37.77****, -122.41****)


Looking up the coordinates, the precise point corresponds to:

**Answer**:

```python
37.774929, -122.419416
```
## Challenge Completed.
