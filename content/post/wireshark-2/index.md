---
title: "Wireshark 2"
description: "picoCTF writeup by loogalicious and Wakeful Cloud"
date: 2021-04-09T15:58:15-06:00
categories: [
  "picoCTF"
]
tags: [
  "Forensics",
  "loogalicious",
  "Wakeful Cloud"
]
---

### Challenge Rating
* Artificialness: `7/10`
* Skill: `5/10`
* Time: `3 hours`

*Learn more about how we rate challenges [here](/post/rating).*

This is another Wireshark challenge, expanding on the [first challenge](/post/wireshark-1).
Wireshark is a piece of software used to monitor network traffic.

Starting off I, loogalicious, opened the file in Wireshark. After browsing 
through it, the 90 or so flags in HTTP requests and the token caught my eye. 
After Wakeful Cloud and I made a script to extract all of the flags, we realized
 they were red herrings. Then I found the DNS queries for [a website](http://reddshrimpandherring.com/)
 that was another red herring.  