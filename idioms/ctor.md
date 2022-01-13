# 构造器

## 描述

Rust没有构造器作为语言构造。
相反，惯例是使用一个[关联函数][]`new`来创建一个对象：

```rust
/// Time in seconds.
///
/// # Example
///
/// ```
/// let s = Second::new(42);
/// assert_eq!(42, s.value());
/// ```
pub struct Second {
    value: u64
}

impl Second {
    // Constructs a new instance of [`Second`].
    // Note this is an associated function - no self.
    pub fn new(value: u64) -> Self {
        Self { value }
    }

    /// Returns the value in seconds.
    pub fn value(&self) -> u64 {
        self.value
    }
}
```

## 默认构造器

Rust通过[`Default`][std-default]trait支持默认构造器：

```rust
/// Time in seconds.
///
/// # Example
///
/// ```
/// let s = Second::default();
/// assert_eq!(0, s.value());
/// ```
pub struct Second {
    value: u64
}

impl Second {
    /// Returns the value in seconds.
    pub fn value(&self) -> u64 {
        self.value
    }
}

impl Default for Second {
    fn default() -> Self {
        Self { value: 0 }
    }
}
```

如果所有类型的所有字段都实现了`Default`，也可以派生出`Default`，就像对`Second`那样：

```rust
/// Time in seconds.
///
/// # Example
///
/// ```
/// let s = Second::default();
/// assert_eq!(0, s.value());
/// ```
#[derive(Default)]
pub struct Second {
    value: u64
}

impl Second {
    /// Returns the value in seconds.
    pub fn value(&self) -> u64 {
        self.value
    }
}
```

**注意：**当为一个类型实现`Default`时，既不需要也不建议同时提供一个没有参数的相关函数`new`。

**提示：**实现或派生`Default`的好处是，你的类型现在可以用于需要实现`Default`的地方，最突出的是标准库中的任何[`*or_default`函数][std-or-default]。

## 参见

- [default 习语](default.md)对`Default`trait更深入的描述。

- [生成器模式](../patterns/creational/builder.md)用于构建有多种配置的对象。

[关联函数]: https://doc.rust-lang.org/stable/book/ch05-03-method-syntax.html#associated-functions
[std-default]: https://doc.rust-lang.org/stable/std/default/trait.Default.html
[std-or-default]: https://doc.rust-lang.org/stable/std/?search=or_default

> Latest commit fa8e722 on 22 Nov 2021