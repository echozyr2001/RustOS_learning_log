# RustOS_learning_log

___

## 11月3日

主要在做rustlings。

完成intro、variables、functions、if、quiz1、primitive_types、vecs、move_semantics。

* 其中主要对vecs、move_semantics有疑问。

## 11月4日

完善rustlings，有些题目不会，参考了别人的解决方法。

完成structs、enums、strings、modules、hashmaps、quiz2、options、errors、generics、traits、quiz3、tests、lifetimes、iterators、box、arc、rc、cow、threads、macros、clippy、using_as、from_into、from_str、try_from_into、as_ref_mut。

* 其中主要对strings、options、box、arc、rc、cow、threads、traits有疑问。

## 11月5日

在准备考试，今天就晚上看了以下课程直播。解决了之前vecs、move_semantics的疑问。

决定从今天开始每天设置一个目标，在第二天反馈目标完成情况。

* 目标
    * 看easy rust前20章，与配套视频。
    * 重做rustlings，能做多少做多少。

## 11月6日

**看easy rust的一些记录**

* 很多类型都可以通过.into()来转换

* Rust中的一些类型非常简单。它们被称为**拷贝类型**。这些简单的类型都在栈中，编译器知道它们的大小。这意味着它们非常容易复制，所以当你把它发送到一个函数时，编译器总是会复制。它总是复制，因为它们是如此的小而简单，没有理由不复制。所以你不需要担心这些类型的所有权问题。这些简单的类型包括:**整数、浮点数、布尔值(`true`和`false`)和`char`**。

* 无值变量

```rust
fn main() {
    let my_number;
    {
        my_number = 100;
    }
    println!("{}", my_number);
}
```

`my_number`不是`mut`，我们在给它50之前并没有给它一个值，所以它的值一直没有改变。最后，`my_number`的真正代码只是`let my_number = 100;`。

* 切片是一种引用

**rustlings进度**

Progress: [##########################>---------------------------------] 41/94

#### 总结

完成昨天设置的目标。

* 目标
    * 继续看20章easy rust。
    * rustlings从41题开始有点困难了，需要配合the book，计划完成并掌握modules。
    * 开始准备下一阶段操作系统的实验。
