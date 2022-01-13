# 有守护的RAII

## 描述

[RAII][wikipedia]代表"Resource Acquisition is Initialisation"，”资源获取即初始化“。
该模式的本质是，资源初始化在对象的构造器中完成，最终化（资源释放）在析构器中完成。
这种模式在Rust中得到了扩展，即使用RAII对象作为某些资源的守护对象，并依靠类型系统来确保访问总是由守护对象来调解。

## 例子

互斥守护是std库中这种模式的典型例子（这是真正实现的简化版本）：

```rust,ignore
use std::ops::Deref;

struct Foo {}

struct Mutex<T> {
    // We keep a reference to our data: T here.
    //..
}

struct MutexGuard<'a, T: 'a> {
    data: &'a T,
    //..
}

// Locking the mutex is explicit.
impl<T> Mutex<T> {
    fn lock(&self) -> MutexGuard<T> {
        // Lock the underlying OS mutex.
        //..

        // MutexGuard keeps a reference to self
        MutexGuard {
            data: self,
            //..
        }
    }
}

// Destructor for unlocking the mutex.
impl<'a, T> Drop for MutexGuard<'a, T> {
    fn drop(&mut self) {
        // Unlock the underlying OS mutex.
        //..
    }
}

// Implementing Deref means we can treat MutexGuard like a pointer to T.
impl<'a, T> Deref for MutexGuard<'a, T> {
    type Target = T;

    fn deref(&self) -> &T {
        self.data
    }
}

fn baz(x: Mutex<Foo>) {
    let xx = x.lock();
    xx.foo(); // foo is a method on Foo.
    // The borrow checker ensures we can't store a reference to the underlying
    // Foo which will outlive the guard xx.

    // x is unlocked when we exit this function and xx's destructor is executed.
}
```

## 动机

如果一个资源在使用后必须进行最终处理，RAII可以用来进行最终处理。
如果在最终处理后访问该资源是一个错误，那么这个模式可以用来防止这种错误。

## 优势

防止在资源没有最终处理和在最终处理后使用资源时出现错误。

## 讨论

RAII是一种有用的模式，可以确保资源被适当地取消分配或被最终处理。
我们可以利用Rust中的借用检查器来静态地防止在最终处理完成后使用资源所产生的错误。

借用检查器的核心目的是确保对数据的引用不会超过该数据的生命周期。
RAII守护模式之所以有效，是因为守护对象包含了对底层资源的引用，并且只暴露了这种引用。
Rust确保守护对象不能超过底层资源的生命周期，并且守护对象所调解资源的引用不能超过守护对象的生命周期。
为了解这一点，检查一下没有生命周期标注的`deref`的签名是有帮助的。

```rust,ignore
fn deref<'a>(&'a self) -> &'a T {
    //..
}
```

返回的资源引用与`self`具有相同的生命周期(`'a`)。
因此，借用检查器确保对`T`的引用的生命周期短于（不超过）`self`的生命周期。

请注意，实现`Deref`并不是这个模式的核心部分，它只是让使用守护对象更符合人体工程学。
在守护对象上实现一个`get`方法也同样有效。

## 参见

[惯常做法：析构器中的最终处理](../../idioms/dtor-finally.md)

RAII是C++中的一种常见模式：[cppreference.com](http://en.cppreference.com/w/cpp/language/raii),
[wikipedia][wikipedia].

[wikipedia]: https://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization

[风格指南条目](https://doc.rust-lang.org/1.0.0/style/ownership/raii.html)
（目前仅是占位符）。

> Latest commit b809265 on 22 Apr 2021