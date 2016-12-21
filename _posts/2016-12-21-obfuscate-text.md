---
layout: post
title: "Obfuscate Text"
modified: 2016-12-21 11:29:57 -0800
tags: [python,obfuscate]
image:
  feature:
  credit:
  creditlink:
comments:
share:
---

When re-writing my [resume](/about) I came to a little dilemma -- how do I hide my email and phone number from the evil spammers. First I thought I should write a JS script, but I don't know JS that well, and not sure if that would be very useful as an exercise. So, I present to you the `Python` text obfuscator. Well, not really - it just changes the characters with [HTML codes](http://www.ascii.cl/htmlcodes.htm). Because there is one-to-one mapping, any spammer could decode it into a real number or email. We are just counting on spammers' laziness.

{% gist zafartahirov/66cadd5140b13da26147922d3bd77de5 %}
