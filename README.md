# Rust设计模式

一本关于Rust编程语言设计模式和惯常做法开源书的简体中文译本，你可以在[这里](https://fomalhauthmj.github.io/patterns/)阅读。
本书的原始仓库为[Rust Design Patterns](https://github.com/rust-unofficial/patterns)，感谢每一位开源贡献者！

## 贡献方式

欢迎提出Issues和PR来做出贡献，包括但不限于：通知原始仓库的更新内容，校正翻译的错误或不足之处。


你可以查看原始仓库的[Umbrella issue](https://github.com/rust-unofficial/patterns/issues/116)以了解所有可能被添加的模式、反面模式和惯常做法。

建议阅读原始仓库的[贡献指南](./CONTRIBUTING.md)，以获得更多关于如何为这个仓库做贡献的信息。

## Building with mdbook

This book is built with [mdbook](https://rust-lang.github.io/mdBook/). You can
install it by running `cargo install mdbook`.

If you want to build it locally you can run one of these two commands in the root
directory of the repository:

- `mdbook build`

  Builds static html pages as output and place them in the `/book` directory by
  default.

- `mdbook serve`

  Serves the book at `http://localhost:3000` (port is changeable, take a look at
  the terminal output to be sure) and reloads the browser when a change occurs.

## License

The content of this repository is licensed under **MPL-2.0**; see [LICENSE](./LICENSE).
