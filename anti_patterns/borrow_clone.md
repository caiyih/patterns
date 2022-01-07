# 通过Clone来满足借用检查器

## 描述

借用检查器通过确保以下两种情况来防止Rust用户开发不安全的代码：只存在一个可变引用，或者可能存在多个但都是不可变引用。
如果编写的代码不符合这些条件，当开发者通过克隆变量来解决编译器错误时，就会出现这种反面模式。

## 例子

```rust
// define any variable
let mut x = 5;

// Borrow `x` -- but clone it first
let y = &mut (x.clone());

// perform some action on the borrow to prevent rust from optimizing this
//out of existence
*y += 1;

// without the x.clone() two lines prior, this line would fail on compile as
// x has been borrowed
// thanks to x.clone(), x was never borrowed, and this line will run.
println!("{}", x);
```

## 动机

特别是对初学者来说，用这种模式来解决借用检查器的迷惑问题是很诱人的。
然而，这有严重的后果。使用`.clone()`会导致数据被复制。
两者之间的任何变化都是不同步的——就像存在两个完全独立的变量一样。

有一些特殊情况——`Rc<T>`被设计用来智能地处理克隆。
它在内部精确地管理着一份数据的副本，克隆它只会克隆引用。

还有`Arc<T>`，它提供了在堆中分配的T类型的值的共享所有权。
对`Arc`调用`.clone()`会产生一个新的`Arc`实例，它指向与源`Arc`相同的堆上的分配，同时增加一个引用计数。

一般来说，克隆应该是深思熟虑的，并充分了解后果。
如果克隆被用来使借用检查器的错误消失，那就很可能说明这种反面模式可能在使用。

尽管`.clone()`暗示着不好的模式，但有时**写低效的代码也是可以的**，例如在以下情况下：

- 开发者仍然对所有权不熟悉
- 代码没有很大的速度或内存限制（如黑客马拉松项目或原型）
- 满足借用检查器相当复杂，而你更愿意优化可读性而不是性能

如果怀疑有不必要的克隆，在评估是否需要克隆之前，应该充分理解[Rust Book的所有权章节](https://doc.rust-lang.org/book/ownership.html)

此外，请确保在你的项目中始终运行`cargo clippy`，它将检测到一些不需要`.clone()`的情况，例如[1](https://rust-lang.github.io/rust-clippy/master/index.html#redundant_clone),
[2](https://rust-lang.github.io/rust-clippy/master/index.html#clone_on_copy),
[3](https://rust-lang.github.io/rust-clippy/master/index.html#map_clone)或[4](https://rust-lang.github.io/rust-clippy/master/index.html#clone_double_ref).

## 参见

- [在发生改变的枚举中使用`mem::{take(_), replace(_)}`来保留所有值](../idioms/mem-replace.md)
- [`Rc<T>`文档，用于智能地处理.clone()](http://doc.rust-lang.org/std/rc/)
- [`Arc<T>`文档，线程安全的引用计数指针](https://doc.rust-lang.org/std/sync/struct.Arc.html)
- [Rust中关于所有权的技巧](https://web.archive.org/web/20210120233744/https://xion.io/post/code/rust-borrowchk-tricks.html)

> Latest commit 9834f57 on 25 Aug 2021