---
title: Eclipse Plugin Nearly Ready
tags: development java eclipse
categories: Frame2
excerpt: The Eclipse plugin for Frame2 is almost done.
classes: wide
---

_This post was originally part of a series documenting an open source web framework I worked on. The framework is well and dead, but I’m keeping these posts for posterity._

As the title says, the Eclipse plugin for Frame2 is almost done. Of course, “done” is a relative term. In this case, it means that the plugin has been updated to work with the latest version of the framework, including the updated DTD. Unfortunately, it also means that it still has the same limitations that previous versions have had. Specifically, the plugin still does not support creating a Frame2 project with web services support. This can be added by hand and the Frame2 plugin won’t stomp on it, but neither are there any nice wizards for the web services tasks. This limitation is the next item on my list — I’m pretty sure that I can create an optional plugin that can be downloaded that will add the web services functionality for anyone that wants it. This will also help keep the download size of the “core” plugin as small as possible.

One other feature that the plugin is missing is a contextual editor for the Frame2 configuration file. It would nice to be able to get contextual help and hints when editing the config by hand, but I don’t think I have the time in the foreseeable future to research how this works in Eclipse.
