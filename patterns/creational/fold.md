# Fold

## 描述

在数据集的每一项上运行一个算法，以创建一个新的项，从而创建一个全新的集合。

我不清楚这里的词源。
Rust编译器中使用了`fold`和`folder`这两个术语，尽管在我看来，它更像`map`，而不是通常意义上的`fold`。 
更多细节见下面的讨论。

## 例子

```rust,ignore
// The data we will fold, a simple AST.
mod ast {
    pub enum Stmt {
        Expr(Box<Expr>),
        Let(Box<Name>, Box<Expr>),
    }

    pub struct Name {
        value: String,
    }

    pub enum Expr {
        IntLit(i64),
        Add(Box<Expr>, Box<Expr>),
        Sub(Box<Expr>, Box<Expr>),
    }
}

// The abstract folder
mod fold {
    use ast::*;

    pub trait Folder {
        // A leaf node just returns the node itself. In some cases, we can do this
        // to inner nodes too.
        fn fold_name(&mut self, n: Box<Name>) -> Box<Name> { n }
        // Create a new inner node by folding its children.
        fn fold_stmt(&mut self, s: Box<Stmt>) -> Box<Stmt> {
            match *s {
                Stmt::Expr(e) => Box::new(Stmt::Expr(self.fold_expr(e))),
                Stmt::Let(n, e) => Box::new(Stmt::Let(self.fold_name(n), self.fold_expr(e))),
            }
        }
        fn fold_expr(&mut self, e: Box<Expr>) -> Box<Expr> { ... }
    }
}

use fold::*;
use ast::*;

// An example concrete implementation - renames every name to 'foo'.
struct Renamer;
impl Folder for Renamer {
    fn fold_name(&mut self, n: Box<Name>) -> Box<Name> {
        Box::new(Name { value: "foo".to_owned() })
    }
    // Use the default methods for the other nodes.
}
```

在AST上运行`Renamer`的结果是一个与旧AST相同的新AST，但每个名字都改为`foo`。
现实生活中的`folder`可能会在结构本身的节点之间保留一些状态。

也可以定义一个`folder`，将一个数据结构映射到一个不同的（但通常是类似的）数据结构。
例如，我们可以将AST`fold`成HIR树（HIR代表high-level intermediate representation，高级中间表示法）。

## 动机

通过对结构中的每个节点进行一些操作来映射一个数据结构是很常见的。 
对于简单数据结构的简单操作，可以使用`Iterator::map`来完成。
对于更复杂的操作，也许前面的节点会影响后面节点的操作，或者在数据结构上的迭代不是简单的，使用`fold`模式更合适。

与访问者模式一样，`fold`模式允许我们将数据结构的遍历与对每个节点进行的操作分开。

## 讨论

以这种方式映射数据结构在函数式语言中是很常见的。
在OO语言中，更常见的是在原地改变数据结构。
“函数式”方法在Rust中很常见，主要是由于对不可变性的偏好。
使用新的数据结构，而不是改变旧的数据结构，在大多数情况下使代码推理更容易。

通过改变`fold_*`方法接受节点的方式，可以对效率和可重用性之间的权衡进行调整。

在上面的例子中，我们对`Box`指针进行操作。由于这些指针排他地拥有其数据，数据结构的原始副本不能被重新使用。
另一方面，如果一个节点没有改变，重新使用它是非常有效的。

如果我们对借来的引用进行操作，原来的数据结构可以被重用；但是，一个节点即使没有变化，也必须被克隆，这可能很昂贵。

使用引用计数指针可以获得两全其美的效果——我们可以重用原来的数据结构，而且我们不需要克隆未改变的节点。
然而，它们在使用上不太符合人体工程学，而且意味着数据结构不能被改变。

## 参见

迭代器有一个`fold`方法，但是这个方法将一个数据结构`fold`成一个值，而不是`fold`成一个新的数据结构。迭代器的`map`更像是这种`fold`模式。

在其他语言中，`fold`通常是在Rust迭代器的意义上使用，而不是这种模式。
一些函数式语言拥有强大的结构，可以对数据结构进行灵活的映射。

[访问者](../behavioural/visitor.md)模式与`fold`密切相关。
它们的共同概念是在一个数据结构上遍历，对每个节点进行操作。
然而，访问者模式并不创建一个新的数据结构，也不消耗旧的数据结构。

> Latest commit 9834f57 on 25 Aug 2021