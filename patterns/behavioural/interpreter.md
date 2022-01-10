# 解释器

## 描述

如果一个问题经常发生，并且需要长时间重复的步骤来解决，那么问题实例可能用一种简单的语言来表达，一个解释器对象可以通过解释用这种简单语言写的句子来解决这个问题。

基本上，对于我们定义的任何种类的问题：

- [领域特定语言](https://en.wikipedia.org/wiki/Domain-specific_language)，
- 该语言的语法，
- 解决问题实例的解释器。

## 动机

我们的目标是将简单的数学表达式翻译成后缀表达式（或[逆波兰表示法](https://en.wikipedia.org/wiki/Reverse_Polish_notation)）
为了简单起见，我们的表达式由十个数字`0`，...，`9`和两个操作符`+`，`-`组成。例如，表达式`2 + 4`被翻译成`2 4 +`。

## 有关我们问题的上下文无关文法

我们的任务是把中缀表达式翻译成后缀表达式。 
让我们为`0`, ..., `9`, `+`, 和`-`上的一组中缀表达式定义一个上下文无关文法，其中：

- 终结符： `0`, ..., `9`
- 非终结符： `exp`, `term`, `+`, `-`
- 起始符是`exp`
- 接下来是产生规则

```ignore
exp -> exp + term
exp -> exp - term
exp -> term
term -> 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9
```

**注意：** 这个文法应该根据我们要做的事情进行进一步的转化。例如，我们可能需要消除左递归。
详情请查阅[Compilers: Principles,Techniques, and Tools](https://en.wikipedia.org/wiki/Compilers:_Principles,_Techniques,_and_Tools)(aka Dragon Book).

## 解法

我们简单地实现了一个递归下降分析器。为了简单起见，当一个表达式在语法上出错时（例如，根据语法定义，`2-34`或`2+5-`是错误的），代码会panic。

```rust
pub struct Interpreter<'a> {
    it: std::str::Chars<'a>,
}

impl<'a> Interpreter<'a> {

    pub fn new(infix: &'a str) -> Self {
        Self { it: infix.chars() }
    }

    fn next_char(&mut self) -> Option<char> {
        self.it.next()
    }

    pub fn interpret(&mut self, out: &mut String) {
        self.term(out);

        while let Some(op) = self.next_char() {
            if op == '+' || op == '-' {
                self.term(out);
                out.push(op);
            } else {
                panic!("Unexpected symbol '{}'", op);
            }
        }
    }

    fn term(&mut self, out: &mut String) {
        match self.next_char() {
            Some(ch) if ch.is_digit(10) => out.push(ch),
            Some(ch) => panic!("Unexpected symbol '{}'", ch),
            None => panic!("Unexpected end of string"),
        }
    }
}

pub fn main() {
    let mut intr = Interpreter::new("2+3");
    let mut postfix = String::new();
    intr.interpret(&mut postfix);
    assert_eq!(postfix, "23+");

    intr = Interpreter::new("1-2+3-4");
    postfix.clear();
    intr.interpret(&mut postfix);
    assert_eq!(postfix, "12-3+4-");
}
```

## 讨论

可能有一种错误的看法，认为解释器设计模式是关于形式语言的文法设计和这些文法的分析器的实现。
事实上，这种模式是以一种更具体的方式来表达问题实例，并实现解决这些问题实例的函数/类/结构体。
Rust语言有`macro_rules!`，允许定义特殊的语法和如何将这种语法扩展到源代码的规则。

在下面的例子中，我们创建了一个简单的`macro_rules!`，计算`n`维向量的[欧几里得长度](https://en.wikipedia.org/wiki/Euclidean_distance)。
写`norm!(x,1,2)`可能比把`x,1,2`打包成一个`Vec`并调用一个计算长度的函数更容易表达和更有效率。

```rust
macro_rules! norm {
    ($($element:expr),*) => {
        {
            let mut n = 0.0;
            $(
                n += ($element as f64)*($element as f64);
            )*
            n.sqrt()
        }
    };
}

fn main() {
    let x = -3f64;
    let y = 4f64;

    assert_eq!(3f64, norm!(x));
    assert_eq!(5f64, norm!(x, y));
    assert_eq!(0f64, norm!(0, 0, 0)); 
    assert_eq!(1f64, norm!(0.5, -0.5, 0.5, -0.5));
}
```

## 参见

- [解释器模式](https://en.wikipedia.org/wiki/Interpreter_pattern)
- [上下文无关文法]](https://en.wikipedia.org/wiki/Context-free_grammar)
- [macro_rules!](https://doc.rust-lang.org/rust-by-example/macros.html)

> Latest commit 9834f57 on 25 Aug 2021