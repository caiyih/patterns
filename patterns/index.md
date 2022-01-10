# 设计模式

[设计模式](https://en.wikipedia.org/wiki/Software_design_pattern) 
是“在软件设计的特定背景下，对一个经常发生的问题的一般可重复使用的解决方案”。
设计模式是描述一种编程语言文化的好方法。
设计模式具有很强的语言特异性——在一种语言中属于模式的东西，在另一种语言中可能由于语言特性而不需要，或者由于缺少特性而无法表达。

如果过度使用，设计模式会给程序增加不必要的复杂性。
然而，它们是分享关于一种编程语言的中高级知识的好方法。

## Rust中的设计模式

Rust有许多特性。这些特性通过消除整类问题给我们带来了巨大的好处。其中有些也是Rust的**独特**模式。

## YAGNI

如果你不熟悉，YAGNI是一个缩写，代表`You Aren't Going to Need It`。这是一个重要的软件设计原则，在你写代码时要应用。

> 我曾经写过的最好的代码是我从未写过的代码。

如果我们将YAGNI应用于设计模式，我们会发现Rust的特性允许我们抛开许多模式。
例如，在Rust中没有必要使用[策略模式](https://en.wikipedia.org/wiki/Strategy_pattern)因为我们有[traits](https://doc.rust-lang.org/book/traits.html)。

TODO：加入一些代码来说明这些traits。

> Latest commit 9834f57 on 25 Aug 2021