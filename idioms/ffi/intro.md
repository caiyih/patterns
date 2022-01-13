# FFI 惯常做法

编写FFI代码本身就是一个完整的课程。
然而，这里有几个惯常做法可以作为指导，避免没有经验的用户在`unsafe`Rust中踩坑。

本节包含了在开发FFI时可能有用的惯常做法。

1. [错误处理的惯常做法](./errors.md)——用整数值和哨兵返回值处理错误（如`NULL`指针）。

2. [接受字符串](./accepting-strings.md)，使用最少的不安全代码。

3. [传递字符串](./passing-strings.md)到FFI函数。

> Latest commit 606bcff on 26 Feb 2021