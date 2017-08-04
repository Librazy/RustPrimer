# 注释与文档

## 注释

在 Rust 里面注释分成两种，行注释和块注释。它的形式和 C 语言是一样的。两种注释分别是：

1. 行注释使用 `//` 放在注释前面。比如:

        ```rust
        // I love Rust, but I hate Rustc.
        ```
2. 块注释分别使用 `/*` 和 `*/` 包裹需要注释的内容。比如：

        ```rust
        /* W-Cat 是个大胖猫，N-Cat 是个高度近视猫。*/

        /* Rust 支持 /* 嵌套 */
        块注释 */
        ```

## 文档

Rust 自带有文档功能的注释，分别是 `///` 和 `//!` 。支持 Markdown 格式

1. `///` 用来描述的它后面接着的项，置于要说明的对象上方，一般用于函数或结构体（字段）的说明。
2. `//!` 用来描述包含它的项，一般用在模块文件的头部。

文档注释内部可使用markdown格式的标记语法，可用于 rustdoc 工具的自动文档提取。

```rust
//! # The first line
//! The second line
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let five = 5;
///
/// assert_eq!(6, add_one(5));
/// # fn add_one(x: i32) -> i32 {
/// #     x + 1
/// # }
/// ```
fn add_one(x: i32) -> i32 {
        x + 1
}
```

```rust
//! # The Rust Standard Library
//!
//! The Rust Standard Library provides the essential runtime
//! functionality for building portable Rust software.
```

文档注释也可以使用 `/** ... */` 与 `/*! ... */` 的形式。

### 生成 html 文档

使用 `cargo doc` 可以生成 crate 的 html 文档，在 `target/doc` 文件夹下。或者也可以用 `rustdoc main.rs` 生成。
