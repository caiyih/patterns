# 倾向于较小的crates

## 描述

倾向于选择能做好一件事的较小的crates。

Cargo和crates.io使得添加第三方库变得很容易，比C或C++等语言要容易得多。
此外，由于crates.io上的包在发布后不能被编辑或删除，任何现在能工作的构建在未来也应该继续工作。
我们应该利用这种工具的优势，使用更小、更细的依赖关系。

## 优势

* 小的crates更容易理解，并鼓励更多的模块化代码。
* Crates允许在项目之间重用代码。
  例如，`url`crate是作为Servo浏览器引擎的一部分而开发的，但后来在该项目之外被广泛使用。
* 由于Rust的编译单元是crate，将一个项目分割成多个crate可以使更多的代码被并行构建。

## 劣势

* 当一个项目同时依赖一个crate的多个冲突版本时，这可能导致“依赖地狱”。例如，`url`crate有1.0和0.5两个版本。由于`url:1.0`的`Url`和`url:0.5`的`Url`是不同的类型，使用`url:0.5`的HTTP客户端将不接受来自使用`url:1.0`的Web爬虫的`Url`值。
* crates.io上的软件包没有经过组织。一个crate可能写得很差，有无用的文档，或直接是恶意的。
* 两个小crate的优化程度可能低于一个大crate，因为编译器默认不执行链接时优化（link-time optimization,LTO）。

## 例子

[`ref_slice`](https://crates.io/crates/ref_slice) 提供将`&T`转换为`&[T]`函数的crate。

[`url`](https://crates.io/crates/url)提供处理URLs工具的crate。

[`num_cpus`](https://crates.io/crates/num_cpus)提供一个函数来查询机器上的CPU数量的crate。

## 参见

* [crates.io: The Rust community crate host](https://crates.io/)

> Latest commit 881f51f on 8 Mar 2021