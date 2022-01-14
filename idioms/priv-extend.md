# `#[non_exhaustive]`和私有字段的可扩展性

## 描述

在一小部分情况下，库作者可能想在不破坏后向兼容性的情况下，为公共结构体添加公共字段或为枚举添加新的变体。

Rust为这个问题提供了两种解决方案：

- 在`struct`，`enum`和`enum`变体上使用`#[non_exhaustive]`。
  关于所有可以使用`#[non_exhaustive]`的地方的详细文档，见[文档](https://doc.rust-lang.org/reference/attributes/type_system.html#the-non_exhaustive-attribute)。

- 你可以向结构体添加一个私有字段，以防止它被直接实例化或与之匹配（见备选方案）。

## 例子

```rust
mod a {
    // Public struct.
    #[non_exhaustive]
    pub struct S {
        pub foo: i32,
    }
    
    #[non_exhaustive]
    pub enum AdmitMoreVariants {
        VariantA,
        VariantB,
        #[non_exhaustive]
        VariantC { a: String }
    }
}

fn print_matched_variants(s: a::S) {
    // Because S is `#[non_exhaustive]`, it cannot be named here and
    // we must use `..` in the pattern.
    let a::S { foo: _, ..} = s;
    
    let some_enum = a::AdmitMoreVariants::VariantA;
    match some_enum {
        a::AdmitMoreVariants::VariantA => println!("it's an A"),
        a::AdmitMoreVariants::VariantB => println!("it's a b"),

        // .. required because this variant is non-exhaustive as well
        a::AdmitMoreVariants::VariantC { a, .. } => println!("it's a c"),

        // The wildcard match is required because more variants may be
        // added in the future
        _ => println!("it's a new variant")
    }
}
```

## 备选方案：结构体的`Private fields`

`#[non_exhaustive]`只适用于跨crate边界的情况。
在一个crate内，可以使用私有字段方法。

在结构体中添加字段基本上是一个向后兼容的变化。
然而，如果客户端使用某种模式来解构结构体实例，他们可能会命名结构体中的所有字段，而添加新字段会破坏这种模式。
客户端可以命名一些字段并在模式中使用`..`，在这种情况下，添加另一个字段是向后兼容的。
将结构体中的至少一个字段设置为私有，迫使客户端使用后一种形式的模式，确保结构体是面向未来的。

这种方法的缺点是，你可能需要在结构体中添加一个原本不需要的字段。
你可以使用`()`类型，这样就没有运行时的开销，并在字段名前加上`_`，以避免未使用字段的警告。

```rust
pub struct S {
    pub a: i32,
    // Because `b` is private, you cannot match on `S` without using `..` and `S`
    //  cannot be directly instantiated or matched against
    _b: ()
}
```

## 讨论

在`struct`上，`#[non_exhaustive]`允许以向后兼容的方式添加额外字段。
它也会阻止客户端使用结构体的构造器，即使所有字段都是公开的。
这可能很有帮助，但值得考虑的是，你是否*希望*额外的字段被客户端发现是一个编译器错误，而不是默默地不被发现。

`#[non_exhaustive]`也可以应用于枚举的变体。
`#[non_exhaustive]`变体的行为与`#[non_exhaustive]`结构体的行为相同。

慎重使用：在添加字段或变体时，增加主版本通常是更好的选择。
`#[non_exhaustive]`可能适用于这样的情况：你正在为一个可能与你的库不同步变化的外部资源建模，但这不是一个通用工具。

### 劣势

`#[non_exhaustive]`会使你的代码使用起来更不符合人体工程学，特别是在被迫处理未知的枚举变体的时候。
它应该只在需要这些改变，却又**不需要**递增主版本时使用。

当`#[non_exhaustive]`被应用于`enum`时，它迫使客户端处理通配符变体。
如果在这种情况下没有采取合理的行动，这可能会导致丑陋的代码和只在极其罕见情况下才会执行的代码路径。
如果客户端在这种情况下决定`panic!()`，那么在编译时暴露这个错误可能会更好。
事实上，`#[non_exhaustive]`迫使客户端处理"Something else"的情况；在这种情况下，很少有明智的行动可以采取。

## 参见

- [为枚举和结构体引入#[non_exhaustive]属性的RFC](https://github.com/rust-lang/rfcs/blob/master/text/2008-non-exhaustive.md)

> Latest commit 567a1f1 on 1 Sep 2021