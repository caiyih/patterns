# `#![deny(warnings)]`

## 描述

一个善意的crate作者想确保他们的代码在构建时不会出现警告。所以他用以下内容来注释其crate根。

## 例子

```rust
#![deny(warnings)]

// All is well.
```

## 优势

注释很短，如果出现错误，会停止构建。

## 劣势

通过不允许编译器产生构建警告，crate作者失去了Rust引以为傲的稳定性。 
有时，新特性或旧的错误特性需要改变处理逻辑，因此，在转为`deny`之前，会有`warn`的lint，并有一定的缓冲期。

例如，人们发现一个类型可以有两个具有相同方法的`impl`块。
这被认为是一个坏主意，但为了使过渡顺利，`overlapping-inherent-impls` lint被引入，给那些偶然发现这个事实的人一个警告，即使它在未来的版本中将成为一个硬编码错误。

另外，有时API会被废弃，所以在它们消失前使用会发出警告。

当某些事情发生改变，所有这些都有潜在的破坏构建的可能性。

此外，提供额外lint的crate（例如[rust-clippy]）不能再被使用，除非注释被删除。这可以通过[-cap-lints]来缓解。
命令行参数`--cap-lints=warn`可将所有`deny`lint错误变成警告。

## 替代方案

有两种方法可以解决这个问题：第一，我们可以将构建设置与代码解耦；第二，我们可以指明我们想要显式拒绝的lint。

下面的命令行参数在所有警告设置为`deny`的情况下构建：

```RUSTFLAGS="-D warnings" cargo build```

这可以由任何个人开发者完成（或者在Travis这样的CI工具中设置，但请记住，当有变化时，这可能会破坏构建），而不需要对代码进行修改。

另外，我们可以在代码中指定我们想要`deny`的lint。
下面是一个（希望）可以安全拒绝的警告lint的列表（截至Rustc 1.48.0）:

```rust,ignore
#[deny(bad-style,
       const-err,
       dead-code,
       improper-ctypes,
       non-shorthand-field-patterns,
       no-mangle-generic-items,
       overflowing-literals,
       path-statements ,
       patterns-in-fns-without-body,
       private-in-public,
       unconditional-recursion,
       unused,
       unused-allocation,
       unused-comparisons,
       unused-parens,
       while-true)]
```

此外，以下`allow`lint可能是一个`deny`的好主意。

```rust,ignore
#[deny(missing-debug-implementations,
       missing-docs,
       trivial-casts,
       trivial-numeric-casts,
       unused-extern-crates,
       unused-import-braces,
       unused-qualifications,
       unused-results)]
```

有些人可能还想在他们的列表中加入`missing-copy-implementations`lint。

请注意，我们没有明确添加`deprecated`的lint，因为可以肯定的是，未来会有更多被废弃的API。

## 参见

- [所有的clippy lints](https://rust-lang.github.io/rust-clippy/master)
- [deprecate attribute]文档
- 输入`rustc -W help`可查看你系统上的lint。也可以输入
`rustc --help`查看选项。
- [rust-clippy]是一个用于写出更好的Rust代码的lint集合。

[rust-clippy]: https://github.com/Manishearth/rust-clippy
[deprecate attribute]: https://doc.rust-lang.org/reference/attributes.html#deprecation
[--cap-lints]: https://doc.rust-lang.org/rustc/lints/levels.html#capping-lints

> Latest commit 39a2f36 on 18 Oct 2021