---
title: FindBugs Is Even Cooler Than I Thought
tags: development java findbugs
categories: Frame2
excerpt: Fine tuning FindBugs to achieve better results.
classes: wide
redirect_from: "/findbugs-is-even-cooler-than-i-thought-a94249fb471c"
---

_This post was originally part of a series documenting an open source web framework I worked on. The framework is well and dead, but I’m keeping these posts for posterity._

I’ve mentioned before that I’m using [FindBugs](http://findbugs.sourceforge.net/index.html) to help make sure that the Frame2 code is up to snuff. The Eclipse compiler can catch a lot of things, but [FindBugs](http://findbugs.sourceforge.net/index.html) has some neat tricks up its sleeve to find things that Eclipse can’t. [FindBugs](http://findbugs.sourceforge.net/index.html) uses static analysis of the code to search for patterns that indicate possible issues. Being the Type A person that I am, I always turn on all of the possible matches and whittle the list down. Quite often, however, [FindBugs](http://findbugs.sourceforge.net/index.html) will find a false positive or two. Up until now, I’ve either tried to rework the code to avoid the warning or disabled the category altogether. I’ve never liked the approach of turning off a whole category just to avoid seeing a warning or two, so I spent the morning muttering to myself how useful it would be if [FindBugs](http://findbugs.sourceforge.net/index.html) allowed me to ignore certain warnings like I can do with `@SuppressWarnings`.

Then I went and [RTFM](http://findbugs.sourceforge.net/manual/index.html).

[FindBugs](http://findbugs.sourceforge.net/index.html) has a [filtering system](http://findbugs.sourceforge.net/manual/filter.html) that makes `@SuppressWarnings` look amateur. Here’s an example: in the `SoapRequestProcessor`, [FindBugs](http://findbugs.sourceforge.net/index.html) marked a warning that an `Exception` was being caught when no `Exception` was being thrown. After looking at the code and verifying that the try block in question throws several different exceptions, I created a filter entry. The entry looks like this:

```xml    
<Match>
  <Class name="org.megatome.frame2.front.SoapRequestProcessor"/>
  <Method name="getEvents"/>
  <Bug pattern="REC_CATCH_EXCEPTION"/>
</Match>
```

This tells [FindBugs](http://findbugs.sourceforge.net/index.html) to match a specific bug type in a specified method in a desired class. This entry can be used in both an inclusion or exclusion filter — I use it to exclude that warning from the results.

Here’s a more complex example:

```xml
<Match>
  <Class name="~.*introspector\.Bean\d+" />
  <Bug pattern="EI_EXPOSE_REP,EI_EXPOSE_REP2"/>
</Match>
```

There are some test classes that don’t exactly follow good rules of programming when it comes to dealing with mutability. Since they are test classes, I don’t really care to fix them. Instead I set up the filter to match all classes in an introspector package named Bean1, Bean2, etc. Simple as can be!

If you aren’t using [FindBugs](http://findbugs.sourceforge.net/index.html), you should be. [Go check it out](http://findbugs.sourceforge.net/index.html).
