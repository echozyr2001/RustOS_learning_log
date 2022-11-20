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

编译器运行的平台（x86_64）与可执行文件运行的目标平台不同的情况，称为 **交叉编译** (Cross Compile)。

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

# 特权级机制

为了保护我们的批处理操作系统不受到出错应用程序的影响并全程稳定工作，单凭软件实现是很难做到的，而是需要 CPU 提供一种特权级隔离机制，使 CPU 在执行应用程序和操作系统内核的指令时处于不同的特权级。

## RISC-V 特权级架构

RISC-V 架构中一共定义了 4 种特权级：

| 级别 | 编码 | 名称                                |
| ---- | ---- | ----------------------------------- |
| 0    | 00   | 用户/应用模式 (U, User/Application) |
| 1    | 01   | 监督模式 (S, Supervisor)            |
| 2    | 10   | 虚拟监督模式 (H, Hypervisor)        |
| 3    | 11   | 机器模式 (M, Machine)               |

其中，级别的数值越大，特权级越高，掌控硬件的能力越强。从表中可以看出， M 模式处在最高的特权级，而 U 模式处于最低的特权级。在CPU硬件层面，除了M模式必须存在外，其它模式可以不存在。

之前我们给出过支持应用程序运行的一套 [执行环境栈](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/1app-ee-platform.html#app-software-stack) ，现在我们站在特权级架构的角度去重新看待它：

![../_images/PrivilegeStack.png](http://rcore-os.cn/rCore-Tutorial-Book-v3/_images/PrivilegeStack.png)

和之前一样，白色块表示一层执行环境，黑色块表示相邻两层执行环境之间的接口。这张图片给出了能够支持运行 Unix 这类复杂系统的软件栈。其中操作系统内核代码运行在 S 模式上；应用程序运行在 U 模式上。运行在 M 模式上的软件被称为 **监督模式执行环境** (SEE, Supervisor Execution Environment)，如在操作系统运行前负责加载操作系统的 Bootloader – RustSBI。站在运行在 S 模式上的软件视角来看，它的下面也需要一层执行环境支撑，因此被命名为 SEE，它需要在相比 S  模式更高的特权级下运行，一般情况下 SEE 在 M 模式上运行。



## RISC-V特权级切换

### 特权级切换的起因

我们知道，批处理操作系统被设计为运行在内核态特权级（RISC-V 的 S 模式），这是作为  SEE（Supervisor Execution Environment）的 RustSBI  所保证的。而应用程序被设计为运行在用户态特权级（RISC-V 的 U  模式），被操作系统为核心的执行环境监管起来。在本章中，这个应用程序的执行环境即是由“邓式鱼” 批处理操作系统提供的  AEE（Application Execution Environment)  。批处理操作系统为了建立好应用程序的执行环境，需要在执行应用程序之前进行一些初始化工作，并监控应用程序的执行，具体体现在：

- 当启动应用程序的时候，需要初始化应用程序的用户态上下文，并能切换到用户态执行应用程序；
- 当应用程序发起系统调用（即发出 Trap）之后，需要到批处理操作系统中进行处理；
- 当应用程序执行出错的时候，需要到批处理操作系统中杀死该应用并加载运行下一个应用；
- 当应用程序执行结束的时候，需要到批处理操作系统中加载运行下一个应用（实际上也是通过系统调用 `sys_exit` 来实现的）。

这些处理都涉及到特权级切换，因此需要应用程序、操作系统和硬件一起协同，完成特权级切换机制。



## 用户栈与内核栈

在 Trap 触发的一瞬间， CPU 就会切换到 S 特权级并跳转到 `stvec` 所指示的位置。但是在正式进入 S 特权级的 Trap 处理之前，上面 提到过我们必须保存原控制流的寄存器状态，这一般通过内核栈来保存。注意，我们需要用专门为操作系统准备的内核栈，而不是应用程序运行时用到的用户栈。

使用两个不同的栈主要是为了安全性：如果两个控制流（即应用程序的控制流和内核的控制流）使用同一个栈，在返回之后应用程序就能读到 Trap  控制流的历史信息，比如内核一些函数的地址，这样会带来安全隐患。于是，我们要做的是，在批处理操作系统中添加一段汇编代码，实现从用户栈切换到内核栈，并在内核栈上保存应用程序控制流的寄存器状态。





## 任务的概念形成

把应用程序的一次执行过程（也是一段控制流）称为一个 **任务** ，把应用执行过程中的一个时间片段上的执行片段或空闲片段称为 “ **计算任务片** ” 或“ **空闲任务片** ” 。当应用程序的所有任务片都完成后，应用程序的一次任务也就完成了。从一个程序的任务切换到另外一个程序的任务称为 **任务切换** 。为了确保切换后的任务能够正确继续执行，操作系统需要支持让任务的执行“暂停”和“继续”。

一旦一条控制流需要支持“暂停-继续”，就需要提供一种**控制流切换的机制**，而且需要保证程序执行的控制流被切换出去之前和切换回来之后，能够继续正确执行。这需要让**程序执行的状态（也称上下文）**，即在执行过程中同步变化的资源（如寄存器、栈等）保持不变，或者变化在它的预期之内。不是所有的资源都需要被保存，事实上只有那些对于程序接下来的正确执行仍然有用，且在它被切换出去的时候有被覆盖风险的那些资源才有被保存的价值。这些需要保存与恢复的资源被称为 **任务上下文 (Task Context)**  。



在前两章，我们已经看到了两种上下文保存/恢复的实例。让我们再来回顾一下它们：

- 第一章“应用程序与基本执行环境”中，我们介绍了 [函数调用与栈](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-function-call-and-stack) 。当时提到过，为了支持嵌套函数调用，不仅需要硬件平台提供特殊的跳转指令，还需要保存和恢复 [函数调用上下文](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-function-context)  。注意在上述定义中，函数调用包含在普通控制流（与异常控制流相对）之内，且始终用一个固定的栈来保存执行的历史记录，因此函数调用并不涉及控制流的特权级切换。但是我们依然可以将其看成调用者和被调用者两个执行过程的“切换”，二者的协作体现在它们都遵循调用规范，分别保存一部分通用寄存器，这样的好处是编译器能够有足够的信息来尽可能减少需要保存的寄存器的数目。虽然当时用了很大的篇幅来说明，但其实整个过程都是编译器负责完成的，我们只需设置好栈就行了。
- 第二章“批处理系统”中第一次涉及到了某种异常（Trap）控制流，即两条控制流的特权级切换，需要保存和恢复 [系统调用（Trap）上下文](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#term-trap-context) 。当时，为了让内核能够 *完全掌控* 应用的执行，且不会被应用破坏整个系统，我们必须利用硬件提供的特权级机制，让应用和内核运行在不同的特权级。应用运行在 U 特权级，它所被允许的操作进一步受限，处处被内核监督管理；而内核运行在 S 特权级，有能力处理应用执行过程中提出的请求或遇到的状况。

应用程序与操作系统打交道的核心在于硬件提供的 Trap 机制，也就是在 U 特权级运行的应用控制流和在 S 特权级运行的 Trap  控制流（操作系统的陷入处理部分）之间的切换。Trap 控制流是在 Trap  触发的一瞬间生成的，它和原应用控制流有着很密切的联系，因为它几乎唯一的目标就是处理 Trap 并恢复到原应用控制流。而且，由于 Trap  机制对于应用来说几乎是透明的，所以基本上都是 Trap 控制流在“负重前行”。Trap 控制流需要把 Trap  上下文（即几乎所有的通用寄存器）保存在自己的内核栈上，因为在 Trap 处理过程中所有的通用寄存器都可能被用到。可以回看 [Trap 上下文保存与恢复](http://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/4trap-handling.html#trap-context-save-restore) 小节。



## 任务切换的设计与实现

任务切换是来自两个不同应用在内核中的 Trap 控制流之间的切换。当一个应用 Trap 到 S 模式的操作系统内核中进行进一步处理（即进入了操作系统的 Trap 控制流）的时候，其 Trap 控制流可以调用一个特殊的 `__switch` 函数。这个函数表面上就是一个普通的函数调用：在 `__switch` 返回之后，将继续从调用该函数的位置继续向下执行。但是其间却隐藏着复杂的控制流切换过程。具体来说，调用 `__switch` 之后直到它返回前的这段时间，原 Trap 控制流 *A* 会先被暂停并被切换出去， CPU 转而运行另一个应用在内核中的 Trap 控制流 *B* 。然后在某个合适的时机，原 Trap 控制流 *A* 才会从某一条 Trap 控制流 *C* （很有可能不是它之前切换到的 *B* ）切换回来继续执行并最终返回。不过，从实现的角度讲， `__switch` 函数和一个普通的函数之间的核心差别仅仅是它会 **换栈** 。



需要保存一个应用的更多信息，我们将它们都保存在一个名为 **任务控制块** (Task Control Block) 的数据结构中：

```
// os/src/task/task.rs

#[derive(Copy, Clone)]
pub struct TaskControlBlock {
    pub task_status: TaskStatus,
    pub task_cx: TaskContext,
}
```

可以看到我们还在 `task_cx` 字段中维护了上一小节中提到的任务上下文。任务控制块非常重要，它是内核管理应用的核心数据结构。在后面的章节我们还会不断向里面添加更多内容，从而实现内核对应用更全面的管理。



我们可以总结一下应用的运行状态变化图：

![../_images/fsm-coop.png](http://rcore-os.cn/rCore-Tutorial-Book-v3/_images/fsm-coop.png)





# Lab1

## 实现fn sys_task_info(ti *mut TaskInfo) -> isize

### 需求分析

该系统调用需要能够查询当前正在执行的任务信息，包括**任务状态**、**任务使用的系统调用以及调用次数**、**任务运行的总时长**。

 ```rust
struct TaskInfo {
    status: TaskStatus,
    syscall_times: [u32; MAX_SYSCALL_NUM],
    time: usize
}
 ```

从上面的描述中，我们可以获取以下信息：

1. 正在运行的任务表示任务状态一定是Running。
2. 每个Task至少需要增加`syscall_times`字段和`time`字段。
3. 从`syscall_times`的定义可以看出，是在提示我们用`syscall`的`id`做索引来统计对应系统调用的调用次数。
4. 可以使用TaskManager来维护TaskInfo信息。



由于任务状态一定是Running因此可以不用记录直接赋值；

为了记录任务总运行时间，我们可以记录任务开始的时间，然后用任务结束的时间减去任务开始的时间即可。

综上所述，我们需要记录的就是任务使用的系统调用以及调用次数syscall_times和任务开始的时间start_time，可以参考TaskManager将syscall_times和start_time封装成TaskInfoInner。

```rust
#[derive(Clone, Copy)]
pub struct TaskInfoInner {
  pub syscall_times: [u32; MAX_SYSCALL_NUM],
  pub start_time: usize,
}
```

然后给Task对象加入一个task_info_inner字段。

```rust
pub struct TaskControlBlock {
    pub task_status: TaskStatus,
    pub task_cx: TaskContext,
    // 添加
    pub task_info_inner: TaskInfoInner,
}
```

之后给TaskManager实现对TaskInfo的getter和setter

```rust
// 调用系统调用，记录次数
fn set_syscall_times(&self, sys_call_id: usize) {..}

// 拿到当前task的TaskInfo
fn get_current_task_info(&self) -> TaskInfo {..}

// start_time在task初始化时赋值
lazy_static! {
  ..
  task_info_inner: TaskInfoInner {
    syscall_times: [0; MAX_SYSCALL_NUM],
    start_time: get_time_ms(),
  }
  ..
}
```

然后对外部提供接口

```rust
pub fn record_syscall(syscall_id: usize) {
    TASK_MANAGER.set_syscall_times(syscall_id);
}

pub fn get_task_info() -> TaskInfo {
    TASK_MANAGER.get_current_task_info()
}
```

最后在使用系统调用的时候记录，并且完善目的系统调用

```rust
pub fn syscall(syscall_id: usize, args: [usize; 3]) -> isize {
    ..
    set_syscall_times(syscall_id);
    ..
}

pub fn sys_task_info(ti: *mut TaskInfo) -> isize {
    unsafe { *ti = get_task_info() }
    0
}
```





# 视屏教学 虚拟内存相关知识

虚拟内存：把虚拟地址空间映射到实际的物理内存地址空间上面去

VA：虚拟内存地址

PA：物理内存地址

一个进程申请内存的过程：申请大小为0x1000大小的内存，放在以VA0x10000000作为开始的地方，然后再申请大小为0x2000大小的内存，放在以AV0x50000000作为开始的地方。在操作系统收到这两个请求的时候，首先查看PA有没有足够的内存，如果没有，直接返回OOM（out of memoty）。如果还有足够的内存分别在PA0x12340000和0x43210000两个位置，那么就会建立一个映射表

| VA         | PA         | Size   |
| ---------- | ---------- | ------ |
| 0x10000000 | 0x12340000 | 0x1000 |
| 0x50000000 | 0x43210000 | 0x2000 |

每一个对VA的读写都需要经过地址翻译才能找到真正的PA，所以这个过程必须要快。有两个优化方向

1. 让硬件自动去完成查表和转换
2. 优化表的存储数据结构

首先来考虑一个问题，内存地址转换的最小单位是什么？或者说SIze列最小可以是多少。

如果映射粒度太细，比如说每一个字节都建立一个映射，这样会倒置内存的绝大部份都用来存储映射表了，这样就本末倒置了。举个例子，在32位系统中，每一条映射信息需要储存4*3=12字节，也就是说，假设我想映射1G的内存地址空间，那么就需要而外的12G内存来储存映射表。

为了解决这个问题，就定义了一个叫做页的概念，通常**每个页的大小为4kb**，这是通用的大小，但是也会有一些系统使用了比4kb大很多的页，叫做巨页。我们主要关心4kb的页。

那么在映射表中，最小的映射单位就是页了，所以映射表通常也叫**页表**，表的每一行称为页表项（Page Table Entry，**PTE**）。



## 分级页表

