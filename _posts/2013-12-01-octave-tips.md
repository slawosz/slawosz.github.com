---
title: "Octave tips"
layout: post
old_url: /2013/12/01/octave-tips
categories: ['published']
---

I am taking Coursera Machine Learning course, which requires octave usage for programming assighments. Here are some tricks I learned:

*Getting matrix without first column:*

```
octave:11> a = [1 2 3; 4 5 6]
a =

   1   2   3
   4   5   6

octave:12> a(:,2:end)
ans =

   2   3
   5   6
```
