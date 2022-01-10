# 命令

## 描述

命令模式的基本思想是将行动分离成它自己的对象，并将它们作为参数传递。

## 动机

假设我们有一连串的行动或事务被封装为对象。
我们希望这些行动或命令之后在不同的时间以某种顺序被执行或调用。
这些命令也可能因某些事件而被触发。
例如，当用户按下一个按钮，或在一个数据包到达时。
此外，这些命令可能是可撤销的。这可能对编辑器的操作很有用。 
我们可能想存储已执行命令的日志，这样，如果系统崩溃，我们可以之后重新应用这些变化。

## 例子

定义两个数据库操作`create table`和`add field`。每一个操作都是一个可撤销的命令，例如，`drop table`和`remove field`。
当用户调用数据库迁移操作时，那么每条命令都按照定义的顺序执行，当用户调用回滚操作时，那么整个命令集将以相反的顺序调用。

## 方法：使用 trait 对象

我们定义了一个共同的trait，用两个操作`execute`和`rollback`来封装我们的命令。所有的命令`structs`必须实现这个trait。

```rust
pub trait Migration {
    fn execute(&self) -> &str;
    fn rollback(&self) -> &str;
}

pub struct CreateTable;
impl Migration for CreateTable {
    fn execute(&self) -> &str {
        "create table"
    }
    fn rollback(&self) -> &str {
        "drop table"
    }
}

pub struct AddField;
impl Migration for AddField {
    fn execute(&self) -> &str {
        "add field"
    }
    fn rollback(&self) -> &str {
        "remove field"
    }
}

struct Schema {
    commands: Vec<Box<dyn Migration>>,
}

impl Schema {
    fn new() -> Self {
        Self { commands: vec![] }
    }

    fn add_migration(&mut self, cmd: Box<dyn Migration>) {
        self.commands.push(cmd);
    }

    fn execute(&self) -> Vec<&str> {
        self.commands.iter().map(|cmd| cmd.execute()).collect()
    }
    fn rollback(&self) -> Vec<&str> {
        self.commands
            .iter()
            .rev() // reverse iterator's direction
            .map(|cmd| cmd.rollback())
            .collect()
    }
}

fn main() {
    let mut schema = Schema::new();

    let cmd = Box::new(CreateTable);
    schema.add_migration(cmd);
    let cmd = Box::new(AddField);
    schema.add_migration(cmd);

    assert_eq!(vec!["create table", "add field"], schema.execute());
    assert_eq!(vec!["remove field", "drop table"], schema.rollback());
}
```

## 方法：使用函数指针

我们可以遵循另一种方法，将每个单独的命令创建为不同的函数，并存储函数指针，以便以后在不同的时间调用这些函数。
由于函数指针实现了所有三个trait `Fn`,`FnMut`和`FnOnce`，我们也可以传递和存储闭包而不是函数指针。

```rust
type FnPtr = fn() -> String;
struct Command {
    execute: FnPtr,
    rollback: FnPtr,
}

struct Schema {
    commands: Vec<Command>,
}

impl Schema {
    fn new() -> Self {
        Self { commands: vec![] }
    }
    fn add_migration(&mut self, execute: FnPtr, rollback: FnPtr) {
        self.commands.push(Command { execute, rollback });
    }
    fn execute(&self) -> Vec<String> {
        self.commands.iter().map(|cmd| (cmd.execute)()).collect()
    }
    fn rollback(&self) -> Vec<String> {
        self.commands
            .iter()
            .rev()
            .map(|cmd| (cmd.rollback)())
            .collect()
    }
}

fn add_field() -> String {
    "add field".to_string()
}

fn remove_field() -> String {
    "remove field".to_string()
}

fn main() {
    let mut schema = Schema::new();
    schema.add_migration(|| "create table".to_string(), || "drop table".to_string());
    schema.add_migration(add_field, remove_field);
    assert_eq!(vec!["create table", "add field"], schema.execute());
    assert_eq!(vec!["remove field", "drop table"], schema.rollback());
}
```

## 方法：使用 `Fn` trait 对象

最后，我们可以将实现`Fn`trait的每个命令分别存储在向量中，而不是定义一个共同的命令trait。

```rust
type Migration<'a> = Box<dyn Fn() -> &'a str>;

struct Schema<'a> {
    executes: Vec<Migration<'a>>,
    rollbacks: Vec<Migration<'a>>,
}

impl<'a> Schema<'a> {
    fn new() -> Self {
        Self {
            executes: vec![],
            rollbacks: vec![],
        }
    }
    fn add_migration<E, R>(&mut self, execute: E, rollback: R)
    where
        E: Fn() -> &'a str + 'static,
        R: Fn() -> &'a str + 'static,
    {
        self.executes.push(Box::new(execute));
        self.rollbacks.push(Box::new(rollback));
    }
    fn execute(&self) -> Vec<&str> {
        self.executes.iter().map(|cmd| cmd()).collect()
    }
    fn rollback(&self) -> Vec<&str> {
        self.rollbacks.iter().rev().map(|cmd| cmd()).collect()
    }
}

fn add_field() -> &'static str {
    "add field"
}

fn remove_field() -> &'static str {
    "remove field"
}

fn main() {
    let mut schema = Schema::new();
    schema.add_migration(|| "create table", || "drop table");
    schema.add_migration(add_field, remove_field);
    assert_eq!(vec!["create table", "add field"], schema.execute());
    assert_eq!(vec!["remove field", "drop table"], schema.rollback());
}
```

## 讨论

如果我们的命令很小，并且可以被定义为函数或者作为一个闭包传递，那么使用函数指针可能是更好的，因为它没有利用动态分发。 
但如果我们的命令是一个完整的结构体，其中有一堆函数和变量被定义为独立的模块，那么使用trait对象会更合适。
应用案例可以在[`actix`](https://actix.rs/)中找到，它在为路由注册处理函数时使用trait对象。
在使用`Fn`trait对象的情况下，我们可以用与函数指针相同的方式创建和使用命令。

关于性能，在性能和代码的简单性和组织性之间总是有一个权衡。
静态分发可以提供更快的性能，而动态分发在我们构造应用程序时提供了灵活性。

## 参见

- [命令模式](https://en.wikipedia.org/wiki/Command_pattern)

- [命令模式的另一个例子](https://web.archive.org/web/20210223131236/https://chercher.tech/rust/command-design-pattern-rust)

> Latest commit 9834f57 on 25 Aug 2021