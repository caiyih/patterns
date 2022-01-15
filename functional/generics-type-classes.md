# 作为类型类的泛型

## 描述

Rust的类型系统设计得更像函数式语言（如Haskell）而不是命令式语言（如Java和C++）。
因此，Rust可以把许多类型的编程问题变成“静态类型”问题。
这是选择函数式语言的最大优势之一，对Rust的许多编译时保证至关重要。

这个想法的一个关键部分是泛型的工作方式。例如，在C++和Java中，泛型是编译器的一个元编程结构。
C++中的`vector<int>`和`vector<char>`只是一个`vector`类型（被称为`template`）的相同模板代码的两个不同副本，其中填入了两种不同类型。

在Rust中，泛型参数创建了函数式语言中所谓的“类型类约束”，用户填写的每个不同的参数*实际上都会改变类型*。
换句话说，`Vec<isize>`和`Vec<char>`*是两种不同的类型*，被类型系统的所有部分识别为不同的类型。

这被称为**单态化**，不同的类型由**多态的**代码创建。 
这种特殊的行为需要`impl`块来指定泛型参数：泛型的不同值会导致不同的类型，而不同的类型可以有不同的`impl`块。

在面向对象的语言中，类可以从其父辈那里继承行为。
然而，这不仅允许将额外的行为附加到类型类的特定成员上，而且还允许附加到额外的行为上。

最接近的是Javascript和Python中的运行时多态性，在那里，新的成员可以被任意构造器随意地添加到对象中。
然而，与这些语言不同的是，Rust的所有额外方法在使用时都可以被类型检查，因为它们的泛型是静态定义的。
这使得它们在保持安全的同时更具有实用性。

## 例子

假设你正在为一系列的实验室机器设计一个存储服务器。
由于涉及到软件，有两个不同的协议需要你支持。BOOTP（用于PXE网络启动），和NFS（用于远程挂载存储）。

你的目标是有一个用Rust编写的程序，可以处理这两个协议。
它将有协议处理器，并监听两种请求。
然后，主要的应用逻辑将允许实验室管理员为实际文件配置存储和安全控制。

实验室里的机器对文件的请求包含相同的基本信息，无论它们来自什么协议：一个认证方法，和一个要检索的文件名。 
一个直接的实现会是这样的：

```rust,ignore

enum AuthInfo {
    Nfs(crate::nfs::AuthInfo),
    Bootp(crate::bootp::AuthInfo),
}

struct FileDownloadRequest {
    file_name: PathBuf,
    authentication: AuthInfo,
}
```

这种设计可能工作得足够好。
但现在假设你需要支持添加*协议特定*的元数据。
例如，对于NFS，你想确定他们的挂载点是什么，以便强制执行额外的安全规则。

当前结构体的设计方式将协议决定权留给了运行时。
这意味着任何适用于一种协议而不适用于另一种协议的方法都需要程序员在进行运行时检查。

以下是获得NFS挂载点的代码：

```rust,ignore
struct FileDownloadRequest {
    file_name: PathBuf,
    authentication: AuthInfo,
    mount_point: Option<PathBuf>,
}

impl FileDownloadRequest {
    // ... other methods ...

    /// Gets an NFS mount point if this is an NFS request. Otherwise,
    /// return None.
    pub fn mount_point(&self) -> Option<&Path> {
        self.mount_point.as_ref()
    }
}
```

`mount_point()`的每个调用者都必须检查`None`并编写代码来处理它。
即使他们知道在给定的代码路径中只有NFS请求会被使用。

如果不同的请求类型被混淆，产生编译时错误会更理想。
毕竟，用户的整个代码路径，包括他们使用库中的哪些函数，都会知道一个请求是NFS请求还是BOOTP请求。

在Rust中，这其实是可以做到的! 解决办法是*添加一个泛型*，以便分割API。

下面是它的代码：

```rust
use std::path::{Path, PathBuf};

mod nfs {
    #[derive(Clone)]
    pub(crate) struct AuthInfo(String); // NFS session management omitted
}

mod bootp {
    pub(crate) struct AuthInfo(); // no authentication in bootp
}

// private module, lest outside users invent their own protocol kinds!
mod proto_trait {
    use std::path::{Path, PathBuf};
    use super::{bootp, nfs};

    pub(crate) trait ProtoKind {
        type AuthInfo;
        fn auth_info(&self) -> Self::AuthInfo;
    }

    pub struct Nfs {
        auth: nfs::AuthInfo,
        mount_point: PathBuf,
    }

    impl Nfs {
        pub(crate) fn mount_point(&self) -> &Path {
            &self.mount_point
        }
    }

    impl ProtoKind for Nfs {
        type AuthInfo = nfs::AuthInfo;
        fn auth_info(&self) -> Self::AuthInfo {
            self.auth.clone()
        }
    }

    pub struct Bootp(); // no additional metadata

    impl ProtoKind for Bootp {
        type AuthInfo = bootp::AuthInfo;
        fn auth_info(&self) -> Self::AuthInfo {
            bootp::AuthInfo()
        }
    }
}

use proto_trait::ProtoKind; // keep internal to prevent impls
pub use proto_trait::{Nfs, Bootp}; // re-export so callers can see them

struct FileDownloadRequest<P: ProtoKind> {
    file_name: PathBuf,
    protocol: P,
}

// all common API parts go into a generic impl block
impl<P: ProtoKind> FileDownloadRequest<P> {
    fn file_path(&self) -> &Path {
        &self.file_name
    }

    fn auth_info(&self) -> P::AuthInfo {
        self.protocol.auth_info()
    }
}

// all protocol-specific impls go into their own block
impl FileDownloadRequest<Nfs> {
    fn mount_point(&self) -> &Path {
        self.protocol.mount_point()
    }
}

fn main() {
    // your code here
}
```

采用这种方法，如果用户使用了错误的类型：

```rust,ignore
fn main() {
    let mut socket = crate::bootp::listen()?;
    while let Some(request) = socket.next_request()? {
        match request.mount_point().as_ref()
            "/secure" => socket.send("Access denied"),
            _ => {} // continue on...
        }
        // Rest of the code here
    }
}
```

他们会得到一个语法错误。
`FileDownloadRequest<Bootp>`类型没有实现`mount_point()`，只有`FileDownloadRequest<Nfs>`类型实现。
而这是由NFS模块创建的，当然不是BOOTP模块!

## 优势

首先，它允许在多个状态下共有的字段被去掉重复。
通过使共享字段泛型化，保证其只被实现一次。

其次，它使`impl`块更容易阅读，因为它们是按状态分解的。
所有状态下通用的方法只在一个块中出现，而一个状态下特有的方法则在单独的块中出现。

这两点都意味着代码行数更少，而且组织得更好。

## 劣势

目前这增加了二进制文件的大小，这是由于编译器中实现单态化的方式造成的。
希望这种实现方式在未来能够得到改善。

## 替代方案

* 如果一个类型由于构造或部分初始化而似乎需要一个“分开的API”，可以考虑用[生成器模式](../patterns/creational/builder.md)来代替。

* 如果类型之间的API不发生变化——只有行为发生变化——那么最好使用[策略模式](../patterns/behavioural/strategy.md)代替。

## 参见

这种模式在整个标准库中都被使用：

* `Vec<u8>`可以从一个字符串中转换出，与其他类型的`Vec<T>`不同。[^1]
* 当它们只包含一个实现了`Ord`trait的类型时，它们也可以被转换到二叉堆中。[^2]
* `to_string`方法是只针对`str`类型的`Cow`。[^3]

它也被几个流行的crate使用，以允许API的灵活性:

* 用于嵌入式设备的`embedded-hal`生态系统广泛使用了这种模式。
例如，它允许静态地验证用于控制嵌入式引脚的设备寄存器的配置。
当一个引脚进入一个模式时，它返回一个`Pin<MODE>`结构体，其泛型决定了在该模式下可用的功能，这些功能不在`Pin`本身上。[^4]

* `hyper`HTTP客户端库利用这一点为不同的可插拔请求提供了丰富的API。
不同连接器的客户端有不同的方法以及不同的trait实现，而一组核心方法适用于任何连接器。[^5]

* “类型状态”模式——对象根据内部状态或不变量获得和失去API——在Rust中使用相同的基本概念和稍微不同的技术来实现。[^6]

[^1]: [impl From\<CString\> for Vec\<u8\>]( https://doc.rust-lang.org/stable/src/std/ffi/c_str.rs.html#799-801)

[^2]: [impl\<T\> From\<Vec\<T, Global\>\> for BinaryHeap\<T\>](https://doc.rust-lang.org/stable/src/alloc/collections/binary_heap.rs.html#1345-1354)

[^3]: [impl\<'\_\> ToString for Cow\<'\_, str>]( https://doc.rust-lang.org/stable/src/alloc/string.rs.html#2235-2240)

[^4]: [https://docs.rs/stm32f30x-hal/0.1.0/stm32f30x_hal/gpio/gpioa/struct.PA0.html](https://docs.rs/stm32f30x-hal/0.1.0/stm32f30x_hal/gpio/gpioa/struct.PA0.html)

[^5]: [https://docs.rs/hyper/0.14.5/hyper/client/struct.Client.html](https://docs.rs/hyper/0.14.5/hyper/client/struct.Client.html)

[^6]: [类型状态模式的例子]( https://web.archive.org/web/20210325065112/https://www.novatec-gmbh.de/en/blog/the-case-for-the-typestate-pattern-the-typestate-pattern-itself/)和[Rusty Typestate系列（一篇扩展论文）](https://web.archive.org/web/20210328164854/https://rustype.github.io/notes/notes/rust-typestate-series/rust-typestate-index)

> Latest commit 7e96169 on 15 Sep 2021