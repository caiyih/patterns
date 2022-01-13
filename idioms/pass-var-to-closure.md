# 传递变量到闭包

## 描述

默认情况下，闭包通过借用来捕获其环境。或者你可以使用 `move`-closure 来移动整个环境。
然而，你往往只想把一些变量转移到闭包中，给它一些数据的拷贝，通过引用传递，或者执行一些其他的转换。

为此，在单独的作用域中使用变量重绑定。

## 例子

使用

```rust
use std::rc::Rc;

let num1 = Rc::new(1);
let num2 = Rc::new(2);
let num3 = Rc::new(3);
let closure = {
    // `num1` is moved
    let num2 = num2.clone();  // `num2` is cloned
    let num3 = num3.as_ref();  // `num3` is borrowed
    move || {
        *num1 + *num2 + *num3;
    }
};
```

而不是

```rust
use std::rc::Rc;

let num1 = Rc::new(1);
let num2 = Rc::new(2);
let num3 = Rc::new(3);

let num2_cloned = num2.clone();
let num3_borrowed = num3.as_ref();
let closure = move || {
    *num1 + *num2_cloned + *num3_borrowed;
};
```

## 优势

复制的数据和闭包定义在一起，所以它们的目的更明确，而且即使它们没有被闭包消耗，也会被立即丢弃。

无论数据是被复制还是被移动，闭包都使用与周围代码相同的变量名。

## 劣势

闭包体的额外缩进。

> Latest commit 9834f57 on 25 Aug 2021