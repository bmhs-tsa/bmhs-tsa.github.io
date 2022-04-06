---
title: "Get Ahead"
description: "picoCTF writeup by Wakeful Cloud"
date: 2021-03-30T22:38:13-06:00
categories: [
  "picoCTF 2021"
]
tags: [
  "Web Exploitation",
  "Wakeful Cloud"
]
---

### Challenge Rating
* Artificialness: `5/10`
* Skill: `3/10`
* Time: `5 minutes`

*Learn more about how we rate challenges [here](/post/rating).*

## What are HTTP verbs?
Normally when you access a website, your browser makes an HTTP request to `GET`
content from the server. However, your browser isn't always fetching data from a
server, sometimes it `POST`s data too (Such as when logging in to a website). HTTP
verbs are the standardized method for telling the server what to do. The most common
HTTP verbs are:
* `GET`: fetch data from a server (eg: retrieve an HTML file)
* `HEAD`: check what the server will do before making a `GET` request (eg: check response size before fetching something)
* `POST`: post data to a server (eg: log in to a website)
* `PUT`: update an existing resource
* `DELETE`: delete a resource

## Solving the challenge
Although the `Get` in `Get Ahead` may throw you off, remember that web browsers
make `GET` requests by default so this challenge may be a little too easy if
all you had to do was just visit the link with a browser and open [the inspector](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_are_browser_developer_tools).
Instead, focus on the `Ahead` part - as in a `HEAD` request. If you use a tool such
as [Postman](https://www.postman.com), this is fairly easy to do:
![Postman screenshot](/post/get-ahead/postman.png)

Please note that the flag is a header, not the response body.