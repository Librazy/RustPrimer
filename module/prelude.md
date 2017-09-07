# Prelude

Rust 的标准库，有一个 `std::prelude` 模块，这里面包含了默认导入（ `std` 库是默认导入的，然后 `std::prelude` 下面的东西也是默认导入的）的所有符号。这相当于在所有的 crate 的根下添加了 `extern crate std;` ，并在每个模块添加 `use std::prelude::v1::*;` 。

当前第一版的 `std::prelude` （即 `std::prelude::v1` ）有以下内容：

```rust
std::marker::{Copy, Send, Sized, Sync}
std::ops::{Drop, Fn, FnMut, FnOnce}
std::mem::drop
std::boxed::Box
std::borrow::ToOwned
std::clone::Clone
std::cmp::{PartialEq, PartialOrd, Eq, Ord}
std::convert::{AsRef, AsMut, Into, From}
std::default::Default
std::iter::{Iterator, Extend, IntoIterator, DoubleEndedIterator, ExactSizeIterator}
std::option::Option::{self, Some, None}
std::result::Result::{self, Ok, Err}
std::slice::SliceConcatExt
std::string::{String, ToString}
std::vec::Vec
```

除了 `std::prelude` 引入了我们非常常用的一些符号之外，其他的一些模块和库也常常会包含一个 `prelude` 以引入常用的符号，而无需手动一个个引入它们，如 `std::io::prelude` 、 `std::os::unix::prelude` 等等。虽然它们不像 `std::prelude` 一样被默认引入，但相比手动引入所有的常用符号也方便了许多。