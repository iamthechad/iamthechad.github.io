---
title: JUnit Just Wants To Be Your Friend
excerpt: My co-worker tests his code with the `toString()` and visual grep method. I'm not sure he would have ever caught the bug on his own.
tags: development java testing
categories: Development
classes: wide
redirect_from: "/junit-just-wants-to-be-your-friend-8b42a97a5bf7"
---

This is why you’re not allowed to have alcohol in the workplace.

**Co-Worker (who does not believe in unit tests)**: Okay. I just checked my changes in.

**Me**: There’s a bug in your code.

**Co-Worker**: How do you know? I just checked it in!

**Me**: It broke my unit tests. It looks like you’re not setting the `flibbergibbet` value correctly.

**Co-Worker**: What? How can you tell?

**Me**: [JUnit](http://www.junit.org/) showed me the difference.

**Co-Worker**: Oh. I guess I’ll fix it now. I would have found it eventually…

For what it’s worth, we’re handling [Electronic Data Interchange](http://en.wikipedia.org/wiki/Electronic_Data_Interchange) documents — things that look like a page or so of line noise at a time. The bug was that one character wasn’t getting written to the output. My co-worker tests his code with the `toString()` and [visual grep](http://catb.org/jargon/html/V/vgrep.html) method. I’m not sure he would have ever caught the bug on his own. I keep trying to explain that unit tests are a [Good Thing](http://catb.org/jargon/html/G/Good-Thing.html)…
