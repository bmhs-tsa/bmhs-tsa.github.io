---
title: "Matryoshka Doll"
description: "picoCTF writeup by loogalicious"
date: 2021-04-02T16:32:39-06:00
categories: [
    "picoCTF"
]
tags: [
    "Forensics",
    "loogalicious"
]
---

### Challenge Rating
* Artificialness: `6/10`
* Skill: `4/10`
* Time: `10 minutes`

*Learn more about how we rate challenges [here](/post/rating).*

### What software do you need?
Any type of zip extraction, [7zip](https://www.7-zip.org/) or [WinRAR](https://www.win-rar.com/start.html) work fine.

### What do you need to know in order to solve it?
The aspect of this challenge is to find hidden files inside image files.

### What do you actually do?
Archive the original file with (in my case) WinRAR, then extract it.
In the extracted file, you'll find another file and so on, so forth.
Continue the archive-extract until you find a text file which
contains the flag.