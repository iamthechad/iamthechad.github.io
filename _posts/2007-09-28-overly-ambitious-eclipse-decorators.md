---
title: Overly Ambitious Eclipse Decorators
tags: development java eclipse
categories: Frame2
excerpt: "I don’t think It’s just me, but maybe I am the only person who gets easily frustrated working on Eclipse plugins."
classes: wide
redirect_from: "/overly-ambitious-eclipse-decorators-2ad523b6710"
---

_This post was originally part of a series documenting an open source web framework I worked on. The framework is well and dead, but I’m keeping these posts for posterity._

I don’t think It’s just me, but maybe I am the only person who gets easily frustrated working on Eclipse plugins. I almost feel like I’ve missed a secret meeting sometimes that ties together all of the loose ends of plugin development. One particular problem I ran into lately was with adding a project decorator. I thought it would be nice if Frame2 projects had some kind of icon that identified them. I added the extension to the plugin and set the `enablement` for `IJavaProject`. I thought this was appropriate, since Frame2 projects are Java projects. This setting resulted in every single file in the project being decorated. I thought this was a bit strange, so I added a lightweight label decorator class. The `ILightweightLabelDecorator` interface defines a method that implementors use to apply the decorator to objects. Sure enough, this method was called once for every file in the project. However, I could not filter these inputs since every time the method was called, the input parameter was the same!

Changing the enablement class to `IProject` yielded the desired behavior - the decorator method only gets called once for each project in the workspace. A little filtering to check if a project is a Java project and has the Frame2 nature, and presto! - project decorators.

Now, why did I see the behavior I did? Chances are it’s a bug, but it seems equally likely that it’s supposed to behave that way and I haven’t found the decoder ring that tells me that.
