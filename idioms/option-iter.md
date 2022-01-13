# `Option`的迭代

## 描述

`Option`可以被看作是一个包含零或一个元素的容器。
特别是，它实现了`IntoIterator`trait，因此可以用于需要这种类型的通用代码。

## 例子

由于`Option`实现了`IntoIterator`，它可以作为[`.extend()`](https://doc.rust-lang.org/std/iter/trait.Extend.html#tymethod.extend)的一个参数：

```rust
let turing = Some("Turing");
let mut logicians = vec!["Curry", "Kleene", "Markov"];

logicians.extend(turing);

// equivalent to
if let Some(turing_inner) = turing {
    logicians.push(turing_inner);
}
```

如果你需要把一个`Option`粘到现有迭代器的末尾，你可以把它传递给[`.chain()`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.chain):

```rust
let turing = Some("Turing");
let logicians = vec!["Curry", "Kleene", "Markov"];

for logician in logicians.iter().chain(turing.iter()) {
    println!("{} is a logician", logician);
}
```

注意，如果`Option`总是`Some`，那么在元素上使用[`std::iter::once`](https://doc.rust-lang.org/std/iter/fn.once.html)更常见。

另外，由于`Option`实现了`IntoIterator`，所以可以使用`for`循环对其进行迭代。
这相当于用`if let Some(..)`来匹配它，在大多数情况下，你应该选择后者。

## 参见

* [`std::iter::once`](https://doc.rust-lang.org/std/iter/fn.once.html)是一个迭代器，正好产生一个元素。
它是`Some(foo).into_iter()`的一个更易读的替代品。

* [`Iterator::filter_map`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.filter_map)是[`Iterator::flat_map`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.flat_map)的一个版本，专门用于返回`Option`的映射函数。

* [`ref_slice`](https://crates.io/crates/ref_slice)crate提供了将`Option`转换为零或单个元素切片的函数。

* [`Option<T>`文档](https://doc.rust-lang.org/std/option/enum.Option.html)

> Latest commit 9834f57 on 25 Aug 2021