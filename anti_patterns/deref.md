# `Deref` 多态性

## 描述

滥用`Deref` trait来模拟结构体间的继承，从而重用方法。

## 例子

有时我们想模仿以下来自OO语言（如Java）的常见模式：

```java
class Foo {
    void m() { ... }
}

class Bar extends Foo {}

public static void main(String[] args) {
    Bar b = new Bar();
    b.m();
}
```

我们可以使用deref多态性的反面模式来做到这一点：

```rust
use std::ops::Deref;

struct Foo {}

impl Foo {
    fn m(&self) {
        //..
    }
}

struct Bar {
    f: Foo,
}

impl Deref for Bar {
    type Target = Foo;
    fn deref(&self) -> &Foo {
        &self.f
    }
}

fn main() {
    let b = Bar { f: Foo {} };
    b.m();
}
```

Rust中没有结构体的继承。相反，我们使用组合，并在`Bar`中包含一个`Foo`的实例（因为字段是一个值，它被内联存储，所以如果有字段，它们在内存中的布局与Java版本相同（可能，如果你想确定，你应该使用`#[repr(C)]`））。

为了使方法调用生效，我们为`Bar`实现了`Deref`，以`Foo`为目标（返回嵌入的`Foo`字段）。这意味着当我们解除对`Bar`的引用时（例如，使用`*`），我们将得到一个`Foo`。
这很奇怪。解引用通常从对`T`的引用中得到一个`T`，这里我们有两个不相关的类型。
然而，由于点运算符做了隐式解引用，这意味着方法调用将搜索`Foo`和`Bar`的方法。

## 优势

你可以节省一点模板代码，例如：

```rust,ignore
impl Bar {
    fn m(&self) {
        self.f.m()
    }
}
```

## 劣势

最重要的是这是一个令人惊讶的惯常做法--未来的程序员在代码中读到这句话时，不会想到会发生这种情况。
这既因为我们在滥用`Deref` trait，而不是按照预期（文档等）使用它。
也因为这里的机制是完全隐含的。

这种模式没有像Java或C++中的继承那样在`Foo`和`Bar`之间引入子类型。此外，由`Foo`实现的特性不会自动为`Bar`实现，所以这种模式与边界检查以及泛型编程的互动性很差。

使用这种模式，在`self`的语义上与大多数OO语言有细微的不同。
通常情况下，它仍然是对子类的引用，在这种模式下，它将是定义方法的"类"。

最后，这种模式只支持单继承，没有接口的概念，没有基于类的隐私，也没有其他与继承有关的特性。
所以，它给人的体验会让习惯了Java继承等的程序员感到微妙的惊讶。

## 讨论

我们没有一个好的替代方案。根据具体的情况，使用traits重新实现或者手动写出派发给`Foo`的facade方法可能更好。
我们确实打算在Rust中加入与此类似的继承机制，但要达到稳定的Rust，可能还需要一些时间。详情可见：
[blog](http://aturon.github.io/blog/2015/09/18/reuse/)
[posts](http://smallcultfollowing.com/babysteps/blog/2015/10/08/virtual-structs-part-4-extended-enums-and-thin-traits/)
and this [RFC issue](https://github.com/rust-lang/rfcs/issues/349)

`Deref` trait 是为实现自定义指针类型而设计的。
目的是让它通过指向`T`的指针到达`T`，而不是在不同类型之间转换。
遗憾的是，这一点并没有（也许不能）由trait定义强制执行。

Rust试图在显式和隐式机制之间取得谨慎的平衡，倾向于类型之间的显式转换。
点运算符中的自动解引用是一个人机工程学强烈支持隐式机制的情况，但其目的是将其限制在间接程度上，而不是在任意类型之间的转换。

## 参见

- [集合是智能指针的惯常做法](../idioms/deref.md).
- 为了较少的模板代码的代表crate [delegate](https://crates.io/crates/delegate)
  或[ambassador](https://crates.io/crates/ambassador)
- [`Deref` trait文档](https://doc.rust-lang.org/std/ops/trait.Deref.html).

> Latest commit fb57f21 on 10 Mar 2021