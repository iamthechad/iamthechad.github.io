---
title: Known Defects and FindBugs
tags: development java oss
categories: Development
excerpt: "I ran into a bit of a quandary the other day. I had to modify a piece of legacy code in our system — one that has no tests written against it."
classes: wide
---

I ran into a bit of a quandary the other day. I had to modify a piece of legacy code in our system — one that has no tests written against it. My first task was to write as many [JUnit](http://www.junit.org/) tests as I could to document the behavior of the component to help ensure that I didn’t break existing functionality with my changes.

After a few tests, however, I began to notice something troubling. The component I was writing tests against had some bugs in it, and in order to make my unit tests pass, I had to code to those bugs. This in itself did not bother me too much, since I had no expectation that the component I was testing was error-free.

What bothered me is that unit tests are often the best documentation of how a system works, and I would be creating misleading documentation. So, I did what anyone in this situation would do: I emailed [someone a lot smarter than me](http://langrsoft.com/).

The reply email contained several suggestions, all of which I plan to implement. The first suggestion is to carefully name the test methods to make it clear which ones work because of defects. The second suggestion is to create an annotation that can be added to a method to indicate that it works because of a defect, coupled with some way to report on all uses of the annotation.

After a few hours of coding, I’m happy to announce that I’ve created a Java annotation named `KnownDefect` as well as a detector plugin for [FindBugs](http://findbugs.sourceforge.net/) to collect all of the annotation instances.

In the parlance of “[Working Effectively with Legacy Code](http://www.amazon.com/Working-Effectively-Legacy-Robert-Martin/dp/0131177052/ref=pd_bbs_sr_1?ie=UTF8&s=books&qid=1212164658&sr=1-1)”, I have been creating “[characterization tests](http://www.artima.com/weblogs/viewpost.jsp?thread=198296)”. These tests look like normal unit tests, but they document the current state of the system, warts and all. Using the `KnownDefect` annotation helps make the warts stick out better.

Here’s an example:

```java
public void testValidateNullResponse() {
   try {
     ProtocolParser.validateResponse(null);
     fail();
   } catch (InvalidRequestException expected) {
     // Caught expected exception
   }
}
```

This test documents the system as it currently exists, but the behavior is obviously not correct. The `ProtocolParser` class should be throwing an `InvalidResponseException`, not an `InvalidRequestException`. To document this defect, I change the method name and add the `KnownDefect` annotation to the test method, resulting in the following:

```java
@KnownDefect("Should throw InvalidResponseException")
public void testValidateNullResponseShowsKnownDefect() {
   try {
     ProtocolParser.validateResponse(null);
     fail();
   } catch (InvalidRequestException expected) {
     // Caught expected exception
   }
}
```

The correct behavior is now documented, and the test still passes.

Instead of writing a new tool to manage the `KnownDefect` instances, I ended up writing a new plugin for FindBugs. The FindBugs detector will collect all instances of the annotation and group them under the “Correctness” heading so they’re all in one place. Bug reports can then be created and the defects fixed or not as business needs dictate. (Sadly, the FindBugs plugin for Eclipse does not find the annotation in Groovy code. The standalone version of FindBugs works fine, so I assume there’s a resource filtering issue with the Eclipse plugin.)

Instructions on installation and use are available at the [project page](https://github.com/iamthechad/knowndefects).
