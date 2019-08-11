---
title: Frame2 and MacOS X
tags: development java macos
categories: Frame2
excerpt: Currently, Frame2 does not work on Macs.
classes: wide
---

_This post was originally part of a series documenting an open source web framework I worked on. The framework is well and dead, but I’m keeping these posts for posterity._

Currently, Frame2 does not work on Macs. This, of course, is due to the fact that there is no (easily) available Java 6 implementation for MacOS. I’ve been trying to decide if this is an issue or not.

Reasons why it might be an issue:

*   I use a Mac for most of development, and I hate having to drop into an emulator to work on Frame2
*   I really would like Frame2 to run on the Mac, if just for completeness.

Reasons why it probably doesn’t matter:

*   Who really runs Java webapps on Macs? Tomcat on Windows or Linux is OK, but it just feels dirty to me on the Mac.

Hopefully, there will be a version of Java 6 available for the Mac soon. At that point, I’ll probably make the necessary updates to make everything work. Until then, if you’re writing a web app and using a Mac — go with Rails.
