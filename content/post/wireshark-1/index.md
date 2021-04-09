---
title: "Wireshark Doo Dooo Do Doo"
description: "picoCTF writeup by loogalicious"
date: 2021-04-09T15:48:38-06:00
categories: [
    "picoCTF"
]
tags: [
    "Forensics",
    "loogalicious"
]
---

### Challenge Rating
* Artificialness: `7/10`
* Skill: `4/10`
* Time: `15-30 minutes`

*Learn more about how we rate challenges [here](/post/rating).*

This challenge is a Wireshark-based challenge, which involves analyzing network
 traffic through Wireshark. 

### What software do you need?
[Wireshark](https://www.wireshark.org/) and a [ROT13](https://rot13.com/) decoder,
 which you can find online.

### What do you need to know in order to solve it?
You need to know how to export Wireshark objects into HTTP and possibly how to 
recognize ROT13 encryptions if you don't want to spend 20 minutes looking up 
different decryption websites.

### What do you actually do?
Open the file in Wireshark, first of all. Then `Export Objects` into `HTTP` and 
click `Save All`. It'll put all of the HTTP requests, which in this case, 
involves text files. Go into the destination of your choice, mine was Downloads
, and open the `%5c` (which is a [percent-encoded](https://en.wikipedia.org/wiki
/Percent-encoding) character) folder in a text editor of your choice. The text 
is encoded in ROT13, which I encoded after trying several other algorithms. The
 encrypted text contains the flag. 

