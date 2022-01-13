# 传递字符串

## 描述

当向FFI函数传递字符串时，应该遵循四个原则：

1. 使拥有的字符串的生命周期尽可能长。
2. 在转换过程中尽量减少`unsafe`代码。
3. 如果C代码可以修改字符串数据，使用`Vec`而不是`CString`。
4. 除非外部函数API要求，否则字符串的所有权不应该转移给被调用者。

## 动机

Rust内置了对C风格字符串的支持，有`CString`和`CStr`类型。
然而，对于从Rust函数中发送字符串到外部函数调用，我们可以采取不同的方法。

最好的做法很简单：用`CString`的方式来减少`unsafe`的代码。
然而，次要的注意事项是，*对象必须活得足够长*，这意味着生命周期应该最大化。
此外，文档解释说，`CString`进行"round-tripping"修改是未定义行为，所以在这种情况下需要额外的工作。

## 代码示例

```rust,ignore
pub mod unsafe_module {

    // other module content

    extern "C" {
        fn seterr(message: *const libc::c_char);
        fn geterr(buffer: *mut libc::c_char, size: libc::c_int) -> libc::c_int;
    }

    fn report_error_to_ffi<S: Into<String>>(
        err: S
    ) -> Result<(), std::ffi::NulError>{
        let c_err = std::ffi::CString::new(err.into())?;

        unsafe {
            // SAFETY: calling an FFI whose documentation says the pointer is
            // const, so no modification should occur
            seterr(c_err.as_ptr());
        }

        Ok(())
        // The lifetime of c_err continues until here
    }

    fn get_error_from_ffi() -> Result<String, std::ffi::IntoStringError> {
        let mut buffer = vec![0u8; 1024];
        unsafe {
            // SAFETY: calling an FFI whose documentation implies
            // that the input need only live as long as the call
            let written: usize = geterr(buffer.as_mut_ptr(), 1023).into();

            buffer.truncate(written + 1);
        }

        std::ffi::CString::new(buffer).unwrap().into_string()
    }
}
```

## 优势

这个例子的编写方式是为了确保：

1. `unsafe`块尽可能小。
2. `CString`存活得足够久。
3. 类型转换的错误被尽可能传播。

一个常见的错误（常见到在文档中）是不在第一个块中使用变量：

```rust,ignore
pub mod unsafe_module {

    // other module content

    fn report_error<S: Into<String>>(err: S) -> Result<(), std::ffi::NulError> {
        unsafe {
            // SAFETY: whoops, this contains a dangling pointer!
            seterr(std::ffi::CString::new(err.into())?.as_ptr());
        }
        Ok(())
    }
}
```

这段代码将导致一个悬垂指针，因为`CString`的生命周期并没有因为指针的创建而延长，这与创建引用的情况不同。

另一个经常提出的问题是，初始化1k个零的向量是“慢”的。
然而，最近的Rust版本实际上将这个特殊的宏优化为对`zmalloc`的调用，这意味着它的速度和操作系统返回零内存的能力一样快（这相当快）。

## 劣势

没有？

> Latest commit 606bcff on 26 Feb 2021