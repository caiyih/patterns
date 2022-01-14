# 简单的文档初始化

## 描述

如果一个结构体需要花费大量精力来初始化，那么在编写文档时，用一个将结构体作为参数的辅助函数来包装你的例子可能会更快。

## 动机

有时，一个结构体有多个或复杂的参数和几个方法。
这些方法中的每一个都应该有例子。

例如：

```rust,ignore
struct Connection {
    name: String,
    stream: TcpStream,
}

impl Connection {
    /// Sends a request over the connection.
    ///
    /// # Example
    /// ```no_run
    /// # // Boilerplate are required to get an example working.
    /// # let stream = TcpStream::connect("127.0.0.1:34254");
    /// # let connection = Connection { name: "foo".to_owned(), stream };
    /// # let request = Request::new("RequestId", RequestType::Get, "payload");
    /// let response = connection.send_request(request);
    /// assert!(response.is_ok());
    /// ```
    fn send_request(&self, request: Request) -> Result<Status, SendErr> {
        // ...
    }

    /// Oh no, all that boilerplate needs to be repeated here!
    fn check_status(&self) -> Status {
        // ...
    }
}
```

## 例子

与其输入所有这些模板代码来创建一个`Connection`和`Request`，不如直接创建一个将它们作为参数的包装辅助函数：

```rust,ignore
struct Connection {
    name: String,
    stream: TcpStream,
}

impl Connection {
    /// Sends a request over the connection.
    ///
    /// # Example
    /// ```
    /// # fn call_send(connection: Connection, request: Request) {
    /// let response = connection.send_request(request);
    /// assert!(response.is_ok());
    /// # }
    /// ```
    fn send_request(&self, request: Request) {
        // ...
    }
}
```

**注意：**在上面的例子中，`assert!(response.is_ok());`这一行在测试时不会实际运行，因为它是在一个从未被调用的函数中。

## 优势

更简洁，避免了例子中的重复代码。

## 劣势

由于例子是在一个函数中，代码将不会被测试。
尽管在运行`cargo test`时，它仍然会被检查，以确保它能编译。
所以当你需要`no_run`时，这种模式是最有用的。有了这个，你不需要添加`no_run`。

## 讨论

如果不需要断言，这种模式很好用。

如果需要，另一种方法是创建一个公共方法来创建一个帮助器实例，该方法被标注为`#[doc(hidden)]`（这样用户就不会看到它）。
然后这个方法可以在rustdoc内部被调用，因为它是crate公共API的一部分。

> Latest commit 9834f57 on 25 Aug 2021