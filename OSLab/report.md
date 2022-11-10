# 一些零时记录（未整理）



## 平台与目标三元组

编译器在编译、链接得到可执行文件时需要知道，程序要在哪个 **平台** (Platform) 上运行， **目标三元组** (Target Triplet) 描述了目标平台的 CPU 指令集、操作系统类型和标准运行时库。

我们研究一下现在 `Hello, world!` 程序的目标三元组是什么：

```shell
$ rustc --version --verbose
   rustc 1.65.0 (897e37553 2022-11-02)
   binary: rustc
   commit-hash: 897e37553bba8b42751c67658967889d11ecd120
   commit-date: 2022-11-02
   host: aarch64-apple-darwin
   release: 1.65.0
   LLVM version: 15.0.0
```

其中 host 一项表明默认目标平台是 `aarch64-apple-darwin`， CPU 架构是 aarch64，CPU 厂商是 apple，操作系统是darwin。

接下来，我们希望把 `Hello, world!` 移植到 RISC-V 目标平台 `riscv64gc-unknown-none-elf` 上运行。

## 修改目标平台

将程序的目标平台换成 `riscv64gc-unknown-none-elf`，试试看会发生什么：

```
$ cargo run --target riscv64gc-unknown-none-elf
   Compiling os v0.1.0 (/home/shinbokuow/workspace/v3/rCore-Tutorial-v3/os)
error[E0463]: can't find crate for `std`
  |
  = note: the `riscv64gc-unknown-none-elf` target may not be installed
```

报错的原因是目标平台上确实没有 Rust 标准库 std，也不存在任何受 OS 支持的系统调用。 这样的平台被我们称为 **裸机平台** (bare-metal)。

幸运的是，除了 std 之外，Rust 还有一个不需要任何操作系统支持的核心库 core， 它包含了 Rust 语言相当一部分核心机制，可以满足本门课程的需求。 有很多第三方库也不依赖标准库 std，而仅仅依赖核心库 core。

为了以裸机平台为目标编译程序，我们要将对标准库 std 的引用换成核心库 core。



编译器运行的平台（x86_64）与可执行文件运行的目标平台不同的情况，称为 **交叉编译** (Cross Compile)。



## 在提”供语义项 panic_handler“之后遇到报错

error: language item required, but not found: `eh_personality`

发现是由于

然后在 `os` 目录下新建 `.cargo` 目录，并在这个目录下创建 `config` 文件，输入如下内容：

```shell
# os/.cargo/config
[build]
target = "riscv64gc-unknown-none-elf"
```

这里错误的将.cargo文件夹创建在src目录下了



QEMU有两种运行模式：

`User mode` 模式，即用户态模拟，如 `qemu-riscv64` 程序， 能够模拟不同处理器的用户态指令的执行，并可以直接解析ELF可执行文件， 加载运行那些为不同处理器编译的用户级Linux应用程序。

`System mode` 模式，即系统态模式，如 `qemu-system-riscv64` 程序， 能够模拟一个完整的基于不同CPU的硬件系统，包括处理器、内存及其他外部设备，支持运行完整的操作系统。

**在mac上使用brew install qemu安装，没有用户态模拟**



### Qemu 启动流程

`virt` 平台上，物理内存的起始物理地址为 `0x80000000` ，物理内存的默认大小为 128MiB ，它可以通过 `-m` 选项进行配置。在本书中，我们只会用到最低的 8MiB 物理内存，对应的物理地址区间为 `[0x80000000,0x80800000)` 。如果使用上面给出的命令启动 Qemu ，那么在 Qemu 开始执行任何指令之前，首先两个文件将被加载到 Qemu 的物理内存中：即作为 bootloader 的 `rustsbi-qemu.bin` 被加载到物理内存以物理地址 `0x80000000` 开头的区域上，同时内核镜像 `os.bin` 被加载到以物理地址 `0x80200000` 开头的区域上。

为什么加载到这两个位置呢？这与 Qemu  模拟计算机加电启动后的运行流程有关。一般来说，计算机加电之后的启动流程可以分成若干个阶段，每个阶段均由一层软件负责，每一层软件的功能是进行它应当承担的初始化工作，并在此之后跳转到下一层软件的入口地址，也就是将计算机的控制权移交给了下一层软件。Qemu 模拟的启动流程则可以分为三个阶段：第一个阶段由固化在 Qemu 内的一小段汇编程序负责；第二个阶段由 bootloader  负责；第三个阶段则由内核镜像负责。

- 第一阶段：将必要的文件载入到 Qemu 物理内存之后，Qemu CPU 的程序计数器（PC, Program Counter）会被初始化为 `0x1000` ，因此 Qemu 实际执行的第一条指令位于物理地址 `0x1000` ，接下来它将执行寥寥数条指令并跳转到物理地址 `0x80000000` 对应的指令处并进入第二阶段。从后面的调试过程可以看出，该地址 `0x80000000` 被固化在 Qemu 中，作为 Qemu 的使用者，我们在不触及 Qemu 源代码的情况下无法进行更改。
- 第二阶段：由于 Qemu 的第一阶段固定跳转到 `0x80000000` ，我们需要将负责第二阶段的 bootloader `rustsbi-qemu.bin` 放在以物理地址 `0x80000000` 开头的物理内存中，这样就能保证 `0x80000000` 处正好保存 bootloader 的第一条指令。在这一阶段，bootloader 负责对计算机进行一些初始化工作，并跳转到下一阶段软件的入口，在 Qemu 上即可实现将计算机控制权移交给我们的内核镜像 `os.bin` 。这里需要注意的是，对于不同的 bootloader  而言，下一阶段软件的入口不一定相同，而且获取这一信息的方式和时间点也不同：入口地址可能是一个预先约定好的固定的值，也有可能是在  bootloader 运行期间才动态获取到的值。我们选用的 RustSBI 则是将下一阶段的入口地址预先约定为固定的 `0x80200000` ，在 RustSBI 的初始化工作完成之后，它会跳转到该地址并将计算机控制权移交给下一阶段的软件——也即我们的内核镜像。
- 第三阶段：为了正确地和上一阶段的 RustSBI 对接，我们需要保证内核的第一条指令位于物理地址 `0x80200000` 处。为此，我们需要将内核镜像预先加载到 Qemu 物理内存以地址 `0x80200000` 开头的区域上。一旦 CPU 开始执行内核的第一条指令，证明计算机的控制权已经被移交给我们的内核，也就达到了本节的目标。

如何得到一个能够在 Qemu 上成功运行的内核镜像呢？首先我们需要通过链接脚本调整内核可执行文件的内存布局，使得内核被执行的第一条指令位于地址 `0x80200000` 处，同时代码段所在的地址应低于其他段。这是因为 Qemu 物理内存中低于 `0x80200000` 的区域并未分配给内核，而是主要由 RustSBI  使用。其次，我们需要将内核可执行文件中的元数据丢掉得到内核镜像，此内核镜像仅包含实际会用到的代码和数据。这则是因为 Qemu  的加载功能过于简单直接，它直接将输入的文件逐字节拷贝到物理内存中，因此也可以说这一步是我们在帮助 Qemu 手动将可执行文件加载到物理内存中。



## 手动加载内核可执行文件

上面得到的内核可执行文件完全符合我们对于内存布局的要求，但是我们不能将其直接提交给 Qemu ，因为它除了实际会被用到的代码和数据段之外还有一些多余的元数据，这些元数据无法被 Qemu 在加载文件时利用，且会使代码和数据段被加载到错误的位置。

文件时利用，且会使代码和数据段被加载到错误的位置。如下图所示：

![../_images/load-into-qemu.png](http://rcore-os.cn/rCore-Tutorial-Book-v3/_images/load-into-qemu.png)

图中，红色的区域表示内核可执行文件中的元数据，深蓝色的区域表示各个段（包括代码段和数据段），而浅蓝色区域则表示内核被执行的第一条指令，它位于深蓝色区域的开头。图示的上半部分中，我们直接将内核可执行文件 `os` 加载到 Qemu 内存的 `0x80200000` 处，由于内核可执行文件的开头是一段元数据，这会导致 Qemu 内存 `0x80200000` 处无法找到内核第一条指令，也就意味着 RustSBI 无法正常将计算机控制权转交给内核。相反，图示的下半部分中，将元数据丢弃得到的内核镜像 `os.bin` 被加载到 Qemu 之后，则可以在 `0x80200000` 处正确找到内核第一条指令。



## 函数调用与栈

函数调用上下文由调用者和被调用者分别保存，其具体过程分别如下：

- 调用函数：首先保存不希望在函数调用过程中发生变化的 **调用者保存寄存器** ，然后通过 jal/jalr 指令调用子函数，返回之后恢复这些寄存器。
- 被调用函数：在被调用函数的起始，先保存函数执行过程中被用到的 **被调用者保存寄存器** ，然后执行函数，最后在函数退出之前恢复这些寄存器。

## 使用 RustSBI 提供的服务

之前我们对 RustSBI  的了解仅限于它会在计算机启动时进行它所负责的环境初始化工作，并将计算机控制权移交给内核。但实际上作为内核的执行环境，它还有另一项职责：即在内核运行时响应内核的请求为内核提供服务。当内核发出请求时，计算机会转由 RustSBI  控制来响应内核的请求，待请求处理完毕后，计算机控制权会被交还给内核。从内存布局的角度来思考，每一层执行环境（或称软件栈）都对应到内存中的一段代码和数据，这里的控制权转移指的是 CPU 从执行一层软件的代码到执行另一层软件的代码的过程。这个过程和函数调用比较像，但是内核无法通过函数调用来请求 RustSBI  提供的服务，这是因为内核并没有和 RustSBI 链接到一起，我们仅仅使用 RustSBI 构建后的可执行文件，因此内核对于 RustSBI  的符号一无所知。事实上，内核需要通过另一种复杂的方式来“调用” RustSBI 的服务：

```shell
// os/src/main.rs
mod sbi;

// os/src/sbi.rs
use core::arch::asm;
#[inline(always)]
fn sbi_call(which: usize, arg0: usize, arg1: usize, arg2: usize) -> usize {
    let mut ret;
    unsafe {
        asm!(
            "ecall",
            inlateout("x10") arg0 => ret,
            in("x11") arg1,
            in("x12") arg2,
            in("x17") which,
        );
    }
    ret
}
```

我们将内核与 RustSBI 通信的相关功能实现在子模块 `sbi` 中，因此我们需要在 `main.rs` 中加入 `mod sbi` 将该子模块加入我们的项目。在 `os/src/sbi.rs` 中，我们首先关注 `sbi_call` 的函数签名， `which` 表示请求 RustSBI 的服务的类型（RustSBI 可以提供多种不同类型的服务）， `arg0` ~ `arg2` 表示传递给 RustSBI 的 3 个参数，而 RustSBI 在将请求处理完毕后，会给内核一个返回值，这个返回值也会被 `sbi_call` 函数返回。尽管我们还不太理解函数 `sbi_call` 的具体实现，但目前我们已经知道如何使用它了：当需要使用 RustSBI 服务的时候调用它就行了。

在 `sbi.rs` 中我们定义 RustSBI 支持的服务类型常量，它们并未被完全用到：

```shell
// os/src/sbi.rs
#![allow(unused)] // 此行请放在该文件最开头
const SBI_SET_TIMER: usize = 0;
const SBI_CONSOLE_PUTCHAR: usize = 1;
const SBI_CONSOLE_GETCHAR: usize = 2;
const SBI_CLEAR_IPI: usize = 3;
const SBI_SEND_IPI: usize = 4;
const SBI_REMOTE_FENCE_I: usize = 5;
const SBI_REMOTE_SFENCE_VMA: usize = 6;
const SBI_REMOTE_SFENCE_VMA_ASID: usize = 7;
const SBI_SHUTDOWN: usize = 8;
```

如字面意思，服务 `SBI_CONSOLE_PUTCHAR` 可以用来在屏幕上输出一个字符。我们将这个功能封装成 `console_putchar` 函数：

```shell
// os/src/sbi.rs
pub fn console_putchar(c: usize) {
    sbi_call(SBI_CONSOLE_PUTCHAR, c, 0, 0);
}
```

注意我们并未使用 `sbi_call` 的返回值，因为它并不重要。如果读者有兴趣的话，可以试着在 `rust_main` 中调用 `console_putchar` 来在屏幕上输出 `OK` 。接着在 Qemu 上运行一下，我们便可看到由我们自己输出的第一条 log 了。

类似的，还可以将关机服务 `SBI_SHUTDOWN` 封装成 `shutdown` 函数：

```shell
// os/src/sbi.rs
pub fn shutdown() -> ! {
    sbi_call(SBI_SHUTDOWN, 0, 0, 0);
    panic!("It should shutdown!");
}
```

