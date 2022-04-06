---
title: "Web Gauntlet 3"
description: "picoCTF writeup by Wakeful Cloud"
date: 2021-03-31T00:32:29-06:00
categories: [
  "picoCTF 2021"
]
tags: [
  "Web Exploitation",
  "Injection Vulnerability",
  "Wakeful Cloud"
]
---

### Challenge Rating
* Artificialness: `3/10`
* Skill: `6/10`
* Time: `30 minutes` (Assuming you already solved [Web Gauntlet 2](/post/web-gauntlet-2))

*Learn more about how we rate challenges [here](/post/rating).*

## Preface
This is the followup challenge to [Web Gauntlet 2](/post/web-gauntlet-2) which
itself is a followup to [Web Gauntlet](https://play.picoctf.org/practice/challenge/88). This article assumes you've already solved Web Gauntlet 2.

### What's different?
Not a whole lot - just the maximum user input length. Our previous exploit 
(`admi'||CHAR(1540/LENGTH(` and `))||'`) is 29 characters long. We only need to
shave off 4 characters to be within the 25 character limit, that shouldn't be
too hard.

### Condensing our payload
The first thing I noticed is that the *bridged* string (` AND password=`) contains 1 of the characters in `admin`: `a`. Since SQLite also has a
[`SUBSTR` function](https://www.w3resource.com/sqlite/core-functions-substr.php),
we can use that bridged text meaning we can eliminate `CHAR(1540/LENGTH())`.
This leaves us with:
```sql
SELECT username, password FROM users WHERE username=''||substr(' AND password=',7,1)||'dmin';
```
Now the total input is only 22 characters, 3 under the limit. Sweet! So if you 
set the `username` to `'||substr(` and the `password` to `,7,1)||'dmin`,
you should see the flag on `filter.php`.