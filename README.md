# RustOS_learning_log

## 11月27日

推进lab2，复习了一下地址的映射，还是不知道该怎么做……

## 11月26日

最近太忙了，无进展……

## 11月24日

今天在解决昨天遗留的问题，没有进展……

很长一段时间的目标应该是一样的，就不重复写了。

## 11月23日

昨天在忙老师那边的任务，没有更新日志。

今天继续阅读rCore第四章，开始尝试lab2。重写了sys_get_time()，个人感觉没问题但是测试不通过，暂时没有找到解决方案。明天继续。

* 目标
    * rCore第四章，推进lab2。

## 11月21日

今天通知要给老师当助教，讲系统调用的相关实验。这两天应该都是准备实验内容，只能抽空做RustOS。

rCore第章看了一点，结合老师讲的内存分页，有一点理解了。

* 目标
    * rCore第四章，推进lab2。

## 11月20日

今天主要把之间没看的关于内存地址分页的内容看完了。了解到了多级页表的概念和底层实现。

还了解了RISC-V中satp寄存器的作用。

* 目标
    * rCore第四章，推进lab2。

## 11月19日

今天完善了一下lab1的代码，然后完成了lab1的报告。

* 目标：
    * 看rCore第四章，推进lab2。

## 11月18日补充

刚刚突然想到一种可能，每次检测结果都是我的运行时间过长，而总运行时长的初始化我设置的是0，应该是通过get_time()获取时间才对。

修改之后果然所有测试都通过了。

那么明天的任务就是做整理了。

## 11月18日

今天继续进行rCore实验1。经过一下午的努力，还有两个测试点不通过。

```shell
[PASS] found <Hello, world from user mode program!>
[PASS] found <Test power_3 OK18591!>
[PASS] found <Test power_5 OK18591!>
[PASS] found <Test power_7 OK18591!>
[PASS] found <get_time OK18591! (\d+)>
[PASS] found <Test sleep OK18591!>
[PASS] found <current time_msec = (\d+)>
[PASS] found <time_msec = (\d+) after sleeping (\d+) ticks, delta = (\d+)ms!>
[PASS] found <Test sleep1 passed18591!>
[PASS] found <Test write A OK18591!>
[PASS] found <Test write B OK18591!>
[PASS] found <Test write C OK18591!>
[FAIL] not found <string from task info test>
[FAIL] not found <Test task info OK18591!>
[PASS] not found <FAIL: T.T>
```

从这句来看应该是时间的问题

```shell
Panicked at src/bin/ch3_taskinfo.rs:26, assertion failed: t2 - t1 <= info.time + 1
```

明天再继续解决这个问题。

* 目标
    * 解决今天遇到的问题，完成实验1。
    * 整理之前的内容。

## 11月17日

* 一些记录
    * RISC-V指令集和典型的CISC指令集的代表8051指令集相比：引入了指令长度编码；指令集规模较小，指令格式规整；每条指令实现单个功能；内存访问只能通过LOAD/STORE。
    * RISC-V的官方标准分为用户指令集（User-Level Instruction Set Architecture）与特权架构（Privileged  Architecture）。其中，拥护指令集可以进一步分为基础整数指令集（Based Integer Instruction  Set）和扩展指令集(Extension)。
    * RISC-V将用户指令集和特权架构分开的目的，是希望不同特权架构的处理器可以在ABI互相兼容。
* 目标
    * rCore第三章。

## 11月16日

白天主要在上课和做作业，没有进行Rust的学习。

晚上跟着教学直播学习risc-v。

* 目标（最近学校里的事情有点多，这边的进度稍微放缓）
    * 看risc-v手册第10章。

## 11月15日

又看了一天的rCore第三章，稍微有了一点思路，但是过程忘记整理了。

* 目标
    * risc-v学习。
    * 继续rCore第三章。

## 11月14日

鸽了，什么也没做……

* 目标：
    * 继续rust学习，从项目入手。

## 11月13日

参考了这片博客**[Rustle](https://medium.com/pragmatic-programmers/rustle-5c15d1c153a1)**复现了其中的`rustle`小程序。

涉及到的不熟悉的知识点总结如下：

### `.unwrap()`方法

#### [unwrap in Rust](https://www.delftstack.com/howto/rust/rust-unwrap/#unwrap-in-rust)

In Rust, to `unwrap` something means to issue the following command: “Give me the result of  the computation, and if there was an error, panic and stop the program.” Because unwrapping is such a straightforward process, it would be  beneficial for us to demonstrate the code for it.

### `.nth()`方法

#### [fn nth(&mut self, n: usize) -> Option\<Self::Item>](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.nth)

Returns the `n`th element of the iterator.

Like most indexing operations, the count starts from zero, so `nth(0)` returns the first value, `nth(1)` the second, and so on.

Note that all preceding elements, as well as the returned element, will be consumed from the iterator. That means that the preceding elements will be discarded, and also that calling `nth(0)` multiple times on the same iterator will return different elements.

`nth()` will return [`None`](https://doc.rust-lang.org/std/option/enum.Option.html#variant.None) if `n` is greater than or equal to the length of the iterator.

### `.skip()`方法

#### [fn skip(self, n: usize) -> Skip\<Self>](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.skip)

Creates an iterator that skips the first `n` elements.

`skip(n)` skips elements until `n` elements are skipped or the end of the iterator is reached (whichever happens first). After that, all the remaining elements are yielded. In particular, if the original iterator is too short, then the returned iterator is empty.

Rather than overriding this method directly, instead override the `nth` method.

### 第三方库`colored`和`bracket-random`

第三方库相关API可以在这个网站查询到：https://docs.rs/。

* 目标：
    * 继续rust学习，从项目入手。

## 11月12日

用了半天，lab1根本没有思路……并且rust的使用上也有一点问题。决定回去看教学视频，跟着老师的进度走。

* 目标：
    * 重回rust学习。

## 11月11日

今天由于时间关系只看完了rCore手册的第二章，实现了对应代码。但是大部分的代码还是copy的。

这部分主要理解了特权机制：应用程序不能执行某些可能破坏计算机系统的指令。

* 目标
    * 完成手册第三章的实验。
    * 完成训练营lab1。

## 11月10日

**OS实验遇到的一些问题**

**1. 第一章在提”供语义项 panic_handler“之后遇到报错。**

```shell
error: language item required, but not found: `eh_personality`
```

发现是由于在 `os` 目录下新建 `.cargo` 目录的时候，错误的将 `.cargo` 目录创建在`src`目录下了。

**2. 没有找到命令`qemu-riscv64`。**

由于在MacOS上使用`brew install qemu`安装qemu只有`qemu-system-riscv64`没有`qemu-riscv64`。目前没找到解决方法，但是也能正常进行实验。

**3. 在实现rCore第一章过程中，`lang_item.rs`中代码报错找不到`prinln!`宏。**

由于没有在`mod console`前面加上`#[macro_use]`导致`console.rs`中的宏没有被识别。

**4. 报错`error[E0554]: #![feature] may not be used on the stable release channel`。**

由于使用的`rustc`为stable版本，切换为`nighty`就可以了

**5. 执行下列命令，生成的os.bin大小为0。**

```shell
rust-objcopy --strip-all target/riscv64gc-unknown-none-elf/release/os -O binary target/riscv64gc-unknown-none-elf/release/os.bin
```
发现是由于`entry.asm`修改之后没有保存。

#### 总结

训练营的OSLab代码前两章没有实验，于是clone了rCore的源码，参考手册完成了第一章的实验练习。更深入理解了操作系统的引导和执行。

成功输出如下内容。

```shell
[rustsbi] RustSBI version 0.3.0-alpha.4, adapting to RISC-V SBI v1.0.0
.______       __    __      _______.___________.  _______..______   __
|   _  \     |  |  |  |    /       |           | /       ||   _  \ |  |
|  |_)  |    |  |  |  |   |   (----`---|  |----`|   (----`|  |_)  ||  |
|      /     |  |  |  |    \   \       |  |      \   \    |   _  < |  |
|  |\  \----.|  `--'  |.----)   |      |  |  .----)   |   |  |_)  ||  |
| _| `._____| \______/ |_______/       |__|  |_______/    |______/ |__|
[rustsbi] Implementation     : RustSBI-QEMU Version 0.2.0-alpha.2
[rustsbi] Platform Name      : riscv-virtio,qemu
[rustsbi] Platform SMP       : 1
[rustsbi] Platform Memory    : 0x80000000..0x88000000
[rustsbi] Boot HART          : 0
[rustsbi] Device Tree Region : 0x87e00000..0x87e00f85
[rustsbi] Firmware Address   : 0x80000000
[rustsbi] Supervisor Address : 0x80200000
[rustsbi] pmp01: 0x00000000..0x80000000 (-wr)
[rustsbi] pmp02: 0x80000000..0x80200000 (---)
[rustsbi] pmp03: 0x80200000..0x88000000 (xwr)
[rustsbi] pmp04: 0x88000000..0x00000000 (-wr)
Hello, world!
Panicked at src/main.rs:17 Shutdown machine!
```

* 目标
    * 完成手册第二章的实验。

## 11月9日

**看easy rust的一些记录**

* 35特性，38闭包有点难懂，准备跳过这一部分，在实际遇到的时候再回过头来看。

* 从42开始，就是进阶内容来，跳过这部分，在实际遇到时再回过头来看。

* 迭代器的类型

    * `.iter()` 引用的迭代器
    * `.iter_mut()` 可变引用的迭代器
    * `.into_iter()` 值的迭代器(不是引用)

* `.map()`方法。这个方法可以让你对每一个元素做一些事情，然后把它传递下去。

* `.for_each()`的方法。这个方法只是让你对每一个元素做一些事情。

* `||`表示闭包。

* 当闭包变得更复杂时，你可以添加一个代码块。那就可以随心所欲的长。

    ```rust
    fn main() {
        let my_closure = || {
            let number = 7;
            let other_number = 10;
            println!("The two numbers are {} and {}.", number, other_number);
              // This closure can be as long as we want, just like a function.
        };
        my_closure();
    }
    ```

**rustlings进度**

Progress: [##################################################>---------] 79/94

#### 总结

easy rust看到了42章，感觉后面很多进阶内容暂时不看easy rust了。

OS的实验完成了lab0的环境搭建。

rustlings做到的不参考基本无法完成的位置。

* 目标
    * 看之前夏季OS课程的资源和文档，争取把前两章看完。
    * 同步推进Rust基础的学习。

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

#### 总结

easy tust看到了35。

RISC-V手册主要了解了一点汇编指令。

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
