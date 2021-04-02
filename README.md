# Website
[![GitHub Pages Status](https://img.shields.io/github/workflow/status/bmhs-tsa/bmhs-tsa.github.io/gh-pages?style=for-the-badge)](https://github.com/bmhs-tsa/bmhs-tsa.github.io/actions/workflows/gh-pages.yml)

The source for [bmhstsa.com](https://bmhstsa.com).

## Development

### Adding a page
1. Install [Hugo](https://gohugo.io)
2. Add a post: `hugo new post/[NAME]/index.md` (Name should be `lowercase-hyphen-separated`)
3. Edit the markdown (Located at `content/post/[NAME]/index.md`)
   1. Make sure your metadata section looks similar to:
```markdown
---
title: "{Challenge Name}"
description: "picoCTF writeup by {Username}"
date: {Let Hugo generate this for you}
categories: [
  "picoCTF"
]
tags: [
  "{Title case CTF type}",
  "{Username}"
]
---
```
4. Start the development server to preview changes: `hugo server -D` (And open the URL Hugo outputs in your browser)