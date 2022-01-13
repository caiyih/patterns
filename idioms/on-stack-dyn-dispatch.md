# 栈上动态分发

## 描述

我们可以对多个值进行动态分发，然而，要做到这一点，我们需要声明多个变量来绑定不同类型的对象。
为了根据需要延长生命周期，我们可以使用延迟条件初始化，如下所示：

## 例子

```rust
use std::io;
use std::fs;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let arg = "-";

    // These must live longer than `readable`, and thus are declared first:
    let (mut stdin_read, mut file_read);

    // We need to ascribe the type to get dynamic dispatch.
    let readable: &mut dyn io::Read = if arg == "-" {
        stdin_read = io::stdin();
        &mut stdin_read
    } else {
        file_read = fs::File::open(arg)?;
        &mut file_read
    };

    // Read from `readable` here.

    Ok(())
}
```

## 动机

Rust默认会对代码进行单态处理。这意味着每一种类型的代码都会被生成一个副本，并被独立优化。
虽然这允许在热点路径上产生非常快的代码，但它也会在性能不重要的地方使代码变得臃肿，从而耗费编译时间和缓存使用量。


幸运的是，Rust允许我们使用动态分发，但我们必须明确要求它。

## 优势

我们不需要在堆上分配任何东西。
我们也不需要初始化一些我们以后不会用到的东西，也不需要把下面的整个代码单一化，以便`File`或`Stdin`一起工作。

## 劣势

该代码需要比基于`Box`的版本有更多的移动语义部分。

```rust,ignore
// We still need to ascribe the type for dynamic dispatch.
let readable: Box<dyn io::Read> = if arg == "-" {
    Box::new(io::stdin())
} else {
    Box::new(fs::File::open(arg)?)
};
// Read from `readable` here.
```

## 讨论

Rust新手通常会了解到，Rust要求所有变量在*使用前*被初始化，所以很容易忽略这样一个事实，即*未使用的*变量很可能是未初始化的。
Rust非常努力地确保这一点，而且只有初始化过的值在其作用域的末端被丢弃。

这个例子符合Rust对我们的所有约束：

* 所有的变量在使用（本例中为借用）之前都被初始化。
* 每个变量只持有单一类型的值。在我们的例子中，`stdin`是`Stdin`类型，`file`是`File`类型，`readable`是`&mut dyn Read`类型。
* 每个被借用值的生命周期都比它的所有借用引用要久。

## 参见

* [析构器中的最终处理](dtor-finally.md)和[RAII守护对象](../patterns/behavioural/RAII.md)可以从对生命周期的严格控制中获益。
* 对于条件填充的`Option<T>`的（可变）引用，可以直接初始化一个`Option<T>`，并使用其[`.as_ref()`]方法来获取其引用。

[`.as_ref()`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.as_ref

> Latest commit a152399 on 21 Apr 2021