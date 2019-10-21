---
title: "Find and replace using regexp and sed."
layout: post
redirect_from: /2013/11/12/find-and-replace-using-regexp-and-sed
read_time: short read
categories: ['published']
---

Sed is awesome tool and can help solving interesting problems
Ie. replacing 1.8 style ruby hashes to 1.9 style:

```bash
find -name '*.*rb' | xargs sed -i 's/:\([a-zA-Z_1-9]\+\) => /\1: /g'
```
