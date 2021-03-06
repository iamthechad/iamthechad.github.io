---
title: Setting UITableView with grouped style background in iOS
excerpt: Set a grouped table's backgroundView to nil to properly show a custom background when in a popover.
tags: development ios
categories: Development
classes: wide
redirect_from: "/setting-uitableview-with-grouped-style-background-in-ios-8ba78156e678"
---

Setting the background of a `UITableView` is a simple operation in most cases. In my latest app, we set the background to a nice texture with a line like this:

```objc
self.tableView.backgroundColor = [UIColor colorWithPatternImage: [UIImage imageNamed: @"myFancyBackground.png"]];
```

For some reason, when a `UITableView` with the grouped style is presented in a popover, this code does not work. It turns out that grouped table views have a custom `backgroundView` property. I don’t know why this property only takes precedence when the table is in a popover, but here’s how to fix it to display your custom background:

```objc
self.tableView.backgroundView = nil;
self.tableView.backgroundColor = [UIColor colorWithPatternImage: [UIImage imageNamed: @"myFancyBackground.png"]];
```

Setting the `backgroundView` to `nil` doesn’t seem to have any ill side effects that I’ve noticed. Hopefully someone can correct me if I’m wrong.
