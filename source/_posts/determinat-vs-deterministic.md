---
title: deterministic 和 determinat 有什么区别？
date: 2017-05-08 19:06:05
updated: 2017-05-08 21:04:05
tags:
categories:
---

deterministic 和 determinat 翻译过来都是“确定性的”，但这并不代表它们可以互相混用，在并行计算的语境下，它们有着一些根本的差别。

<!--more-->

从语源的角度来说，deterministic 的名词形式 [determinism] 指的是哲学中的决定论，决定论声称万物的发展皆是（由上帝 / 物理规律）事先决定好的，没有改变的余地。而 determinat 的名词形式 [determinacy] 指的是集合论中一个关注游戏必胜策略的子领域，或者直接指代“拥有必胜策略”这一属性。可以从语源看出，determinism 强调“没有什么可以改变”，而 determinacy 则更专注于“结果无法改变”（所谓必胜）这一点。

[determinism]: https://www.wikiwand.com/en/Determinism
[determinacy]: https://www.wikiwand.com/en/Determinacy

因此，当计算机科学引入这些术语时，自然借鉴了其原先的语义。比如，“确定性算法”这个术语的英文为“deterministic algorithm”，其解释如下：

{% blockquote Wikipedia https://en.wikipedia.org/wiki/Deterministic_algorithm %}
In computer science, a deterministic algorithm is an algorithm which, given a particular input, will always produce the same output, with the underlying machine always passing through the same sequence of states.
{% endblockquote %}

该释义强调了确定性算法在给定输入的情况下的两个必要属性：

1. 永远产生相同的输出
2. 永远产生相同的状态变化过程

可以看出，deterministic 的“确定性”既包含了结果的确定性，也包含了过程的确定性。

相比之下，determinat 则没有过程的确定性这一层含义。看看这段对 Id 语言（一门函数式并行编程语言）的描述：

{% blockquote Rishiyur S. Nikhil, 1991 http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.18.4920 ID Language Reference Manual %}
Programs in the functional subset of Id are guaranteed to be determinate, by which we mean that that it is impossible for the programmer to write a program that, despite different schedules on different runs, produces two different outcomes.
{% endblockquote %}

这段描述特别强调了 determinate 是无关调度与运行过程的，只求结果的确定性。而正是对执行过程的限制与否，造成了 deterministic 和 determinat 的在并行计算语境下的根本差异。

一段程序（或者一个抽象的执行过程），如果使用 deterministic 进行描述，那么构成它的指令的执行顺序和执行结果都是确定的，这意味着这段程序是很难并行化的。要在保证指令的执行顺序的前提下并行运行多条指令在逻辑上就是不可能的（但在实现上是可能的，比如通过将一条指令进行拆分，从而能够并行运行多条指令的不同部分）。而 determinate 则不同，它只要求执行结果是确定的，这意味着程序内部可以通过并行运行多条互相无依赖的指令进行并行化。

举个简单的例子，计算 `(1 + 2) * (3 + 4)`，如果程序是 deterministic 的，那么计算会包含三步：

1. 计算 `1 + 2`
2. 计算 `3 + 4`
3. 计算 `3 * 7`

而一个 determinate 的程序则能够通过并行化两步完成计算：

1. 计算 `1 + 2`，同时计算 `3 + 4`
2. 计算 `3 * 7`

容易推论，deterministic 的程序的运行时间取决于程序中的指令总数，而 determinate 的程序的运行时间取决于最长依赖路径的长度（假定资源无限）。这保证了 determinate 的程序能够充分利用系统的并行性进行加速。

总结来说，deterministic 和 determinate 都强调结果的确定性，但 deterministic 在此之上还要求执行过程的确定性。在表达“确定性”的场合，需要根据语义进行选择，而不是随意地混用它们[^1]。

[^1]: 最近有用 deterministic 替代 determinate 的趋势，在这种情况下，文献的作者通常对 deterministic 进行了分类或是对其使用语境进行了详细的描述
