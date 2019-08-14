---
title: Supporting the iPhone 5 Screen
tags: ios development
categories: Development
excerpt: Adding iPhone 5 support to your app.
classes: wide
redirect_from: "/supporting-the-iphone-5-screen-42bb001eca95"
---

I had to dig a bit to find this information when adding iPhone 5 support to my app. It’s pretty easy: **you need to supply a 640px by 1136px launch image**. That’s it. Of course, there may be layout issues that need tending to afterwards, but that’s a different story.

I suspect that Apple forced the inclusion of the launch image to make all devs use them. I know quite a few apps that don’t have launch images, even though [Apple’s HIG says they’re required](http://developer.apple.com/library/ios/#documentation/userexperience/conceptual/mobilehig/IconsImages/IconsImages.html).
