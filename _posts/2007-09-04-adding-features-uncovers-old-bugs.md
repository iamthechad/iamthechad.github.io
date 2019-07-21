---
title: Adding Features Uncovers Old Bugs
layout: post
date: '2007-09-04T00:00:00.000Z'
tags: development
---

_This post was originally part of a series documenting an open source web framework I worked on. The framework is well and dead, but I’m keeping these posts for posterity._

This seems to happen to me no matter what project I’m on. I try to implement a simple (or not so simple) feature and end up exposing at least one bug that’s been hiding around for quite some time.

As an illustration, I was adding a feature to Frame2 today to allow some parameters to be passed between events within a mapping — without making developers use the `HttpSession`. After a couple of false starts, I had a pretty good idea of how to make the changes and starting ripping out code. I came up with several unit tests, and modifying code to make the tests pass. Pretty standard [Test Driven Development](http://en.wikipedia.org/wiki/Test-driven_development)…until I wrote the test that made sure event chain validation was working.

Before I explain the bug, I’ll give a high level description of how Frame2 works when submitting form data:

  1.  Form is filled out by user and submitted.
  1.  Frame2 looks up the appropriate event from the form action.
  1.  An instance of the appropriate event is created and populated by the introspector.
  1.  Based on the mapping for the event, Frame2 will then validate the event. This may involve the [Commons Validator](http://commons.apache.org/validator/) if it is enabled and/or overriding the `Event.validate()` method.
  1.  If validation fails, the mapping must specify a location for the framework to forward back to. This is usually the same page where the data was entered.
  1.  If validation passes, the framework then iterates through the list of event handlers, calling the `handle()` method. If `handle()` returns null, the framework continues to the next handler. A non-null result usually indicates an error, and the framework will forward to the location named by the result.
  1.  Once all of the handlers have been invoked, the framework will default to the view specified in the mapping. Usually this is just a JSP, but Frame2 supports forwarding to events — called “chaining”.
  1.  For the chained event, Frame2 is supposed to start back at step 2 and continue processing until it gets to the end of the chain.

Notice how I say “supposed to” in step 8? The framework was actually only going back to step 6, bypassing validation for all events except the first in the chain. Subtle, yes, and unlikely to be noticed by most users — but it’s still a bug. The bug has now been fixed so that the framework goes back to step 2 like it’s supposed to.

In case you’re wondering what use event chaining is, here’s an example:

  1.  User enters a blog entry and submits the form.
  1.  Frame2 validates the form data and begins invoking handlers.
  1.  The blog entry handler persists the data, perhaps to a database.
  1.  The default view for the event mapping tells the framework to forward to the “View All Blog Entries” event.
  1.  The framework creates the appropriate object and begins invoking the handlers in the “View All” mapping.
  1.  The “View All” handler populates all of the blog entries into the “View All” event.
  1.  The default view for the “View All” mapping is a JSP page, which renders the blog entries using [JSTL](http://java.sun.com/products/jsp/jstl/).

Chaining allows the “View All” event to remain its own entity that can be invoked separately, while allowing the same functionality to be used in other places where appropriate.

By way of comparison, the feature I added allows the above sequence to be reworked like this:

  1.  User enters a blog entry and submits the form.
  1.  Frame2 validates the form data and begins invoking handlers.
  1.  The blog entry handler persists the data, perhaps to a database.
  1.  The blog entry handler sets an attribute with the new entry id into the context.
  1.  The default view for the event mapping tells the framework to forward to the “View Specific Blog Entry” event.
  1.  The framework creates the appropriate object and uses the context value(s) with the introspector to set some values in the event.
  1.  The framework then begins invoking the handlers in the “View Specific” mapping.
  1.  The “View Specific” handler looks up the entry with the specified id and sets its data into the event.
  1.  The default view for the “View Specific” mapping is a JSP page, which renders the blog entry using [JSTL](http://java.sun.com/products/jsp/jstl/).
