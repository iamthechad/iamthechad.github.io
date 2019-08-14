---
title: A Consistent API is a Very Good Thing
tags: development java
categories: Frame2
excerpt: Consistency is an important aspect of API design.
classes: wide
redirect_from: "/a-consistent-api-is-a-very-good-thing-f24f11e4b5b"
---

_This post was originally part of a series documenting an open source web framework I worked on. The framework is well and dead, but I’m keeping these posts for posterity._

I’ve been mucking around in the codebase, cleaning up pieces of the API that annoy me. Some bright person thought that it was a good idea to have a method that returns an `Iterator` to access a collection. I’ve been cleaning up all of these methods, changing them so that they return an actual collection - occasionally made unmodifiable through the `Collections` class.

Naturally, this has led to unit test failures. Almost all of the failures have been easy to fix, but once in a while one pops up that makes me scratch my head. Here’s an example:

```java
public void testGetIfEmpty() {
   assertTrue(this.errors.isEmpty());
   assertNull(this.errors.get(FOO));
   assertEquals(0, this.errors.get().length);
   assertEquals(0, this.errors.get(FOO).length);
}
```

In this instance, `errors` is a class that aggregates individual `Error` objects. The `get(String)` method returns an array of `Error` objects that have the specified key, while `get()` retrieves all `Error`s. Pretty simple, right? After my changes, the test fails on the `assertNull()` statement, which isn’t surprising.

The failing statement read `assertNull(this.errors.iterator(FOO))` before I removed the `iterator()` method, which tells us a bit about how the API was originally coded. The `iterator(String)` returned `null` if there were no matching entries. However, as the unit test clearly shows, `get(String)` returns an empty array when it doesn’t find any matching `Error`s. Quite the difference, and potentially confusing - the developer has to remember which method may return `null`.

The API has now been changed to return an empty collection instead of `null` when no matching errors are found. Now, a user of the API simply has to call the method and loop over the result (zero times if it’s empty) without having to make `null` checks.

Whoever wrote this unit test obviously knew that one method returned `null` while the other returned an empty array for the same input, since the test passed. I only wish that person would have looked at those lines and noticed the inconsistency, instead of leaving it for me to find.
