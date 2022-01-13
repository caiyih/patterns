# FFI中的错误处理

## 描述

在C语言等外部语言中，错误是由返回码来表示的。
然而，Rust的类型系统允许通过一个完整的类型来捕获和传播更丰富的错误信息。

这个最佳实践展示了不同种类的错误代码，以及如何以一种可用的方式暴露它们：

1. 简单枚举应该被转换为整数，并作为代码返回。
2. 结构化的枚举应该被转换为整数代码，并有一个字符串错误消息作为细节。
3. 自定义错误类型应该变得”透明“，用C表示。

## 代码示例

### 简单枚举

```rust,ignore
enum DatabaseError {
    IsReadOnly = 1, // user attempted a write operation
    IOError = 2, // user should read the C errno() for what it was
    FileCorrupted = 3, // user should run a repair tool to recover it
}

impl From<DatabaseError> for libc::c_int {
    fn from(e: DatabaseError) -> libc::c_int {
        (e as i8).into()
    }
}
```

### 结构化枚举

```rust,ignore
pub mod errors {
    enum DatabaseError {
        IsReadOnly,
        IOError(std::io::Error),
        FileCorrupted(String), // message describing the issue
    }

    impl From<DatabaseError> for libc::c_int {
        fn from(e: DatabaseError) -> libc::c_int {
            match e {
                DatabaseError::IsReadOnly => 1,
                DatabaseError::IOError(_) => 2,
                DatabaseError::FileCorrupted(_) => 3,
            }
        }
    }
}

pub mod c_api {
    use super::errors::DatabaseError;

    #[no_mangle]
    pub extern "C" fn db_error_description(
        e: *const DatabaseError
        ) -> *mut libc::c_char {

        let error: &DatabaseError = unsafe {
            // SAFETY: pointer lifetime is greater than the current stack frame
            &*e
        };

        let error_str: String = match error {
            DatabaseError::IsReadOnly => {
                format!("cannot write to read-only database");
            }
            DatabaseError::IOError(e) => {
                format!("I/O Error: {}", e);
            }
            DatabaseError::FileCorrupted(s) => {
                format!("File corrupted, run repair: {}", &s);
            }
        };

        let c_error = unsafe {
            // SAFETY: copying error_str to an allocated buffer with a NUL
            // character at the end
            let mut malloc: *mut u8 = libc::malloc(error_str.len() + 1) as *mut _;

            if malloc.is_null() {
                return std::ptr::null_mut();
            }

            let src = error_str.as_bytes().as_ptr();

            std::ptr::copy_nonoverlapping(src, malloc, error_str.len());

            std::ptr::write(malloc.add(error_str.len()), 0);

            malloc as *mut libc::c_char
        };

        c_error
    }
}
```

### 自定义错误类型

```rust,ignore
struct ParseError {
    expected: char,
    line: u32,
    ch: u16
}

impl ParseError { /* ... */ }

/* Create a second version which is exposed as a C structure */
#[repr(C)]
pub struct parse_error {
    pub expected: libc::c_char,
    pub line: u32,
    pub ch: u16
}

impl From<ParseError> for parse_error {
    fn from(e: ParseError) -> parse_error {
        let ParseError { expected, line, ch } = e;
        parse_error { expected, line, ch }
    }
}
```

## 优势

这就保证了外部语言可以清楚地获得错误信息，同时完全不影响Rust代码的API。

## 劣势

这是很大的工作量，有些类型可能不容易被转换为C语言中的表示。

> Latest commit 606bcff on 26 Feb 2021