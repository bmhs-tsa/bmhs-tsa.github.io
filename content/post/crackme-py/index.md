---
title: "crackme-py"
description: "picoCTF writeup by superamario64"
# CHANGE DATE
date: 2021-04-10T16:46:31-06:00
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
* Skill: `2/10`
* Time: `5-10 minutes`

*Learn more about how we rate challenges [here](/post/rating).*

This challenge first looks like just a program to display the largest of two numbers. However, if you open up the source code, you'll notice that there's more to it.

There's another function called `decode_secret`, which (you guessed it) decodes the secret flag in the variable `bezos_cc_secret`. This function is never called in the entire program, but it's very easy to go into the source code and call it at the end of the program. 