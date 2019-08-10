---
title: Frame2 Gets a Point Release
tags: development
---

_This post was originally part of a series documenting an open source web framework I worked on. The framework is well and dead, but I’m keeping these posts for posterity._

Frame2 1.3.1 is now available. The full source package is not updated yet, but it should be in the next day or so. This is a pretty minor release — only two changes, but I think they’re important changes.

1.  `Enum`s are now supported in `Event`s. This relies on the `Enum.valueOf()` behavior, so the values passed in to the event must be legal values that enum values can be built from. This means that things like case are important.
2.  Input views can now be events. I don’t know why this feature was missing. Frame2 already supported using an event for a cancel view (although the unit test didn’t actually test for that). What does it mean to be able to specify events in addition to forwards for input and cancel views? There may be cases where some data needs to be loaded or verified prior to displaying a JSP page. Sometimes this information can be lost when simply redirecting back to a JSP page. By specifying an event, the developer can be sure that the required data has been loaded before the JSP has been displayed.

The Eclipse plugins and sample web applications have been updated with the latest Frame2 jars.
