# 类型合并

## 描述

这种模式的设计是为了允许优雅地处理多个相关类型，同时最大限度地减少内存不安全的表面积。

Rust的别名规则的基石之一是生命周期。
这确保了类型之间的许多访问模式都是内存安全的，包括数据竞争安全。

然而，当Rust类型被输出到其他语言时，它们通常被转化为指针。
在Rust中，指针意味着“用户管理着被指向者的生命周期”。
避免内存不安全是他们的责任。

因此需要对用户的代码有一定程度的信任，特别是在Rust无能为力的释放后使用方面。
然而，有些API设计对另一种语言编写的代码造成的负担比其他设计更重。

风险最低的API是“综合封装”，即与一个对象的所有可能的交互都被放入一个“封装器类型”中，保持着Rust API的整洁。

## 代码示例

为了理解这一点，让我们看一下导出API的一个经典例子：通过一个集合进行迭代。

该API看起来像这样：

1. 迭代器被初始化为`first_key`。
2. 每次调用`next_key'将推进迭代器。
3. 如果迭代器在最后，调用`next_key`将不做任何事情。
4. 如上所述，迭代器被“包裹”在集合中（与原始Rust API不同）。

如果迭代器高效地实现了`nth()`，那么就有可能使它对每个函数的调用都是短暂的：

```rust,ignore
struct MySetWrapper {
    myset: MySet,
    iter_next: usize,
}

impl MySetWrapper {
    pub fn first_key(&mut self) -> Option<&Key> {
        self.iter_next = 0;
        self.next_key()
    }
    pub fn next_key(&mut self) -> Option<&Key> {
        if let Some(next) = self.myset.keys().nth(self.iter_next) {
            self.iter_next += 1;
            Some(next)
        } else {
            None
        }
    }
}
```

这个封装器很简单，不包含任何`不安全`的代码。

## 优势

这使得API的使用更加安全，避免了类型之间的生命周期问题。
参见[基于对象的API](./export.md)以了解更多关于这样做的好处和避免的陷阱。

## 劣势

通常情况下，封装类型是相当困难的，有时Rust API的妥协会使事情变得更容易。

举个例子，考虑一个迭代器，它不能高效地实现`nth()`。
这绝对值得放入特殊的逻辑，使对象在内部处理迭代，或者高效地支持不同的只有外部函数API才会使用的访问模式。

### 尝试封装迭代器（失败）

为了将任何类型的迭代器正确地封装到API中，封装器需要做C版本的代码会做的事情：擦除迭代器的生命周期，并手动管理它。

可以说，这是*相当*难的事情。

这里只是说明了*一个*陷阱。


`MySetWrapper`的第一个版本看起来像这样：

```rust,ignore
struct MySetWrapper {
    myset: MySet,
    iter_next: usize,
    // created from a transmuted Box<KeysIter + 'self>
    iterator: Option<NonNull<KeysIter<'static>>>,
}
```

用`transmute`来延长生命周期，用指针来隐藏它，这已经很难看了。
但它变得更加糟糕：*任何其他操作都会导致Rust的“未定义行为”*。

考虑到在迭代过程中，封装器中的`MySet`可以被其他函数操作，比如为它所迭代的键存储一个新的值。
API并不鼓励这样做，但事实上，一些类似的C库期望这样做。

`myset_store`的一个简单实现：

```rust,ignore
pub mod unsafe_module {

    // other module content

    pub fn myset_store(
        myset: *mut MySetWrapper,
        key: datum,
        value: datum) -> libc::c_int {

        // DO NOT USE THIS CODE. IT IS UNSAFE TO DEMONSTRATE A PROLBEM.

        let myset: &mut MySet = unsafe { // SAFETY: whoops, UB occurs in here!
            &mut (*myset).myset
        };

        /* ...check and cast key and value data... */

        match myset.store(casted_key, casted_value) {
            Ok(_) => 0,
            Err(e) => e.into()
        }
    }
}
```

如果这个迭代器在这个函数被调用时存在，我们就违反了Rust的别名规则之一。 
根据Rust的规定，这个块中的可变引用必须对该对象有*排他性*的访问。
如果迭代器仅仅存在，它就不是排他性的，所以我们有“未定义的行为”！[^1]

为了避免这种情况，我们必须有一种方法来确保可变引用真的是独占的。
这基本上意味着在迭代器的共享引用存在时将其清除，然后再重建它。
在大多数情况下，这仍然会比C版本的效率低。

有些人可能会问：C语言怎么能更有效地做到这一点？
答案是，它作弊了。Rust的别名规则是问题所在，而C只是简单地为了它的指针忽略这些问题。
作为交换，我们经常可以看到在手册中声明在某些或所有情况下“非线程安全”的代码。
事实上，[GNU C library](https://manpages.debian.org/buster/manpages/attributes.7.en.html)有一整个词库专门讨论并发行为！

Rust宁愿让所有的内存都是安全的，既为了安全，也为了优化，这是C代码无法达到的。
被拒绝使用某些捷径是Rust程序员需要付出的代价。

[^1]: 对于那些迷惑不解的C程序员来说，迭代器不需要在这段引起未定义行为的代码中被读取。排他性规则也使编译器优化可能导致迭代器的共享引用出现不一致的观察（例如堆栈溢出或为提高效率而重新排序的指令）。
这些观察可能发生在可变引用创建后的*任何时间*。

> Latest commit 606bcff on 26 Feb 2021