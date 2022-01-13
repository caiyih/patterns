# 访问器

## 描述

访问器封装了一种在对象的异质集合上操作的算法。
它允许在同一数据上写入多种不同的算法，而不必修改数据（或其主要行为）。

此外，访问器模式允许将对象集合的遍历与对每个对象进行的操作分开。

## 例子

```rust,ignore
// The data we will visit
mod ast {
    pub enum Stmt {
        Expr(Expr),
        Let(Name, Expr),
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

// The abstract visitor
mod visit {
    use ast::*;

    pub trait Visitor<T> {
        fn visit_name(&mut self, n: &Name) -> T;
        fn visit_stmt(&mut self, s: &Stmt) -> T;
        fn visit_expr(&mut self, e: &Expr) -> T;
    }
}

use visit::*;
use ast::*;

// An example concrete implementation - walks the AST interpreting it as code.
struct Interpreter;
impl Visitor<i64> for Interpreter {
    fn visit_name(&mut self, n: &Name) -> i64 { panic!() }
    fn visit_stmt(&mut self, s: &Stmt) -> i64 {
        match *s {
            Stmt::Expr(ref e) => self.visit_expr(e),
            Stmt::Let(..) => unimplemented!(),
        }
    }

    fn visit_expr(&mut self, e: &Expr) -> i64 {
        match *e {
            Expr::IntLit(n) => n,
            Expr::Add(ref lhs, ref rhs) => self.visit_expr(lhs) + self.visit_expr(rhs),
            Expr::Sub(ref lhs, ref rhs) => self.visit_expr(lhs) - self.visit_expr(rhs),
        }
    }
}
```

人们可以实现更多的访问器，例如类型检查器，而不需要修改AST数据。

## 动机

访问器模式在任何你想将算法应用于异质数据的地方都很有用。
如果数据是同质的，你可以使用一个类似迭代器的模式。
使用访问器对象（而不是功能化的方法）允许访问器是有状态的，从而在节点之间交流信息。

## 讨论

`visit_*`方法通常会返回void（与例子中不同）。
在这种情况下，有可能将遍历代码抽取出来，并在算法之间共享（也可以提供noop默认方法）。 
在Rust中，常见的方法是为每个数据点提供`walk_*`函数。
例如，

```rust,ignore
pub fn walk_expr(visitor: &mut Visitor, e: &Expr) {
    match *e {
        Expr::IntLit(_) => {},
        Expr::Add(ref lhs, ref rhs) => {
            visitor.visit_expr(lhs);
            visitor.visit_expr(rhs);
        }
        Expr::Sub(ref lhs, ref rhs) => {
            visitor.visit_expr(lhs);
            visitor.visit_expr(rhs);
        }
    }
}
```

在其他语言中（例如Java），数据通常有一个`accept`方法，担任同样的职责。

## 参见

访问器模式是大多数OO语言中的一种常见模式。

[访问器模式](https://en.wikipedia.org/wiki/Visitor_pattern)

[fold](../creational/fold.md)模式与visitor类似，但产生一个新版本的被访数据结构。

> Latest commit b809265 on 22 Apr 2021