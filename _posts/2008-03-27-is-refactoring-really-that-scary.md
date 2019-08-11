---
title: Is Refactoring Really That Scary?
excerpt: I got admonished at my new job for coding practices that have become second nature to me. Is aggressive refactoring really that scary?
tags: development java design
categories: Development
classes: wide
---

I’ve been coding with an agile mindset for quite some time now. Some practices have become second nature to me over the years. One of these practices is aggressive code refactoring, coupled with test driven development (TDD).

I’ve been adding new functionality to our system at my new job, and as usual I’ve been following my standard techniques. My part of the system involves handling certain requests and responses. I wrote unit and functional tests for the request half of the equation, then began to tackle the response side. As I added (and tested) functionality, I began to notice a large amount of similarity in the code. This in turn led me to create an abstract base class for both request and response, pulling up all of the shared functionality. This resulted in the actual request and response classes being quite small, with just a few method implementations from the base class.

So far, this activity should seem pretty normal to most developers. As I said, it’s pretty much second nature for me, and I don’t even remember consciously making the decision to make the abstract class. Imagine my surprise when my team lead admonished me for creating and abstract class, and warned me not to do it again without explicit approval.

Of course I had to ask why we would not refactor code in this manner when the opportunity presented itself. My lead’s answer involved several parts, none of which have convinced me that refactoring is bad:

1.  **The abstract class is not consistent with the rest of the system.** This, to me, seems to be a fault of the system. I’m not knocking consistency, but at some point you have to realize that you may be building something that’s consistently bloated/overcomplicated/etc.
2.  **The abstract class changes the design.** Huh? Sure, if I go generate a UML diagram of the system, there’s going to be this abstract class and inheritance there. This is the only place the design is different. Requests and responses are being correctly handled just like the legacy ones in the system.
3.  **Too much refactoring is bad, and each case should be thoroughly examined before code is changed.** This is how religious wars start. My lead claims that developers start out on the “no refactoring” end of the spectrum, simply because they don’t know any better. Once they learn about refactoring, they then swing to the “refactor everything” end of the spectrum where anything and everything is a potential target. Seasoned, knowledgeable developers fall somewhere in the middle. I got the impression from this speech that my lead considers me to be on the radical end of the spectrum.

Why are people like my lead so scared of refactoring? The arguments given really seem more like rationalizations than actual reasons — they don’t really stand up to scrutiny. I have a hard time believing, for example, that my lead truly thinks that the system is better because each request and response is a totally separate class, albeit with most of the code copied and pasted from one to the other. If anything, he should know that this approach leads to more problems, no matter how pretty it keeps the class diagrams.
