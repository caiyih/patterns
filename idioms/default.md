# `Default` Trait

## 描述

Rust中的许多类型都有一个[构造器]。然而，这是类型*特殊*的；Rust不能抽象出“所有对象都具有`new()`方法”。
为了允许这一点，我们设想了[`Default`]trait，它可以用于容器和其他泛型（例如，见[`Option::unwrap_or_default()`]）。
值得注意的是，一些容器已经在适用的地方实现了它。

不仅像`Cow`、`Box`或`Arc`这样的单元素容器为所包含的`Default`类型实现了 `Default`，人们还可以自动为字段都实现了`Default`的结构体实现`#[derive(Default)]`，所以越多类型实现`Default`，它就越有用。

另一方面，构造器可以接受多个参数，而`default()`方法则不能。
甚至可以有多个名字不同的构造器，但每个类型只能有一个`Default`的实现。

## 例子

```rust
use std::{path::PathBuf, time::Duration};

// note that we can simply auto-derive Default here.
#[derive(Default, Debug, PartialEq)]
struct MyConfiguration {
    // Option defaults to None
    output: Option<PathBuf>,
    // Vecs default to empty vector
    search_path: Vec<PathBuf>,
    // Duration defaults to zero time
    timeout: Duration,
    // bool defaults to false
    check: bool,
}

impl MyConfiguration {
    // add setters here
}

fn main() {
    // construct a new instance with default values
    let mut conf = MyConfiguration::default();
    // do something with conf here
    conf.check = true;
    println!("conf = {:#?}", conf);
        
    // partial initialization with default values, creates the same instance
    let conf1 = MyConfiguration {
        check: true,
        ..Default::default()
    };
    assert_eq!(conf, conf1);
}
```

## 参见

- [构造器]惯常做法是另一种生成实例的方式，这些实例可能是也可能不是“默认”的。
- [`Default`] 文档 （向下滚动查看实现者列表）
- [`Option::unwrap_or_default()`]
- [`derive(new)`]

[构造器]: ctor.md
[`Default`]: https://doc.rust-lang.org/stable/std/default/trait.Default.html
[`Option::unwrap_or_default()`]: https://doc.rust-lang.org/stable/std/option/enum.Option.html#method.unwrap_or_default
[`derive(new)`]: https://crates.io/crates/derive-new/

> Latest commit 9834f57 on 25 Aug 2021