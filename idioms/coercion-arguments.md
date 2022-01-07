# 使用借用类型作为参数

## 描述

当你决定为一个函数参数使用哪种参数类型时，使用解引用强制转换的目标可以增加你代码的灵活性。
通过这种方式，该函数将接受更多的输入类型。

这并不限于可切片或胖指针类型。
事实上，你应该总是倾向于使用**借用类型**而不是**借用所有类型**。
例如`&str`而不是`&String`，`&[T]`而不是`&Vec<T>`，以及`&T`而不是`&Box<T>`。

使用借用类型，你可以避免已经提供一层间接性的所有类型上的多层间接。例如，`String`有一层间接，所以`&String`会有两层间接。我们可以通过使用`&str`来避免这种情况，并且让`&String`在函数被调用时强制变成`&str`。

## 例子

在这个例子中，我们将说明使用`&String`作为函数参数与使用`&str`的一些区别，但这些想法也适用于使用`&Vec<T>`与使用`&[T]`或使用`&Box<T>`与使用`&T`。

考虑这样一个例子，我们希望确定一个词是否包含三个连续的元音。我们不需要拥有字符串来确定这一点，所以我们将使用一个引用。

代码可能看起来像这样：

```rust
fn three_vowels(word: &String) -> bool {
    let mut vowel_count = 0;
    for c in word.chars() {
        match c {
            'a' | 'e' | 'i' | 'o' | 'u' => {
                vowel_count += 1;
                if vowel_count >= 3 {
                    return true
                }
            }
            _ => vowel_count = 0
        }
    }
    false
}

fn main() {
    let ferris = "Ferris".to_string();
    let curious = "Curious".to_string();
    println!("{}: {}", ferris, three_vowels(&ferris));
    println!("{}: {}", curious, three_vowels(&curious));

    // This works fine, but the following two lines would fail:
    // println!("Ferris: {}", three_vowels("Ferris"));
    // println!("Curious: {}", three_vowels("Curious"));

}
```

这样做没问题，因为我们传递的是一个`&String`类型作为参数。
如果我们在最后两行取消注释，这个例子就会失败，因为`&str`类型不会被强制变成`&String`类型。我们可以通过简单地修改参数的类型来解决这个问题。

例如，如果我们把我们的函数声明改成：

```rust, ignore
fn three_vowels(word: &str) -> bool {
```

那么这两个版本都会编译并打印相同的输出。

```bash
Ferris: false
Curious: true
```

但等等，这还不是全部！这个话题还有更多的内容。
很可能你会对自己说：这并不重要，无论如何我都不会使用`&'static str`作为输入（就像我们使用`"Ferris"`时那样）。
即使忽略这个特殊的例子，你仍然会发现使用`&str`会比使用`&String`更灵活。

现在我们来举个例子，有人给了我们一个句子，我们想确定句子中的任何一个词是否包含三个连续的元音。我们也许应该利用我们已经定义的函数，简单地输入句子中的每个词。

这个例子可能是这样的:

```rust
fn three_vowels(word: &str) -> bool {
    let mut vowel_count = 0;
    for c in word.chars() {
        match c {
            'a' | 'e' | 'i' | 'o' | 'u' => {
                vowel_count += 1;
                if vowel_count >= 3 {
                    return true
                }
            }
            _ => vowel_count = 0
        }
    }
    false
}

fn main() {
    let sentence_string =
        "Once upon a time, there was a friendly curious crab named Ferris".to_string();
    for word in sentence_string.split(' ') {
        if three_vowels(word) {
            println!("{} has three consecutive vowels!", word);
        }
    }
}
```

使用我们声明的参数类型为`&str`的函数运行这个例子将产生如下结果

```bash
curious has three consecutive vowels!
```

然而，当我们的函数以参数类型`&String`声明时，这个例子将无法运行。这是因为字符串切片是一个`&str`，而不是一个`&String`，后者需要一次内存分配来转换为`&String`，这不是隐式的，而从`String`转换为`&str`开销很低，而且是隐式的。

## 参见

- [Rust语言参考中关于类型强制转换](https://doc.rust-lang.org/reference/type-coercions.html)
- 关于如何处理`String`和`&str`的更多讨论见
  [blog系列（2015）](https://web.archive.org/web/20201112023149/https://hermanradtke.com/2015/05/03/string-vs-str-in-rust-functions.html)
  by Herman J. Radtke III

> Latest commit dca0dfd on Dec 16 2021