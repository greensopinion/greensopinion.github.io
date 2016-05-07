---
layout: post
title: Horse Phone and Unicode at Tasktop
date: '2016-05-06T12:00:00.000-08:00'
author: David Green
tags:
  - Testing
  - Unicode
modified_time: '2016-05-06T12:00:00.000-08:01'
original_url: https://www.tasktop.com/blog/telephone-horse-and-unicode-at-tasktop/
comments: true
---

Horse Phone &#x265E;&#x260E;, a Tasktop meme that has significance far beyond its unassuming appearance.  A few lucky people have seen glimpses of this rare beast in demos at Tasktop - but what does it mean?

<img src="/images/blog/2016-05/horsephone.png" alt="Horse Phone" width="200" class="alignleft" /> Horse Phone &#x265E;&#x260E; is the tip of the proverbial unicode testing iceberg for Tasktop's connectors.  Our approach to testing integrations, something that we call the [Integration Factory](http://www.tasktop.com/blog/testing-api-economy/), involves thousands of automated tests.  In fact, we run more than 500,000 tests a day on our connectors.  With character encoding being one of the most problematic areas for bugs, a major component of this automated testing approach involves testing for correct handling of unicode characters.  Not only do we test with real language data ranging from Arabic (ar) to Chinese (zh), our test data includes problematic characters such as `!@#$%&` and characters from various [Unicode Planes](https://en.wikipedia.org/wiki/Plane_%28Unicode%29), taking into account semantic equality with [unicode normalization](http://unicode.org/faq/normalization.html).

So when you see that rare beast, the Horse Phone, you know that it means that we're testing connectors ensuring that no &#x260E; or &#x265E; goes awry.
