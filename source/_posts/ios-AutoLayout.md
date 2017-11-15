---
title: Auto Layout 知识点梳理
date: 2014-10-15 22:29:37
tags: iOS
---

视图需要有确定的位置与大小才能正确显示在屏幕上。Auto Layout使用对齐矩阵来确定视图的位置与大小，也就是所谓的约束。我们创建的每一条规则都规定了界面的一部分与另一部分的关系，某一部分可以由另一部分计算得出结果。
y = ax + b;
是一种线性关系。

创建约束的常见的方式：

* NSLayoutConstraint
* VFS
* Xib

第一种，可以在(Interface Builder)IB中布局约束，并且根据需求自定义它们。

第二种，可以使用代码创建单个约束。NSLayoutConstraint类提供`constraintWithItem:attribute:relatedBy:toItem:attribute:multiplier:contant:`方法，可以让你每次创建一个约束，它将某项的属性关联到另一项。

第三种，使用可视化格式语言来表示各项是如何沿着垂直和水平坐标轴布局的。


所有约束都是NSLayoutConstraint类的成员，无论你是以何种方式创建它们的。每个约束都在一个Objective—C对象中存储`y = ax + b`规则，并且通过Auto Layout引擎来表达该规则。可视化约束 是另一种实现相同效果的工具。

