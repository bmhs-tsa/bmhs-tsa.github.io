---
title: "X Marks the Spot"
description: "picoCTF writeup by Wakeful Cloud"
date: 2021-04-02T23:39:19-06:00
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
* Skill: `7/10`
* Time: `4-6 hours`

*Learn more about how we rate challenges [here](/post/rating).*

## Preface
This write-up assumes you understand what an injection vulnerability is. If you
don't you, should check out [Web Gauntlet 2](/post/web-gauntlet-2).

## What is XPath?
[XPath](https://en.wikipedia.org/wiki/XPath) is the standard way of querying
data from an XML document (Similar to SQL). There are a few basic things you
need to know about XPath:
1. Think of XPath queries like a file-path (`/` separated, items are written in
a descending, hierarchical-order)
1. A `/` represents the root element
2. A `//` represents any element in any location
3. You can extract an element's content with `{Element}/text()`
(eg: `//user/name/text()` equals `guest`)
5. You can get a string's length with [`string-length`](https://developer.mozilla.org/en-US/docs/Web/XPath/Functions/string-length)
(eg: `string-length('abc')` equals `3`)
6. You can get a substring with [`substring`](https://developer.mozilla.org/en-US/docs/Web/XPath/Functions/substring)
(eg: `substring('abc', 2, 1)` equals `b`; note: **XPath is one-indexed**)
8. You filter elements with `[{Condition}]`
   1. You can filter by position with `[position() {Operator} {Number}]`
(eg: `[position() = 2]`; note: **XPath is one-indexed**)
   2. You can filter by string-matches with `['A' = 'B']`
   3. You can join different conditions together with `or` or `and`

## Solving the challenge

### Probable Query
In order to develop an injection, it helps to have a query (Although it likely
won't be the same as the actual query picoCTF uses). To develop it, think about
what conditions the server uses to find a user element: does the user's name equal
a user-controlled valued and does the user's password equal another user-controlled
value? The probable query I developed was:
```css
//user[name/text()='[USERNAME INPUT]' and pass/text()='[PASSWORD INPUT]']
```

### Developing an injection
Now that we have a probably query, we can start messing around with single-quote
injection to try to escape the user-controlled strings. I found using `abc` as
the username and `' or {CONDITION} and 'a'='a` as the password allows for us to
run arbitrary boolean expressions on the server (Note that `You're on the right path.`
indicates `true` while `Login failure.` indicates `false`). While this doesn't
allow us to return strings, it's a start.

### Exfiltrating data
We know we can evaluate a boolean expression but how can we extract a string? The 
answer is by testing each character in a string until we get the correct 
character and with enough characters, the entire string.

Initially it may seem like this a place where binary search would work well - we
could possibly check if the nth character code is greater than the a certain
number however the [`fn:string-to-codepoints`](https://www.oreilly.com/library/view/xslt-2nd-edition/9780596527211/re156.html)
function is disabled. Therefore, we will have to go through each character 
sequentially (Linear search). This is the part where this challenge gets very painful. The good news is that we can reasonably assume all characters are inside
of the [printable ASCII range](https://flaviocopes.com/printable-ascii-characters/)
(97 characters) which is far easier to guess than say [Unicode](https://en.wikipedia.org/wiki/Unicode)
([143,859 characters](https://www.unicode.org/versions/Unicode13.0.0/)).

To expedite this process, I developed a mini library for data extraction 
specifically for this challenge. You can view the source code
[here](https://gist.github.com/Wakeful-Cloud/82f56b039d37d8db291fd12ea6de8f15).
This took a long time to extract because I followed multiple incorrect paths
(Hence the flag's message); but I did find [this XML playground](https://cutt.ly/9xhyyyR)
to be extremely helpful for constructing XPath queries. Eventually, I was able
to extract the below XML document (And the flag too):
```xml
<db>
  <poems>
    <poem>
      <author>Robert Frost</author>
      <title>The Road Not Taken</title>
      <text>[UNKNOWN]</text>
    </poem>

    <poem>
      <author>William Carlos Williams</author>
      <title>The Red Wheelbarrow</title>
      <text>[UNKNOWN]</text>
    </poem>

    <poem>
      <author>Pablo Neruda</author>
      <title>Oda a la papa</title>
      <text>[UNKNOWN]</text>
    </poem>

    <poem>
      <author>Gwendolyn Brooks</author>
      <title>We Real Cool</title>
      <text>[UNKNOWN]</text>
    </poem>

    <poem>
      <author>William Blake</author>
      <title>The Tyger</title>
      <text>[UNKNOWN]</text>
    </poem>
  </poems>

  <users>
    <user>
      <name>guest</name>
      <pass>thisisnottheflag</pass>
    </user>

    <user>
      <name>bob</name>
      <pass>thisisnottheflageither</pass>
    </user>

    <user>
      <name>admin</name>
      <pass>picoCTF{h0p3fully_u_t0ok_th3_r1ght_xp4th_f0505d9c}</pass>
    </user>
  </users>
</db>
```

*Note: `[UNKNOWN]` represent strings that are very long and would take a long time to extract; you can probably make an educates guess though.*

### Data verification
While extracting the data is very time-consuming and requires potentially
thousands of HTTP requests, verifying the data is easy. For example, here's
the login credentials to verify the flag (Which could change if you're reading
this far in the future):
* Username: `abc`
* Password: `' or /db/users/user[position()=3]/pass/text() = 'picoCTF{h0p3fully_u_t0ok_th3_r1ght_xp4th_f0505d9c}' and 'a'='a`