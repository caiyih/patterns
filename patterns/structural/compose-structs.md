# 将结构体组合在一起以获得更好的借用

TODO - 这不是一个简洁的名字

## 描述

有时一个大的结构体会给借用检查器带来问题——虽然字段可以被独立借用，但有时整个结构体最终会被一次性使用，从而妨碍其他用途。
一个解决方案可能是将该结构体分解为几个较小的结构体。
然后将这些结构体组合为原始结构体。
然后每个结构体都可以被单独借用，并具有更灵活的行为。

这往往会在其他方面带来更好的设计：应用这种设计模式往往能发现更小的功能单元。

## 例子

下面是一个精心设计的例子，说明借用检查器挫败了我们使用结构体的计划：

```rust
struct A {
    f1: u32,
    f2: u32,
    f3: u32,
}

fn foo(a: &mut A) -> &u32 { &a.f2 }
fn bar(a: &mut A) -> u32 { a.f1 + a.f3 }

fn baz(a: &mut A) {
    // The later usage of x causes a to be borrowed for the rest of the function.
    let x = foo(a);
    // Borrow checker error:
    // let y = bar(a); // ~ ERROR: cannot borrow `*a` as mutable more than once
                       //          at a time
    println!("{}", x);
}
```

我们可以应用这种设计模式，将`A`重构为两个较小的结构体，从而解决借用检查问题：

```rust
// A is now composed of two structs - B and C.
struct A {
    b: B,
    c: C,
}
struct B {
    f2: u32,
}
struct C {
    f1: u32,
    f3: u32,
}

// These functions take a B or C, rather than A.
fn foo(b: &mut B) -> &u32 { &b.f2 }
fn bar(c: &mut C) -> u32 { c.f1 + c.f3 }

fn baz(a: &mut A) {
    let x = foo(&mut a.b);
    // Now it's OK!
    let y = bar(&mut a.c);
    println!("{}", x);
}
```

## 动机

TODO 为什么以及在哪里应该使用该模式。

## 优势

让你可以绕过借用检查器的限制。

通常会产生一个更好的设计。

## 劣势

导致更多冗长的代码。

有时，较小的结构体并不是很好的抽象，所以我们最终得到了一个更糟糕的设计。
这可能是一种“代码气味”，表明该程序应该以某种方式进行重构。

## 讨论

这种模式在没有借用检查器的语言中是不需要的，所以从这个意义上说是Rust独有的。
然而，将功能单元做得更小，往往能使代码更简洁：这是软件工程中公认的原则，与语言无关。

这个模式依赖于Rust的借用检查器能够独立借用字段。
在这个例子中，借用检查器知道`a.b`和`a.c`是不同的，可以独立借用，它不会试图借用`a`的全部，这将使这个模式毫无用处。

> Latest commit 606bcff on 26 Feb 2021