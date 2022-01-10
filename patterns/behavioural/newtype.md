# 新类型

如果在某些情况下，我们希望一个类型的行为类似于另一个类型，或者在编译时强制执行一些行为，而仅仅使用类型别名是不够的，怎么办？

例如，如果我们出于安全考虑（如密码），想为`String`创建一个自定义的`Display`实现。

对于这种情况，我们可以使用`Newtype`模式来提供**类型安全**和**封装**。

## 描述

使用单个字段的元组结构体为一个类型做不透明包装。
这将创建一个新的类型，而不是一个类型的别名（`type`项）。

## 例子

```rust,ignore
// Some type, not necessarily in the same module or even crate.
struct Foo {
    //..
}

impl Foo {
    // These functions are not present on Bar.
    //..
}

// The newtype.
pub struct Bar(Foo);

impl Bar {
    // Constructor.
    pub fn new(
        //..
    ) -> Self {

        //..

    }

    //..
}

fn main() {
    let b = Bar::new(...);

    // Foo and Bar are type incompatible, the following do not type check.
    // let f: Foo = b;
    // let b: Bar = Foo { ... };
}
```

## 动机

新类型的主要动机是抽象化。它允许你在类型之间共享实现细节，同时精确控制接口。
通过使用新类型而不是将实现类型作为API的一部分公开，它允许你向后兼容地改变实现。

新类型可以用来区分单位，例如，包装`f64`以获得可区分的`Miles`和`Kms`。

## 优势

被包装的类型和包装后的类型不是类型兼容的（相对于使用`type`），所以新类型的用户永远不会“混淆“包装前后的类型。

新类型是一个零成本的抽象——没有运行时的开销。

隐私系统确保用户无法访问被包装的类型（如果字段是私有的，默认情况下是私有的）。

## 劣势

新类型的缺点（尤其是与类型别名相比）是没有特殊的语言支持。这意味着可能会有*许多*模板代码。
你需要为你想在包装类型上公开的每个方法提供一个”通过“方法，并为你想在包装类型上实现的每个trait提供一个实现。

## 讨论

新类型在Rust代码中非常常见。抽象或代表单位是最常见的用途，但它们也可以用于其他原因：

- 限制功能（减少暴露的函数或实现的trait），
- 使一个具有复制语义的类型具有移动语义，
- 通过提供一个更具体的类型，从而隐藏内部类型来实现抽象，
  例如，

```rust,ignore
pub struct Foo(Bar<T1, T2>);
```

这里，`Bar`可能是一些公共的、通用的类型，`T1`和`T2`是一些内部类型。
我们模块的用户不应该知道我们通过使用`Bar`来实现`Foo`，但我们在这里真正隐藏的是`T1`和`T2`类型，以及它们如何与`Bar`一起使用。

## 参见

- [“圣经“中的高级类型](https://doc.rust-lang.org/book/ch19-04-advanced-types.html?highlight=newtype#using-the-newtype-pattern-for-type-safety-and-abstraction)
- [Haskell中的新类型](https://wiki.haskell.org/Newtype)
- [类型别名](https://doc.rust-lang.org/stable/book/ch19-04-advanced-types.html#creating-type-synonyms-with-type-aliases)
- [derive_more](https://crates.io/crates/derive_more)是一个用于在新类型上派生许多内置trait的crate。
- [Rust中的新类型模式](https://www.worthe-it.co.za/blog/2020-10-31-newtype-pattern-in-rust.html)

> Latest commit 11a0a13 Dec 14 2021