---
title: "Mod 13"
description: "picoCTF writeup by Whynt"
date: 2021-04-02T16:58:00-06:00
categories: [
  "picoCTF"
]
tags: [
  "Cryptography",
  "Whynt"
]
---

# Mod 26
This is a rot 13 cypher!

## What is ROT 13?
ROT 13 is a cipher where every letter is rotated by 13 letters within the alphabet. It is a type of [caesar cipher](https://en.wikipedia.org/wiki/Caesar_cipher).

## How can this be reversed?
Rot 13 can be reversed the same way as it is encypted, by rotating the letters
by thirteen charecters. In Python we can simply take the [ASCII](https://en.wikipedia.org/wiki/ASCII)
value of each letter (take out special charecters and numbers),and add thirteen,
before using the modulo operator to `loop back`to the beginning of the alphabet.

```python3
ex = 'cvpbPGSarkggvzrVyygelebhaqfbsebghyLicInt'
letts = []
for i in range(len(ex)):
    letts.append(ex[i].lower())
    letts[i] = (ord(letts[i]) - 97 + 13) % 26 
    letts[i] = chr(letts[i] + 97)

print(letts)
```
The output will look something like:
```python3
['p', 'i', 'c', 'o', 'c', 't', 'f', 'n', 'e', 'x', 't', 't', 'i', 'm', 'e', 'i', 'l', 'l', 't', 'r', 'y', 'r', 'o', 'u', 'n', 'd', 's', 'o', 'f', 'r', 'o', 't', 'u', 'l', 'y', 'v', 'p', 'v', 'a', 'g']
```

Manually add back special charecters and capitalization and you will have your flag.

### Online Tools
You can also use the following online tools to accomplish the same thing:
* [dcode.fr/rot-13-cipher](https://www.dcode.fr/rot-13-cipher)
* [rot13.com](https://rot13.com)
* [cryptii.com/pipes/rot13-decoder](https://cryptii.com/pipes/rot13-decoder)
