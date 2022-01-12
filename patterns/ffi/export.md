# 基于对象的API

## 描述

当在Rust中设计暴露于其他语言的API时，有一些重要的设计原则与正常的Rust API设计相反：

1. 所有的封装类型都应该被Rust*拥有*，由用户*管理*，并且*不透明*。
2. 所有的事务性数据类型都应该由用户*拥有*，并且是*透明*的。
3. 所有的库行为应该是作用于封装类型的函数。
4. 所有的库行为都应该被封装成类型，且不是基于结构，而是基于*出处/生命周期*。

## 动机

Rust有对其他语言的内置FFI支持。
它为crate作者提供一种方法，通过不同的ABI（尽管这对这种做法并不重要）提供与C兼容的API。

设计良好的Rust FFI遵循了C语言API的设计原则，同时在Rust中尽可能地减少设计的妥协。任何外部API都有三个目标：

1. 使其易于在目标语言中使用。
2. 尽可能避免API在Rust侧控制内部不安全性。
3. 尽可能地减少内存不安全性和Rust`undefined behaviour`的可能性。

Rust代码必须在一定程度上相信外部语言的内存安全性。
然而，Rust侧每一点`unsafe`的代码都是产生错误的机会，或者加剧了`undefined behaviour`。

例如，如果一个指针的出处是错误的，这可能是由于无效的内存访问造成的段错误。
同时，如果它被不安全的代码所操纵，它就可能成为全面的堆损坏。

基于对象的API设计允许编写具有良好内存安全特性的垫片代码，拥有明确的安全界限。

## 代码示例

POSIX标准定义了访问文件式数据库的API，被称为[DBM](https://web.archive.org/web/20210105035602/https://www.mankier.com/0p/ndbm.h)。
它是一个“基于对象”的API的优秀例子。

下面是C语言的定义，对参与FFI的人来说应该很容易读懂。
下面的评论应该有助于解释细微差别。

```C
struct DBM;
typedef struct { void *dptr, size_t dsize } datum;

int     dbm_clearerr(DBM *);
void    dbm_close(DBM *);
int     dbm_delete(DBM *, datum);
int     dbm_error(DBM *);
datum   dbm_fetch(DBM *, datum);
datum   dbm_firstkey(DBM *);
datum   dbm_nextkey(DBM *);
DBM    *dbm_open(const char *, int, mode_t);
int     dbm_store(DBM *, datum, datum, int);
```

这个API定义了两种类型：`DBM`和`datum`。

`DBM`类型即上文所称的“封装类型”。
它被设计为包含内部状态，并作为库行为的入口。

它对用户是完全不透明的，用户不能自己创建一个`DBM`，因为他们不知道它的大小和布局。
相反，他们必须调用`dbm_open`，而这只能给他们一个*指向`DBM`的指针*。

这意味着所有的`DBM`在Rust意义上是由库“拥有”的。
未知大小的内部状态被保存在由库控制的内存中，而不是用户。
用户只能通过`open`和`close`来管理它的生命周期，并通过其他函数对它进行操作。

`datum`类型即上文所称的“事务性数据类型”。
它被设计用来促进库和用户之间的信息交流。

该数据库被设计用来存储“非结构化数据”，没有预先定义的长度或意义。
因此，`datum`相当于C语言中的Rust slice：一串字节，以及有多少个字节的计数。主要的区别是没有类型信息，也就是`void`所表示的。

请记住，这个头文件是从库的角度来写的。
用户可能有一些他们正在使用的类型，这些类型有已知的大小。
但是库并不关心，根据C语言的转换规则，指针后面的任何类型都可以被转换为`void`。

如前所述，这种类型对用户来说是*透明*的，同时这个类型也是由用户*拥有*的。
由于其内部指针，这有微妙的影响。
问题是，谁拥有这个指针所指向的内存？

对于最佳的内存安全性来说，答案是“用户”。
但是在诸如检索一个值的情况下，用户不知道如何正确地分配它（因为他们不知道这个值有多长）。
在这种情况下，库的代码应该使用用户可以访问的堆——比如C库的`malloc`和`free`——然后在Rust意义上*转移所有权*。

这似乎都是猜测，但这就是C语言中指针的含义。
它和Rust的意思是一样的：“用户定义的生命周期”。
库的用户需要阅读文档，以便正确使用它。
也就是说，有一些决定，如果用户做错了，会产生或大或小的后果。
尽量减少这些是这个最佳实践的目的，关键是要*转移一切透明事务的所有权*。

## 优势

这使用户必须坚持的内存安全保证的数量降到相对较少：

1. 不要用不是由`dbm_open`返回的指针调用任何函数（无效访问或损坏）。
2. 关闭之后，不要在指针上调用任何函数（在free后使用）。
3. 任何`datum`上的`dptr`必须是`NULL`，或者指向一个有效的内存片，其长度为所声明的长度。

此外，它还避免了很多指针出处的问题。
为了理解原因，让我们深入考虑一个替代方案：键的迭代。

Rust的迭代器是众所周知的。
当实现一个迭代器时，程序员会给它的所有者做一个单独的类型，有一定的生命周期，并实现`Iterator`trait。

下面是在Rust中对`DBM`进行迭代的方法:

```rust,ignore
struct Dbm { ... }

impl Dbm {
    /* ... */
    pub fn keys<'it>(&'it self) -> DbmKeysIter<'it> { ... }
    /* ... */
}

struct DbmKeysIter<'it> {
    owner: &'it Dbm,
}

impl<'it> Iterator for DbmKeysIter<'it> { ... }
```

由于Rust的保证，这样做是干净的、习惯性的，而且是安全的。
然而，考虑一下一个直接的API翻译会是什么样子:

```rust,ignore
#[no_mangle]
pub extern "C" fn dbm_iter_new(owner: *const Dbm) -> *mut DbmKeysIter {
    // THIS API IS A BAD IDEA! For real applications, use object-based design instead.
}
#[no_mangle]
pub extern "C" fn dbm_iter_next(
    iter: *mut DbmKeysIter,
    key_out: *const datum
) -> libc::c_int {
    // THIS API IS A BAD IDEA! For real applications, use object-based design instead.
}
#[no_mangle]
pub extern "C" fn dbm_iter_del(*mut DbmKeysIter) {
    // THIS API IS A BAD IDEA! For real applications, use object-based design instead.
}
```

这个API丢失了一个关键信息：迭代器的生命周期不能超过拥有它的`Dbm`对象的生命周期。
库的用户可以使用它，使迭代器的生命周期超过它所迭代的数据，从而导致读取未初始化的内存。

这个用C语言编写的例子包含一个错误，将在后面解释：

```C
int count_key_sizes(DBM *db) {
    // DO NOT USE THIS FUNCTION. IT HAS A SUBTLE BUT SERIOUS BUG!
    datum key;
    int len = 0;

    if (!dbm_iter_new(db)) {
        dbm_close(db);
        return -1;
    }

    int l;
    while ((l = dbm_iter_next(owner, &key)) >= 0) { // an error is indicated by -1
        free(key.dptr);
        len += key.dsize;
        if (l == 0) { // end of the iterator
            dbm_close(owner);
        }
    }
    if l >= 0 {
        return -1;
    } else {
        return len;
    }
}
```

这是一个经典bug。下面是迭代器返回迭代结束标记时的情况：

1. 循环条件将`l`设置为0，并进入循环，因为`0 >= 0`。
2. 长度递增，但在此情况下为0。
3. if语句为真，所以数据库被关闭。这里应该有一个break语句。
4. 循环条件再次执行，引起对已关闭对象的`next`调用。

这个错误最糟糕的地方是什么？
如果Rust的实现很小心的话，这段代码在大多数时候都能正常工作!
如果`Dbm`对象的内存没有被立即重用，内部检查几乎肯定会失败，导致迭代器返回一个`-1`表示错误。 
但偶尔也会造成段错误，甚至更糟糕的是，会造成无意义的内存损坏！

这些都不是Rust所能避免的。
从它的角度来看，它把这些对象放在了它的堆上，返回了它们的指针，并放弃了对它们生命周期的控制。
C语言的代码只是必须“玩得好”。

程序员必须阅读和理解API文档。
虽然有些人认为这在C语言中是理所当然的，但一个好的API设计可以减轻这种风险。
`DBM`的POSIX API通过将迭代器的所有权与它的父级合并来做到这一点。

```C
datum   dbm_firstkey(DBM *);
datum   dbm_nextkey(DBM *);
```

因此，所有的生命周期都被捆绑在一起，避免了不安全因素。

## 劣势

然而，这种设计选择也有一些缺点，也应予以考虑。

首先，API本身变得不那么具有表达性。
在POSIX DBM中，每个对象只有一个迭代器，而且每次调用都会改变其状态。
这比几乎所有语言中的迭代器都要限制得多，尽管它是安全的。
也许对于其他相关的对象，其生命周期没有那么多层次，这种限制比安全性更有代价。

其次，根据API各部分的关系，可能会涉及大量的设计工作。
许多比较容易的设计点都有其他模式与之相关：

- [类型合并](./wrappers.md)将多个Rust类型组合成一个不透明的“对象”。

- [FFI 错误传递](../../idioms/ffi/errors.md)解释了用整数值和哨兵返回值（如`NULL`指针）的错误处理。

- [接受外部字符串]](../../idioms/ffi/accepting-strings.md)允许以最小的不安全代码接受字符串，并且比[向FFI传递字符串](../../idioms/ffi/passing-strings.md)更容易做对。

然而，并不是每个API都可以这样做。
至于谁是他们的受众，则取决于程序员的最佳判断。

> Latest commit 9834f57 on 25 Aug 2021