# 策略（也称 政策）

## 描述

[策略设计模式](https://en.wikipedia.org/wiki/Strategy_pattern)是一种实现关注点分离的技术。它还允许通过[依赖反转](https://en.wikipedia.org/wiki/Dependency_inversion_principle)来解耦软件模块。

策略模式的基本思想是，给定一个解决特定问题的算法，我们只在抽象层面上定义算法的骨架，并将具体的算法实现分成不同的部分。

这样，使用该算法的客户可以选择一个具体的实现，而一般的算法工作流程保持不变。
换句话说，类的抽象规范并不取决于派生类的具体实现，但具体实现必须遵守抽象规范。
这就是为什么我们称之为“依赖反转”。

## 动机

想象一下，我们正在做一个每月都会生成报告的项目。
我们需要以不同的格式（策略）生成报告，例如，以`JSON`或`Plain Text`格式。
但事情随着时间的推移而变化，我们不知道未来可能得到什么样的要求。
例如，我们可能需要以一种全新的格式生成我们的报告，或者只是修改现有的一种格式。

## 例子

在这个例子中，我们的不变量（或抽象）是`Context`、`Formatter`和`Report`，而`Text`和`Json`是我们的策略结构体。
这些策略必须实现`Formatter`的trait。

```rust
use std::collections::HashMap;

type Data = HashMap<String, u32>;

trait Formatter {
    fn format(&self, data: &Data, buf: &mut String);
}

struct Report;

impl Report {
    // Write should be used but we kept it as String to ignore error handling
    fn generate<T: Formatter>(g: T, s: &mut String) {
        // backend operations...
        let mut data = HashMap::new();
        data.insert("one".to_string(), 1);
        data.insert("two".to_string(), 2);
        // generate report
        g.format(&data, s);
    }
}

struct Text;
impl Formatter for Text {
    fn format(&self, data: &Data, buf: &mut String) {
        for (k, v) in data {
            let entry = format!("{} {}\n", k, v);
            buf.push_str(&entry);
        }
    }
}

struct Json;
impl Formatter for Json {
    fn format(&self, data: &Data, buf: &mut String) {
        buf.push('[');
        for (k, v) in data.into_iter() {
            let entry = format!(r#"{{"{}":"{}"}}"#, k, v);
            buf.push_str(&entry);
            buf.push(',');
        }
        buf.pop(); // remove extra , at the end
        buf.push(']');
    }
}

fn main() {
    let mut s = String::from("");
    Report::generate(Text, &mut s);
    assert!(s.contains("one 1"));
    assert!(s.contains("two 2"));

    s.clear(); // reuse the same buffer
    Report::generate(Json, &mut s);
    assert!(s.contains(r#"{"one":"1"}"#));
    assert!(s.contains(r#"{"two":"2"}"#));
}
```

## 优势

主要优势是关注点分离。
例如，在这种情况下，`Report`对`Json`和`Text`的具体实现一无所知，而输出实现则不关心数据如何被预处理、存储和获取。
他们唯一需要知道的是上下文和要实现的特定trait和方法，即`Formatter`和`format`。

## 劣势

每个策略必须至少有一个模块，所以模块的数量随着策略的数量而增加。
如果有许多策略可供选择，那么用户就必须知道策略之间有什么不同。

## 讨论

在前面的例子中，所有策略都在一个文件中实现。
提供不同策略的方法包括：

- 都在一个文件中（如本例所示，类似于作为模块分离的情况）
- 作为模块分开，例如，`formatter::json`模块，`formatter::text`模块
- 使用编译器特性标记，例如`json`特征，`text`特征
- 作为crate分开，例如：`json`crate，`text`crate

Serde crate是`策略`模式在实践中的一个好例子。
Serde允许通过为我们的类型手动实现`Serialize`和`Deserialize`trait来对序列化行为进行[完全定制](https://serde.rs/custom-serialization.html)。
例如，我们可以很容易地将`serde_json`与`serde_cbor`交换，因为它们暴露了类似的方法。
有了这一点，使得助手crate`serde_transcode`更加有用和符合人体工程学。

然而，我们不需要使用traits就可以在Rust中设计这种模式。

下面的玩具例子演示了使用Rust`closures`策略模式的想法：

```rust
struct Adder;
impl Adder {
    pub fn add<F>(x: u8, y: u8, f: F) -> u8
    where
        F: Fn(u8, u8) -> u8,
    {
        f(x, y)
    }
}

fn main() {
    let arith_adder = |x, y| x + y;
    let bool_adder = |x, y| {
        if x == 1 || y == 1 {
            1
        } else {
            0
        }
    };
    let custom_adder = |x, y| 2 * x + y;

    assert_eq!(9, Adder::add(4, 5, arith_adder));
    assert_eq!(0, Adder::add(0, 0, bool_adder));
    assert_eq!(5, Adder::add(1, 3, custom_adder));
}

```

事实上，Rust已经在`Options`的`map`方法中使用了这个想法：

```rust
fn main() {
    let val = Some("Rust");

    let len_strategy = |s: &str| s.len();
    assert_eq!(4, val.map(len_strategy).unwrap());

    let first_byte_strategy = |s: &str| s.bytes().next().unwrap();
    assert_eq!(82, val.map(first_byte_strategy).unwrap());
}
```

## 参见

- [策略模式](https://en.wikipedia.org/wiki/Strategy_pattern)
- [依赖注入](https://en.wikipedia.org/wiki/Dependency_injection)
- [基于政策的设计](https://en.wikipedia.org/wiki/Modern_C++_Design#Policy-based_design)

> Latest commit 9834f57 on 25 Aug 2021