---
title: "crackme-py"
description: "picoCTF writeup by superamario64"
date: 2021-04-09T16:06:11-06:00
categories: [
  "picoCTF"
]
tags: [
  "Reverse Engineering",
  "superamario64"
]
---

### Challenge Rating
* Artificialness: `5/10`
* Skill: `3/10`
* Time: `5-10 minutes`

*Learn more about how we rate challenges [here](/post/rating).*

The CTF starts out by giving you a python file called `crackme.py`. When you run 
it, it just takes two numbers as input and outputs the largest. Not very helpful 
for finding flags, unless...

### How do we find the flag?

When you open the file in any text editor, you'll see that there's more to this 
program than it seems. There's a variable called `bezos_cc_secret` that seemingly 
has gibberish in it. But there's another function, never used in the entire 
program, called `decode_secret` which (you guessed it) decodes `bezos_cc_secret`. 
Just call the function at the end of the program and run it again and you will 
have your flag.
