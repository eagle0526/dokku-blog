---
title: LinkedList介紹
sidebar_label: "LinkedList介紹"
description: LeetCode
last_update:
  date: 2023-11-20
keywords:
  - LeetCode  
sidebar_position: 1
---





Linked list（鏈表）是一種常見的數據結構，用於儲存一系列元素。Linked list 中的元素被稱為節點（node），每個節點包含數據以及一個指向下一個節點的引用（指針或鏈接）。

Linked list 和陣列（Array）相比，有一些不同之處：

1. 內存佔用： Linked list 的節點在內存中不需要連續的空間，而陣列需要。這意味著在Linked list 中，節點可以在內存中散落，每個節點只需要存儲數據和指向下一個節點的引用。這使得 Linked list 在插入和刪除元素時更靈活。
2. 存取效率： 在陣列中，可以通過索引直接存取元素，而在Linked list 中，如果要訪問某個節點，需要從頭節點開始，依次跟踪指針直到找到目標節點。這導致在存取元素方面，Linked list 的效率相對較低。

根據節點之間的關係，Linked list 可以分為單向鏈表、雙向鏈表和循環鏈表等不同類型。

一個簡單的單向鏈表範例：

```md
Node 1       Node 2       Node 3       Node 4
+------+    +------+    +------+    +------+
| Data |    | Data |    | Data |    | Data |
+------+    +------+    +------+    +------+
| Next |--> | Next |--> | Next |--> | None |
+------+    +------+    +------+    +------+

```