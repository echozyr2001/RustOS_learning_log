# RustOS_learning_log

## 11月8日

**看easy rust的一些记录**

* `BTreeMap`是可以排序的`HashMap`。

* `.get()` 是从 `HashMap` 中获取一个值的比较安全的方法。直接使用键取值可能会遇到键不存在的情况，会导致程序崩溃。

* `HashMap`有一个非常有趣的方法，叫做`.entry()`可以在没有键的情况下，用如`.or_insert()`这类方法来插入值。

	* ```rust
    use std::collections::HashMap;
    
    fn main() {
        let book_collection = vec!["L'Allemagne Moderne", "Le Petit Prince", "Eye of the World", "Eye of the World"];
        let mut book_hashmap = HashMap::new();
        for book in book_collection {
            let return_value = book_hashmap.entry(book).or_insert(0); // return_value is a mutable reference. If nothing is there, it will be 0
            *return_value +=1; // Now return_value is at least 1. And if there was another book, it will go up by 1
        }
        for (book, number) in book_hashmap {
            println!("{}, {}", book, number);
        }
    }
    ```
  
* 使用`BinaryHeap<(u8, &str)>`的一个好方法是用于一个事情的集合。这里我们创建一个`BinaryHeap<(u8, &str)>`，其中`u8`是任务重要性的数字。`&str`是对要做的事情的描述。

* `VecDeque`就是一个`Vec`，既能从前面弹出item，又能从后面弹出item。

**rustlings进度**

Progress: [###########################################>----------------] 68/94

hashmaps3有问题，不是很会。

* 目标（由于明天考试，目标少设置一点）
    * 弄懂hashmaps3。
    * 继续RISC-V的学习。

## 11月7日

**看easy rust的一些记录**

* `Vec`可以使用`Vec::with_capacity(number)`定义容量，防止在向其中添加元素时发生重分配（如果你超过了容量，它将使其容量翻倍，并将元素复制到新的空间）。

* `.into()`可以把`&str`变成`String`，同样可以把`[]`变成`Vec`

    * ```rust
        let my_vec: Vec<u8> = [1, 2, 3].into();
        let my_vec2: Vec<_> = [9, 0, 10].into(); 
        // Vec<_> means "choose the Vec type for me"
        // Rust will choose Vec<i32>
        ```

* 在`match`中，用`_`表示其他任何东西。

* 在结构体中，如果字段名和变量名是一样的，就不用写两次。

* 带有`#[]`的信息被称为**属性**。

* 使用`.`运算符时，不需要担心`*`。

* 使用范型时，要提前告诉`T`的属性。

* `Option`和`Result是两个枚举。

* 可以用 `.unwrap()` 在一个Option中获取值，但要小心 `.unwrap()`。这就像拆礼物一样:也许里面有好东西，也许里面有一条愤怒的蛇。只有在你确定的情况下，你才会想要`.unwrap()`。如果你拆开一个`None`的值，程序就会崩溃。

* `.is_some()` 方法来判断是否是 `Some`，`.is_none()`方法判断是否是`None`。

* **所以，`Option`是如果你在想:"也许会有，也许不会有。"也许会有一些东西，也许不会有。" 但`Result`是如果你在想: "也许会失败"。**

**rustlings进度**

Progress: [###########################>--------------------------------] 43/94

#### 总结

由于时间原因，easy rust只看到了33。rustlings目标完成。

看了一点RISC-V手册，有点看不懂。

* 目标
    * 进入RISC-V的学习。
    * rust做为复习。

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

## 11月5日

在准备考试，今天就晚上看了以下课程直播。解决了之前vecs、move_semantics的疑问。

决定从今天开始每天设置一个目标，在第二天反馈目标完成情况。

* 目标
    * 看easy rust前20章，与配套视频。
    * 重做rustlings，能做多少做多少。

## 11月4日

完善rustlings，有些题目不会，参考了别人的解决方法。

完成structs、enums、strings、modules、hashmaps、quiz2、options、errors、generics、traits、quiz3、tests、lifetimes、iterators、box、arc、rc、cow、threads、macros、clippy、using_as、from_into、from_str、try_from_into、as_ref_mut。

* 其中主要对strings、options、box、arc、rc、cow、threads、traits有疑问。

## 11月3日

主要在做rustlings。

完成intro、variables、functions、if、quiz1、primitive_types、vecs、move_semantics。

* 其中主要对vecs、move_semantics有疑问。
