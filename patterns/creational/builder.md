# 生成器

## 描述

通过对生成器助手的调用构造一个对象。

## 例子

```rust
#[derive(Debug, PartialEq)]
pub struct Foo {
    // Lots of complicated fields.
    bar: String,
}

impl Foo {
    // This method will help users to discover the builder
    pub fn builder() -> FooBuilder {
        FooBuilder::default()
    }
}

#[derive(Default)]
pub struct FooBuilder {
    // Probably lots of optional fields.
    bar: String,
}

impl FooBuilder {
    pub fn new(/* ... */) -> FooBuilder {
        // Set the minimally required fields of Foo.
        FooBuilder {
            bar: String::from("X"),
        }
    }

    pub fn name(mut self, bar: String) -> FooBuilder {
        // Set the name on the builder itself, and return the builder by value.
        self.bar = bar;
        self
    }

    // If we can get away with not consuming the Builder here, that is an
    // advantage. It means we can use the FooBuilder as a template for constructing
    // many Foos.
    pub fn build(self) -> Foo {
        // Create a Foo from the FooBuilder, applying all settings in FooBuilder
        // to Foo.
        Foo { bar: self.bar }
    }
}

#[test]
fn builder_test() {
    let foo = Foo {
        bar: String::from("Y"),
    };
    let foo_from_builder: Foo = FooBuilder::new().name(String::from("Y")).build();
    assert_eq!(foo, foo_from_builder);
}
```

## 动机

当你需要许多构造器或构造有副作用时，这很有用。

## 优势

将构建的方法与其他方法分开。

防止构造器的泛滥。

可用于单行的初始化，也可用于更复杂的构造。

## 劣势

比直接创建一个结构体对象，或一个简单的构造器更复杂。

## 讨论

这种模式在Rust中比其他许多语言更频繁地出现（对于更简单的对象），因为Rust缺乏重载。
因为你只能有一个给定名称的单一方法，所以在Rust中拥有多个构造器就不如在C++、Java或其他语言中那么好。

这种模式通常用于生成器对象本身就很有用，而不仅仅是一个生成器。
例如，[`std::process::Command`](https://doc.rust-lang.org/std/process/struct.Command.html)是[`Child`](https://doc.rust-lang.org/std/process/struct.Child.html)（一个进程）的生成器。
在这些情况下，不使用`T'和`TBuilder'的命名模式。

这个例子通过值传递的方式获取并返回生成器。
通常情况下，将生成器作为一个可变引用来获取和返回，更符合人体工程学（也更高效）。
借用检查器使这一工作自然进行。 
这种方法的好处是，人们可以写出像这样的代码：

```rust,ignore
let mut fb = FooBuilder::new();
fb.a();
fb.b();
let f = fb.build();
```

以及`FooBuilder::new().a().b().build()`风格。

## 参见

- [风格指南中的描述](https://web.archive.org/web/20210104103100/https://doc.rust-lang.org/1.12.0/style/ownership/builders.html)
- [derive_builder](https://crates.io/crates/derive_builder)，这是个自动实现这种模式的crate，同时避免了模板代码。
- [Constructor pattern](../../idioms/ctor.md)用于构造比较简单的时候。
- [生成器模式(wikipedia)](https://en.wikipedia.org/wiki/Builder_pattern)
- [复杂值的构造](https://web.archive.org/web/20210104103000/https://rust-lang.github.io/api-guidelines/type-safety.html#c-builder)

> Latest commit 9834f57 on 25 Aug 2021