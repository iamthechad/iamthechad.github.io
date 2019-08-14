---
title: iOS6 table views and accessory segues
excerpt: Getting the correct indexPath for a table cell's accessory action segue is different than for a cell selection segue.
tags: ios development
categories: Development
classes: wide
redirect_from: "/ios6-table-views-and-accessory-segues-23b041b1b879"
---

If you’re performing segues on a table row selection, you’re probably used to doing something like this in the `prepareForSegue:` method:

```objc
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
  if ([[segue identifier] isEqualToString:@"mySegueIdentifier"]) {
    NSIndexPath *indexPath = [self.tableView indexPathForSelectedRow];
    ... do some work ...
  }
}
```

In iOS6, segues can now be connected from a table cell’s accessory action. When making this connection, however, the `indexPathForSelectedRow` returns `nil`. The `prepareForSegue:` method has a `sender` parameter. In the case of an accessory action, this parameter is a `UITableViewCell`. Knowing this, we can get the correct `indexPath` like this:

```objc
- (void)prepareForSegue:(UIStoryboardSegue *)segue sender:(id)sender {
  if ([[segue identifier] isEqualToString:@"mySegueIdentifier"]) {
    NSIndexPath *indexPath = [self.tableView indexPathForCell:sender];
    ... do some work...
  }
}
```
