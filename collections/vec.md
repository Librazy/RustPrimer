# 动态数组 `Vec`

在第七章我们粗略介绍了一下 `Vec` 的用法。实际上，作为 Rust 中一个非常重要的数据类型，熟练掌握 Vec 的用法能大大提升我们在 Rust 世界中的编码能力。

## 特性及声明方式

和我们之前接触到的Array不同， `Vec` 具有动态的添加和删除元素的能力，并且能够以 `O(1)` 的效率进行随机访问。同时，对其尾部进行 `push` 或者 `pop` 操作的效率也是*平摊* `O(1)` 的。
同时，有一个非常重要的特性（虽然我们编程的时候大部分都不会考量它）就是， `Vec` 的所有内容项都是生成在堆空间上的，也就是说，你可以轻易的将 `Vec` move 出一个栈而不用担心内存拷贝影响执行效率——毕竟只是拷贝的栈上的指针。

另外的就是， `Vec<T>` 中的泛型 `T` 必须是 `Sized` 的，也就是说必须在编译的时候就知道存一个内容项需要多少内存。对于那些在编译时候未知大小的项（函数类型等），我们可以用 `Box` 将其包裹，当成一个指针。

### `Vec::new()`

我们可以用 `std::vec::Vec::new()` 的方式来声明一个Vec。

```rust
let mut v1: Vec<i32> = Vec::new();
```

这里需要注意的是， `new` 函数并没有提供一个能显式规定其泛型类型的参数，也就是说，上面的代码能根据 `v1` 的类型自动推导出 `Vec` 的泛型;但是，你不能写成如下的形式：

```rust
let mut v1 = Vec::new::<i32>();
// 与之对比的,collect函数就能指定：
// let mut v2 = (0i32..5).collect::<Vec<i32>>();
```

这是因为这两个函数的声明形式以及实现形式，在此，我们不做深究。

### `vec!` 宏

相比调用new函数，Rust提供了一种更加直观便捷的方式声明一个动态数组： `vec!` 宏。

```rust
let v: Vec<i32> = vec![];

// 以下语句相当于：
// let mut temp = Vec::new();
// temp.push(1);
// temp.push(2);
// temp.push(3);
// let v = temp;
let v = vec![1, 2, 3];

let v = vec![0; 10]; //注意分号，这句话声明了一个 内容为10个0的动态数组
```

### 从迭代器生成

因为 `Vec` 实现了 `FromIterator` 这个 trait ，因此，借助 `collect()` ，我们能将任意一个迭代器转换为 `Vec` 。

```rust
let v: Vec<_> = (1..5).collect();
```

## 访问及修改

### 随机访问

就像数组一样，因为 `Vec` 借助 `Index` 和 `IndexMut` 提供了随机访问的能力，我们通过 `[index]` 来对其进行访问，当然，既然存在随机访问就会出现越界的问题。而在 Rust 中，一旦越界的后果是极其严重的，可以导致 Rust 当前线程 panic 。因此，除非你确定自己在干什么或者在 `for` 循环中，不然我们不推荐通过下标访问。

以下是例子：

```rust
let a = vec![1, 2, 3];
assert_eq!(a[1usize], 2);
```

那么， Rust 中有没有安全的下标访问机制呢？答案是当然有：—— `.get(n: usize)` （`.get_mut(n: usize)`） 函数。
对于一个数组，这个函数返回一个`Option<&T>` (`Option<&mut T>`)，当为 `None` 的时候，即下标越界，其他情况下，我们能安全的获得一个 `Vec` 里面元素的引用。

```rust
let v = vec![1, 2, 3];
assert_eq!(v.get(1), Some(&2));
assert_eq!(v.get(3), None);
```

### 迭代器

对于一个可变数组， Rust 提供了一种简单的遍历形式——for循环。
我们可以获得一个数组的引用、可变引用、所有权。

```rust
let v = vec![1, 2, 3];
for i in &v { .. } // 获得引用
for i in &mut v { .. } // 获得可变引用
for i in v { .. } // 获得所有权，注意此时Vec的属主将会被转移！！
```

但是，这么写很容易出现多层for循环嵌套，因此， `Vec` 提供了一个 `into_iter()` 方法，能显式地将自己转换成一个迭代器。然而迭代器怎么用呢？我们下一章将会详细说明。

### 作为栈 (stack) 容器

Rust 没有单独提供栈容器类型，而是将功能集成于 `Vec` 中。我们可以使用 `push` 入栈并使用 `pop` 出栈。

```rust
let mut stack = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
    // Prints 3, 2, 1
    println!("{}", top);
}
```

### `push` 的效率研究

前面说到， `Vec` 有两个平摊 `O(1)` 的方法，分别是 `pop` 和 `push` ，它们分别代表着将数据从尾部弹出或者装入。理论上来说，因为 `Vec` 是支持随机访问的，因此 `push` 效率应该是一致的。但是实际上，因为 `Vec` 的内部存在着内存拷贝和销毁，因此，如果你想要将一个数组，从零个元素开始，一个一个的填充直到最后生成一个非常巨大的数组的话，预先为其分配内存是一个非常好的办法。

这其中，有个关键的方法是 `reserve()` 。

如下代码：

```rust
use std::time;

fn push_1m(v: &mut Vec<usize>, total: usize) {
    let e = time::SystemTime::now();
    for i in 1..total {
        v.push(i);
    }
    let ed = time::SystemTime::now();
    println!("time spend: {:?}", ed.duration_since(e).unwrap());
}

fn main() {
    let mut v: Vec<usize> = vec![];
    push_1m(&mut v, 5_000_000);
    let mut v: Vec<usize> = vec![];
    v.reserve(5_000_000);
    push_1m(&mut v, 5_000_000);
}
```

在笔者自己的笔记本上，编译好了 debug 的版本，上面的代码跑出了：

```plain
➜  debug git:(master) ✗ time ./demo
time spend: Duration { secs: 0, nanos: 368875346 }
time spend: Duration { secs: 0, nanos: 259878787 }
./demo  0.62s user 0.01s system 99% cpu 0.632 total

```

好像并没有太大差异？然而切换到 release 版本的时候:

```plain
➜  release git:(master) ✗ time ./demo
time spend: Duration { secs: 0, nanos: 53389934 }
time spend: Duration { secs: 0, nanos: 24979520 }
./demo  0.06s user 0.02s system 97% cpu 0.082 total
```

注意消耗的时间的位数。可见，在去除掉 debug 版本的调试信息之后，是否预分配内存消耗时间降低了一倍！

这样的成绩，可见，预先分配内存确实有助于提升效率。

有人可能会问了，你这样纠结这点时间，最后不也是节省在纳秒级别的么，有意义么？当然有意义。

第一，纳秒也是时间，这还是因为这个测试的 `Vec` 只是最简单的内存结构。一旦涉及到大对象的拷贝，所花费的时间可就不一定这么少了。
第二，频繁的申请和删除堆空间，其内存一旦达到瓶颈的时候你的程序将会异常危险。

更多 `Vec` 的操作，请参照标准库的 [API](https://doc.rust-lang.org/std/vec/struct.Vec.html) 。

## 容器、元素的生命周期

### 元素的引用与对容器的借用

前面我们提到，通过 `get()` 与 `[index]` 可以获得容器内对象的引用，而这时容器与元素的借用状态又是如何的呢？我们可以从一段看上去简单的代码入手。

```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = v.get(0).unwrap();

v.push(6);//will complain here
```

编译这段代码,会产生如下报错

```plain
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 --> src\main.rs:6:5
  |
4 |     let first = v.get(0).unwrap();
  |                 - immutable borrow occurs here
5 |
6 |     v.push(6);
  |     ^ mutable borrow occurs here
7 | }
  | - immutable borrow ends here

error: aborting due to previous error
```

可以看到，在获取对第一个元素的引用的时候，产生了对 `v` 的借用，而 `push()` 又要求了一个可变借用，导致了编译错误。我们获取了一个引用后，借用检查会确保这个引用和与我们对 `Vec` 中任何其他引用保持有效。例如，上例中如果 `push()` 导致了 `Vec` 重新分配内存，则所有的引用都将失效（在 C++ 中，容器操作导致迭代器与引用失效常常是 bug 的重要来源）。

同样的，我们也不可能获取一个元素的两份可变引用：

```rust
let mut v = vec![1, 2, 3, 4, 5];

let first = v.get_mut(0).unwrap();

let first_again = &mut v[0];//will complain here
```

要安全的获取多个不同元素的可变引用，我们可以利用数组的迭代器：

```rust
let mut iter = v.iter_mut();

let first = iter.next().unwrap();
let second = iter.next().unwrap();
let forth = iter.nth(1).unwrap();

*first = 10;
*forth = 40;
```

### `Vec` 容器的销毁

当我们销毁一个 `Vec` 的时候，由 `Vec` 持有所有权的所有元素也将被释放。

```rust
{
    let v = vec![1, 2, 3, 4];

    // do stuff with v

} // <- v goes out of scope and is freed here
```

当 `v` 离开作用域，它里面的所有整数也将被释放。对于持有其他类型的 `Vec` 也是同理。要注意的是目前 `Vec` 并不保证元素释放的顺序。
