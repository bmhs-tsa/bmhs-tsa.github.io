---
title: "Web Gauntlet 2"
description: "picoCTF writeup by Wakeful Cloud"
date: 2021-03-30T23:26:46-06:00
categories: [
  "picoCTF"
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
* Time: `2-3 hours`

*Learn more about how we rate challenges [here](/post/rating).*

## Preface
This is the followup challenge to [Web Gauntlet](https://play.picoctf.org/practice/challenge/88).

## What are SQL injections?
SQL injections occur when a server improperly handles user input in such a
way that the user can write SQL queries and run them on the server. For example,
lets say we're running a simple web server that performs a username/password
lookup:
```sql
SELECT username, password FROM users WHERE username='[USERNAME INPUT]' AND password='[PASSWORD INPUT]';
```
Even if you don't know SQL, you can probably figure out that we're querying the
database for users with a `username` equal to `[USERNAME INPUT]` and a `password`
equal to `[PASSWORD INPUT]`. Well, say an attacker were to enter `'`. Just a 
single-quote. What do you think would happen? If the server was programmed
correctly, it would escape the quote or filter it out resulting in the following
query:
```sql
SELECT username, password FROM users WHERE username='''' AND password='[PASSWORD INPUT]';
```
*Note: you escape single-quotes by doubling them up in SQLite.*

But if the server simply concatenates or adds strings together to form a query
such as with the below pseudocode:
```javascript
"SELECT username, password FROM users WHERE username='" + username + "' AND password='" + password + "';"
```
This would produce the following query:
```sql
SELECT username, password FROM users WHERE username=''' AND password='[PASSWORD INPUT]';
```
*Note: this is actually an invalid query but bare with me.*

Notice how the syntax highlighting is different than when the server properly
escapes the single-quote? That's because we now have a single string starting
with an escaped quote (`'' AND password=`). This is called an injection
vulnerability because a crafty attacker could format the username or password
in such a way that when the server concatenates the user-input together, it 
generates a totally different, malicious query.

## Solving the challenge
One important thing to note about this challenge is that our user-input cannot 
contain any of the following `or and true false union like = > < ; -- /* */ admin`
(Which can be found on the `filter.php` page).

### Setting the username
So first we need to figure out how to query the `admin` user without actually 
typing `admin` (Because it's filtered). Luckily, SQLite (The flavor of SQL this
challenge uses) supports [string concatenation](https://www.techonthenet.com/sqlite/functions/concatenate.php)
(*Adding* two or more strings together). So now we can type `admin` with the 
following for the `username`: `admi'||'n` which will turn into:
```sql
SELECT username, password FROM users WHERE username='admi'||'n' AND password='[PASSWORD INPUT]';
```
Sweet!

### Setting the password
We don't know the password and this challenge isn't about getting the password,
so we need to get more creative - think about ways of removing the `AND` part
of the query.

Remember how earlier when the attacker entered a single-quote (`'`) it
generated an invalid SQL query because the string kind of *bridged* the 2 user
inputs? Well if we do something with that string, then that eliminates the
`password=` part of the query meaning we don't have to guess or crack the
password.

My solution is far from elegant but I decided to kill 2 birds with one stone by using the length of the *bridged* string using the [SQLite `LENGTH` function](https://www.w3resource.com/sqlite/core-functions-length.php)
in combination with the [SQLite `CHAR` function](https://www.w3resource.com/sqlite/core-functions-char.php)
to generate the `n` in `admin` via [character codes](https://en.wikipedia.org/wiki/ASCII).

In essence, the *bridged* string is ` AND password=` (14 characters long) and
the ASCII character code for `n` is 110. So then I multiplied 14 by 110 which
is 1540. Finally I concatenated an empty string at the end to make sure there
wasn't an unmatched single-quote. When the server executes the query, it will
undo the multiplication and return the correct character code. I know this
sounds confusing but it's somewhat simple when you see it written out:
```sql
SELECT username, password FROM users WHERE username='admi'||CHAR(1540/LENGTH(' AND password='))||'';
```
So if you set the `username` to `admi'||CHAR(1540/LENGTH(` and the password to
`))||'`, you should see the flag on `filter.php`. Again, not the most elegant
solution. If you want to see a more elegant solution, check out the sequel to
this challenge, [Web Gauntlet 3](/post/web-gauntlet-3).