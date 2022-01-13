# 用`format!`串联字符串

## 描述

可以在可变的`String`上使用`push`和`push_str`方法来建立字符串，或者使用其`+`操作符。
然而，使用`format!`往往更方便，特别是在有字面和非字面字符串混合的地方。

## 例子

```rust
fn say_hello(name: &str) -> String {
    // We could construct the result string manually.
    // let mut result = "Hello ".to_owned();
    // result.push_str(name);
    // result.push('!');
    // result

    // But using format! is better.
    format!("Hello {}!", name)
}
```

## 优势

使用`format!`通常是组合字符串的最简洁和可读的方式。

## 劣势

这通常不是组合字符串的最有效的方法——对一个可变的字符串进行一系列`push`的操作通常是最有效的（特别是当字符串已经被预先分配到预期的大小时）。

> Latest commit 5f1425d on 5 Jan 2021