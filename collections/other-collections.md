# 其他集合类型

Rust 除了最常用的 `Vec` 与 `HashMap` 两种集合类型之外，还有其他一些很好用的集合类型。虽然不是那么常用，但在实现特定的工作时却可以令你事半功倍。

## 集合类型总览

Rust 的集合类型大概可以分成以下四种类型：

* 序列: `Vec`, `VecDeque`, `LinkedList`
* 映射: `HashMap`, `BTreeMap`
* 集: `HashSet`, `BTreeSet`
* 杂项: `BinaryHeap`

大部分时候，我们优先使用 `Vec` 与 `HashMap` ，这两个集合类型覆盖了大部分泛型数据的存储与处理的工作。即使看起来别的稍微合适一点，使用这两个开始或许会是更好的选择。详细的信息可以参考 [std::collections](https://doc.rust-lang.org/std/collections/index.html) 的文档。本节部分内容也翻译于此。

## `Vec` 的使用场景

* 你想把元素集合起来处理或者交给其他地方，并不关心元素的实际属性
* 你想有一个特定顺序的集合，并且只会将元素添加到末尾
* 你想要一个栈结构
* 你想要一个可变数组
* 你想要一个堆数组

## `VecDeque` 的使用场景

* 你想要一个像 `Vec` 一样但支持在序列两端插入的结构
* 你想要一个队列
* 你想要一个双端队列

`VecDeque` 的底层使用了环形缓冲区，相对于 `Vec` 最大的不同就是支持在序列两端插入和弹出。

```rust
//创建VecDeque
let mut buf = VecDeque::new();

//可以在序列两端插入
buf.push_back(3);
buf.push_back(4);
buf.push_back(5);
buf.push_front(2);
buf.push_front(1);

//可以在序列两端弹出
if let Some(elem) = buf.pop_front() {
    assert_eq!(elem, 1);
}
```

## `LinkedList` 的使用场景

* 你想要一个不定长的 `Vec` 或 `VecDeque` 并且无法忍受插入时重分配带来的代价
* 你想要高效的拆分与连接队列
* 你确信你真的需要一个双向链表

大部分时候请选择 `Vec` 或 `VecDeque` ，因为基于数组的容器的效率更高、更省内存，并且可以更好的利用 CPU 的 cache 。

```rust
use std::collections::LinkedList;

//创建链表并添加元素
let mut list1 = LinkedList::new();
list1.push_back('a');

let mut list2 = LinkedList::new();
list2.push_back('b');
list2.push_back('c');

//链表的append是O(1)的
list1.append(&mut list2);

let mut iter = list1.iter();
assert_eq!(iter.next(), Some(&'a'));
assert_eq!(iter.next(), Some(&'b'));
assert_eq!(iter.next(), Some(&'c'));
assert!(iter.next().is_none());

//append后参数将会被清空
assert!(list2.is_empty());
```

## `HashMap` 的使用场景

* 你想将键与值关联
* 你想要一个缓存
* 你想要一个表，不需要其他功能

## `BTreeMap` 的使用场景

* 你对最大或者最小的键值对感兴趣
* 你想要查询最大或最小的比某个值小或大的键
* 你需要得到排序后的所有项
* 你想要一个按键排序的表

顾名思义， `BTreeMap` 是基于B树的存储结构。

```rust
use std::collections::BTreeMap;
use std::collections::Bound::Included;

let mut map = BTreeMap::new();

//插入键值对
map.insert(5, "b");
map.insert(3, "a");
map.insert(8, "c");

//使用range指定key的范围
for (&key, &value) in map.range((Included(&4), Included(&8))) {
    println!("{}: {}", key, value);
}
assert_eq!(Some((&5, &"b")), map.range(4..).next());

//使用keys返回排序的键列表
let keys: Vec<_> = map.keys().cloned().collect();
assert_eq!(keys, [3, 5, 8]);
```

## `Set` 变体的使用场景

* 你想记住你见到的键
* 没有有意义的值可以关联
* 你只需要一个集合

## `BinaryHeap` 的使用场景

* 你需要存储一系列元素但只需要处理最大或最小的元素
* 你想要一个优先队列

```rust
use std::collections::BinaryHeap;

//类型推导让我们可以省略具体的类型
//如本例中的BinaryHeap<i32>
let mut heap = BinaryHeap::new();

//我们可以使用peek来查看当前堆中最大的元素
//现在并没有元素所以返回None
assert_eq!(heap.peek(), None);

// 添加元素
heap.push(1);
heap.push(5);
heap.push(2);

assert_eq!(heap.peek(), Some(&5));

//当前堆的长度
assert_eq!(heap.len(), 3);

//可以使用for循环迭代堆中的元素，但不按照顺序
for x in &heap {
    println!("{}", x);
}

//使用pop弹出元素，按照从大到小的顺序
assert_eq!(heap.pop(), Some(5));
assert_eq!(heap.pop(), Some(2));
assert_eq!(heap.pop(), Some(1));
assert_eq!(heap.pop(), None);

//清除堆
heap.clear();

assert!(heap.is_empty())
```

## 各操作的时间复杂度

|            | get(i)         | insert(i)       | remove(i)      | append       | split_off(i)   |
|------------|----------------|-----------------|----------------|--------------|----------------|
| Vec        | O(1)           | O(n-i)*         | O(n-i)         | O(m)*        | O(n-i)         |
| VecDeque   | O(1)           | O(min(i, n-i))* | O(min(i, n-i)) | O(m)*        | O(min(i, n-i)) |
| LinkedList | O(min(i, n-i)) | O(min(i, n-i))  | O(min(i, n-i)) | O(1)         | O(min(i, n-i)) |

集合类型的复杂度与他们对应的Map相同。

|          | get      | insert   | remove   | predecessor | append |
|----------|----------|----------|----------|-------------|--------|
| HashMap  | O(1)~    | O(1)~*   | O(1)~    | N/A         | N/A    |
| BTreeMap | O(log n) | O(log n) | O(log n) | O(log n)    | O(n+m) |