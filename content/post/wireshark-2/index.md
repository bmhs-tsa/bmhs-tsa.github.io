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

## Initial analysis
Starting off I ([loogalicious](https://github.com/loogalicious)) opened the
file in Wireshark. After browsing through it, I noticed 90 or so flags in
the responses from `http://18.217.1.57/flag` and the AWS EC2 token in packet
`4344` caught my eye:
![Wireshark token screenshot](/post/wireshark-2/wireshark-token.png)

After [Wakeful-Cloud](https://github.com/wakeful-cloud) and I made a script to
extract all of the flags, I realized they were red herrings. Then I found the
DNS queries to `*.reddshrimpandherring.com` and looked through the given website but 
found nothing. This is the point when Wakeful-Cloud switched from helping me to
actually working to solve the challenge.

## DNS queries
The first thing I (Wakeful-Cloud) did towards solving this challenge was to sort
by packet protocol and just familiarize myself with the capture. This also helps
to weed out any *noise* that may have been generated (Since the *noise* is often
generated from a different application/protocol). There were a few odd things
that I noticed such as an enormous amount of **invalid** DNS queries to
`*.reddshrimpandherring.com`:
![Wireshark DNS screenshot](/post/wireshark-2/wireshark-dns-1.png)

What really bothered me is why there were so many invalid queries. I'd expect
that software designed to make lots of DNS queries would have some kind of
[back-off](https://en.wikipedia.org/wiki/Exponential_backoff) algorithm but
whatever was making these requests clearly did not. The next thing I did was
sort by time and just scroll through all the packets. Eventually, I noticed
packet `3969`:
![Wireshark packet 3969 screenshot](/post/wireshark-2/wireshark-dns-2.png)

If you're a web developer, you may recognize the 2 trailing equal-signs as the
padding used by [base 64](https://en.wikipedia.org/wiki/Base64). At this point,
it seemed quite obvious to me that in order to solve the challenge, I'd have to
combine all the invalid DNS queries together and decode it. In order to extract
all the queries, I used [T-Shark](https://www.wireshark.org/docs/man-pages/tshark.html)
and the filter `dns.qry.name matches  ".*\.reddshrimpandherring\.com$" && ip.src == 18.217.1.57`
(You can use this filter in the GUI too). The command to dump all the DNS
queries I used is:
```bash
# Bash
tshark -r ./shark2.pcapng -Y 'dns.qry.name matches  ".*\.reddshrimpandherring\.com$" && ip.src == 18.217.1.57' -T fields -e dns.qry.name | uniq > dns.txt

# Powershell
tshark -r ./shark2.pcapng -Y 'dns.qry.name matches  ".*\.reddshrimpandherring\.com$" && ip.src == 18.217.1.57' -T fields -e dns.qry.name | Get-Unique > dns.txt
```

## Processing the queries
Now that we have a file (`dns.txt`) filled with unique DNS requests, we need
to combine the subdomains and parse the base 64. To do this, I used the below
JS script:
```javascript
//Imports
const {readFileSync, writeFileSync} = require('fs');

//Read the file
const raw = readFileSync('./dns.txt', 'utf-8');

//Split each line
const lines = raw.split('\n').filter(line => line != '');

//Combine
let base64 = '';
for (const line of lines)
{
  //Extract the subdomain
  const subdomain =  /([^.]+)/.exec(line);

  //Append the subdomain
  if (subdomain != null)
  {
    base64 += subdomain[0];
  }
}

//Interpret the base 64
const data = Buffer.from(base64, 'base64');

//Log
console.log(data.toString());
```

*Note: this script assumes your file uses Linux-style new-lines. If you're not,
be sure to update line 8 from `'\n'` to whatever new-line-characters you're using.*

If you run this, it should write the flag to the console.
