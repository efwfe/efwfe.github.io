---
title: Ios UI components
date: 2024-07-03 10:24:20
tags:
---

## TableView

> the table view needs to know how many table view clls to display and what to display in each cell. Normally, the view controllner is responsible for providing this information.

> `UITableViewDataSource`, a protocal for this purpose. All you need to do is set the table view's `dataSource` property to class and implement the required methods of ths protocol.

> The table view also needs to know what to do if the user  taps on a table view cell. Again the view controller for the table view is responsible, and Apple has created the `UITableViewDelegate` protocol for this purpose.



