# 临时可变性

## 描述

通常在准备和处理一些数据后，数据只是被检查，而不会被修改。
这个意图可以通过重新定义可变变量为不可变的来明确。

这可以通过在嵌套块内处理数据或重新定义变量来实现。

## 例子

假定向量在使用前必须进行排序。

使用嵌套块：

```rust,ignore
let data = {
    let mut data = get_vec();
    data.sort();
    data
};

// Here `data` is immutable.
```

使用变量重绑定：

```rust,ignore
let mut data = get_vec();
data.sort();
let data = data;

// Here `data` is immutable.
```

## 优势

由编译器来确保你不会在某个时间点之后意外地改变数据。

## 劣势

嵌套块需要额外缩进。
多写一行，从块中返回数据或重新定义变量。

> Latest commit 2cd70a5 on 22 Jan 2021