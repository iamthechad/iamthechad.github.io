---
title: Approaching the Finish Line
tags: development
---

_This post was originally part of a series documenting an open source web framework I worked on. The framework is well and dead, but I’m keeping these posts for posterity._

Frame2 is almost ready to be released as a new version. I may still make some changes, but it’s definitely in Release Candidate stage now. There are only a few things that I may add before testing the heck out of it:

*   Upgrade unit tests to [JUnit 4](http://www.junit.org/index.htm). There are a lot of tests, especially the ones that mock a servlet container, that do a lot of repetitive work in `setUp()` . JUnit 4 has finally brought the ability to annotate methods to be run before and after all tests in the class with `@BeforeClass` and `@AfterClass`, respectively. Individual test setup activities can still be performed by annotating a method with `@Before`.
*   Fix outstanding bugs. There’s only one right now, and it’s SOAP specific, so I may or may not fix it. It may take me longer to figure out how to reproduce the problem with unit tests than it’s worth.
*   Add outstanding feature requests. Again, there’s only one, but it’s (in my opinion) a biggie — the ability to add parameters to the response when handling an event.
*   Doc updates. The web services doc needs to be written, and I’m sure all of the other doc needs a good examination.

I’ll be testing the latest build with a web application that I’ve been working on for years that I’ve come to think of as an unofficial reference implementation of Frame2. I haven’t decided yet if the web app should become part of the project on Github or not. We’ll see.

Once the testing is done and I’ve released a new version of Frame2, I plan on working on the [Eclipse plugin](https://github.com/iamthechad/frame2/tree/master/Frame2Eclipse). The poor plugin has pretty much been ignored for several years now, and it needs some attention.
