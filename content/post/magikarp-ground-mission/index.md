---
title: "Magikarp Ground Mission"
description: "picoCTF writeup by superamario64"
date: 2021-04-02T16:46:31-06:00
categories: [
  "picoCTF"
]
tags: [
  "General Skills",
  "superamario64"
]
---

### Challenge Rating
* Artificialness: `4/10`
* Skill: `3/10`
* Time: `5-10 minutes`

*Learn more about how we rate challenges [here](/post/rating).*

## What is SSH?

SSH (Secure SHell) is a protocol for securely transmitting commands to a remote 
computer. It makes it possible to access files, run commands, and more, just from 
inside the terminal. More information can be found 
[here](https://searchsecurity.techtarget.com/definition/Secure-Shell).

## Doing the challenge

Pico already gives you the command to use. When you first log in, the directory 
you're in has `1of3.flag.txt` and `instructions-to-2of3.txt`. Clearly, the flag 
is split up into three different files. Use the `cat` command to view these files 
and follow the instructions to get the rest of the flag. Combine each of the 
files in order (`1of3.flag.txt`, `2of3.flag.txt`, and `3of3.flag.txt`) to get the 
flag!
