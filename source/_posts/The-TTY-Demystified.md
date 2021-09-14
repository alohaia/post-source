---
title: The TTY Demystified
comments: true
date: 2021-03-15 17:43:45
mathjax: true
tags:
    - Linux
    - TTY
categories:
    - [Learning, Computer, uncategorized]
    - [Projects, Programming, 自制伪终端]
---

原文：http://www.linusakesson.net/programming/tty/

参考：
- 维基百科
- 百度百科
- 《UNIX 环境高级编程 第 3 版·英文版》

一些概念：

TTY
: Teletype 或 Teletypewriter，原来指的是电传打字机，是通过串行线用打印机键盘通过阅读和发送信息的东西，和古老的电报机区别并不是很大。

<!-- more -->

一个 getty 的处理过程是：一个程序监视物理的 TTY/终端接口。

PTY/伪终端/终端模拟器
: 成对的逻辑终端设备（即 master 和 slave 设备，对 master 的操作会反映到 slave上）。

它们与实际物理设备并不直接相关。如果一个程序把 ptyp3 ( master 设备）看作是一个串行端口设
备，则它对该端口的读/ 写操作会反映在该逻辑终端设备对应的另一个 ttyp3 ( slave 设备）上面。
而 ttyp3 则是另一个程序用于读写操作的逻辑设备。telnet 主机 A 就是通过“伪终端”与主机 A 的
登录程序进行通信。

控制终端
: 可以使用命令 `ps –ax` 来查看进程与哪个控制终端相连。

UART:
: Universal Asynchronous Receiver and Transmitter, 通用异步收发器
: 一种通用串行数据总线，用于异步通信。该总线双向通信，可以实现全双工传输和接收。

Serial device
: 串行设备

嵌入式设计中，UART用来与PC进行通信，包括与监控调试器和其它器件，如 EEPROM 通信。

# The Use Case

## Line Editing

{% asset_img case1.png "case1'case1'" %}

Line discipline
: 一个编辑缓冲区和一些基本的编辑命令（退格，擦除单词，清除行，重新打印）

Line discipline 还包含用于字符回显（character echoing）以及回车（carriage return）和换行（linefeed）之间自动转换的选项。如果愿意，可以将其视为原始的内核级 `sed(1)`（Stream EDitor)

另外，内核提供了数种不同的 line discipline，一次只能将其中之一连接到指定串行设备。默认的 line discipline 提供了 line editing，被称为 **N_TTY**（[driver/tty/n_tty.c](https://github.com/torvalds/linux/blob/master/drivers/tty/n_tty.c)）。其他则被用于别的目的，如管理分组交换数据（packet switched data）（ppp，IrDA，serial mice）。

高级应用程序（如 Vim）可以通过将线路规则置于 raw mode 而不是默认的 cooked (or canonical) mode 来禁用这些功能。

大多数交互式应用程序（编辑器，邮件用户代理，shell，所有依赖于 curses 或 readline 的程序）以 raw mode 运行，并自行处理所有行编辑命令(line discipline 中提到的退格等)。

curses
: CRT (Cathode Ray Tube, 阴极射线管) screen handling and optimization package.
: See `man curses`

## Session Management

用户可能希望同时运行多个程序，并一次与一个程序交互。如果程序陷入无尽的循环，则用户可能想杀死（kill）或挂起（suspend）。在后台启动的程序应能够执行，直到它们尝试写入终端为止，此时应将其挂起。同样，用户输入应仅定向到前台程序。操作系统在 **TTY driver**（进程可能会截获某些信号，并尝试适应这种情况，但大多数情况并非如此。[driver/tty/tty_io.c](https://github.com/torvalds/linux/blob/master/drivers/tty/tty_io.c)）中实现这些功能。

一个操作系统进程是“alive”的（有执行上下文（execution context）），意味着它能进行某些动作。在面对对象的术语中，TTY driver 是一个**被动对象**（passive object） 。它有一些数据字段和一些方法，但是能做些什么的唯一方式只有通过进程的上下文（context）或内核中断处理程序（kernel interrupt handler）调用。Line discipline 同样也是一个被动对象（entity）。

UART driver、line discipline 实例和 TTY driver 这三者有时会一起被称为一个 **TTY device**，或者简单地称为 **TTY**。用户可以通过操作 `/dev` 目录下对应的设备文件来影响任何 TTY device的行为。由于需要对此设备文件的写入权限，用户在某个特定的 TTY 上登录时，必须成为设备文件的所有者（owner）。这个操作习惯上由以 root 权限运行的 `login(1)` 程序完成。

之前图片中的 physical line 当然也可以是一根很长的电话线：

{% asset_img case2.png "case2'case2'" %}

Modem
: Modulator-demodulator，调制解调器，用于计算机的数字信号和可沿电话线传播的脉冲信号之间的相互转换。

这没有多少变化，除了现在系统必须处理调制解调器延迟的情况。

> If the device is a modem, the `open` may delay inside the device driver until the modem is dialed and the call is answered.

让我们来研究典型的桌面环境。Linux 控制台（console）是这样运行的：

{% asset_img case3.png "case3'case3'" %}

TTY driver 和 line discipline 和之前例子中表现得一样，但是现在这里没有了 UART 和 physical terminal。取而代之的是一个用软件模拟并渲染成 VGA 显示的 video terminal （a complex state machine including a frame buffer of characters and graphical character attributes（对 OpenGL 有了解的话应该能比较好地理解这句话））。

VGA
: Video Graphics Array，视频图形阵列，是IBM于1987年提出的一个使用模拟信号的电脑显示标准。
: 这个标准已对于现今的个人电脑市场已经十分过时。现在的个人电脑大多使用 HDMI。

控制台子系统（console subsystem）有些死板。如果我们将终端模拟（terminal emulation）转到用户环境，事情会变得更加复杂。这是 `xterm(1)` 和其克隆（clone）的工作方式：

{% asset_img case4.png "case4'case4'" %}

# Processes

一个 Linux 进程可能处于以下状态之一：

{% asset_img linuxprocess.png "linuxprocess'linuxprocess'" %}

- R: 正在运行或可运行（在运行队列中）
- D: Uninterruptible sleep (wait for some event)，见下文
- S: Interruptible sleep (waiting for some event or signal)，见下文
- T: 暂停/挂起，接收到作业控制信号（job control signal）`SIGTSTP`（比如在终端正在执行命令时按下 Ctrl-z，会将此信号发送给前台进程）时或正在被 debugger 追踪时。
- Z: 僵尸进程，已中止但还未被父进程清除（比如父进程调用 `popen` 函数后没有调用 `pclose` 函数）。

运行 `ps l` 命令，可以查看哪些进程正在运行，哪些进程在休眠（sleeping）。如果进程正在休眠，`WCHAN` （”wait channel“，the name of the *wait queue* ；`man ps`: address of the kernel function where the process is sleeping）这一列说明了进程正在等待的内核事件（kernel event）。

```bash
$ ps l
F   UID   PID  PPID PRI  NI    VSZ   RSS WCHAN  STAT TTY        TIME COMMAND
0   500  5942  5928  15   0  12916  1460 wait   Ss   pts/14     0:00 -/bin/bash
0   500 12235  5942  15   0  21004  3572 wait   S+   pts/14     0:01 vim index.php
0   500 12580 12235  15   0   8080  1440 wait   S+   pts/14     0:00 /bin/bash -c (ps l) >/tmp/v727757/1 2>&1
0   500 12581 12580  15   0   4412   824 -      R+   pts/14     0:00 ps l
```

其中的“wait” wait queue 对应 system call `wait(2)`，一旦这些进程的一个子进程的状态改变了，它们就会被移到运行状态。有两种休眠状态：Interruptible sleep 和 uninterruptible sleep。前者（最常见的情况）意味着，即使进程是 wait queue 的一部分，当有信号被发送给它时，它也可能会被移到运行状态。如果你去查看内核源码，你会发现，任何等待事件的代码都必须在`schedule()` 返回后检查信号是否处于 _pending_ 状态，如果是，则中止 system call 。

> `man 2 wait`: All of these system calls are used to wait for state changes in a child of the calling process, and obtain information about the child whose state has changed. A state  change  is considered to be: the child terminated; the child was stopped by a signal; or the child was resumed by a signal. In the case of a terminated child, performing a wait allows the system to release the resources associated with the child; if a wait is not performed, then the terminated child remains in a "zombie" state.

在上面的 `ps` 的显示结果中， `STAT` 列显示了每个进程现在的状态。一列中可能包含一个或多个属性或标志。

- `S`: 进程是一个 [session leader](#session-leader)
- `+`: 进程属于一个前台进程组（foreground process group)

# Jobs and Sessions

作业控制（Job control）发生在你按下 `^Z` (Ctrl-Z) 来挂起一个程序，或用 `&` 来启动一个程序时。一个作业（job）和一个进程组（process group）是一样的。Internal shell commands（built-in commands in the shell）如 `jobs`，`fg` 和 `bg`，能用来操作一个**会话**（session） 中存在的作业。

每个会话都由一个 **session leader** 管理。session leader，一般是 shell，通过信号和 system call 的复杂协议与内核紧密合作。

下面的 Shell 交互……

{% asset_img jobs_show.png "jobs_show'jobs_show'" %}

……对应以下这些进程……

{% asset_img examplediagram.png "examplediagram'examplediagram'" %}

……和这些内核结构体。

- TTY Driver (`/dev/pts/0`)
```xxx
Size: 45x13
Controlling process group: (101)
Foreground process group: (103)
UART configuration (ignored, since this is an xterm):
  Baud rate, parity, word length and much more.
Line discipline configuration:
  cooked/raw mode, linefeed correction,
  meaning of interrupt characters etc.
Line discipline state:
  edit buffer (currently empty),
  cursor position within buffer etc.
```
- pipe0
```xxx
Readable end (connected to PID 104 as file descriptor 0)
Writable end (connected to PID 103 as file descriptor 1)
Buffer
```

基本思想是每个管道都是一项工作，因为管道中的每个进程都应同时进行操作（停止，恢复，终止）。这就是为什么 `kill(2)` 允许您将信号发送到整个进程组。默认情况下， `fork(2)` 将新创建的子进程与其父进程放置在同一进程组中，因此，例如，键盘产生的 `^C`（Ctrl-c） 会同时影响父子进程。作为其会话负责人职责的一部分，shell 程序每次启动管道（pipeline）时都会创建一个新的进程组。

TTY driver 仅以被动方式跟踪前台进程组 ID。Session leader 必须在必要时显式更新此信息（TTY driver 跟踪的前台进程组 ID）。类似地，TTY driver 跟踪所连接终端的大小（行列数），但是此信息必须由终端模拟器甚至是用户显式更新。

如上图所示，几个进程将它们的标准输入连接到了 `/dev/pts/0`。但是只有前台作业（`ls | sort` 管道）将被允许接收来自 TTY 的输入。同样，仅前台作业将被允许写入 TTY 设备（在默认配置情况下）。如果 `cat` 进程试图写入 TTY，则内核将使用信号将其挂起。

```bash
> cat &
[1] 170055
[1]  + 170055 suspended (tty input)  cat
```

# Signal Madness

现在，让我们仔细看看内核中的 TTY driver，line discipline 和 UART driver 如何与用户态进程（userland processes）通信。

UNIX 文件，包括 TTY driver 文件，当然可以通过魔法一般的 `ioctl(2)` （UNIX 的瑞士军刀）调用来进行读写，并对其进行进一步操作，对此它已经进行了许多与TTY相关的操作定义。不过，`ioctl` 请求必须从进程启动，因此当内核需要与应用程序异步通信时，不能使用它们。

道格拉斯·亚当斯（Douglas Adams）在《银河系漫游指南》中提到了一个极其沉闷的星球，里面栖息着一群沮丧的人类和某种具有锋利牙齿的的物种，它们通过在大腿上用力地咬人与人类交流。这和内核通过发送“麻痹”或“死亡”信号与进程交流的 UNIX 惊人地相似。进程可能会截获某些信号，并尝试做出回应，但大多数进程不会这样做。

因此，信号是一种粗糙的机制，它允许内核与进程异步通信。UNIX 中的信号不是干净的或通用的。相反，每个信号都是唯一的，必须单独研究。

你可以使用命令 `kill -l` 查看系统实现的信号。可能是这样的：

```bash
$ kill -l
 1）SIGHUP 2）SIGINT 3）SIGQUIT 4）SIGILL
 5）SIGTRAP 6）SIGABRT 7）SIGBUS 8）SIGFPE
 9）SIGKILL 10）SIGUSR1 11）SIGSEGV 12）SIGUSR2
13）SIGPIPE 14）SIGALRM 15）SIGTERM 16）SIGSTKFLT
17）SIGCHLD 18）SIGCONT 19）SIGSTOP 20）SIGTSTP
21）SIGTTIN 22）SIGTTOU 23）SIGURG 24）SIGXCPU
25）SIGXFSZ 26）SIGVTALRM 27）SIGPROF 28）SIGWINCH
29）SIGIO 30）SIGPWR 31）SIGSYS 34）SIGRTMIN
35）SIGRTMIN + 1 36）SIGRTMIN + 2 37）SIGRTMIN + 3 38）SIGRTMIN + 4
39）SIGRTMIN + 5 40）SIGRTMIN + 6 41）SIGRTMIN + 7 42）SIGRTMIN + 8
43）SIGRTMIN + 9 44）SIGRTMIN + 10 45）SIGRTMIN + 11 46）SIGRTMIN + 12
47）SIGRTMIN + 13 48）SIGRTMIN + 14 49）SIGRTMIN + 15 50）SIGRTMAX-14
51）SIGRTMAX-13 52）SIGRTMAX-12 53）SIGRTMAX-11 54）SIGRTMAX-10
55）SIGRTMAX-9 56）SIGRTMAX-8 57）SIGRTMAX-7 58）SIGRTMAX-6
59）SIGRTMAX-5 60）SIGRTMAX-4 61）SIGRTMAX-3 62）SIGRTMAX-2
63）SIGRTMAX-1 64）SIGRTMAX
```

```bash
> kill -l
HUP INT QUIT ILL TRAP ABRT BUS FPE KILL USR1 SEGV USR2 PIPE ALRM TERM STKFLT CHLD CONT STOP TSTP TTIN TTOU URG XCPU XFSZ VTALRM PROF WINCH POLL PWR SYS
```

如你所见，信号从 1 开始编号。但是，当将它们用于位掩码（例如 `ps s` 的输出）时，最低有效位对应于信号 1。

> UNIX 中，没有任何一个有效信号的值为 0。

本文将重点介绍以下信号：`SIGHUP`，`SIGINT`，`SIGQUIT`，`SIGPIPE`，`SIGCHLD`，`SIGSTOP`，`SIGCONT`，`SIGTSTP`，`SIGTTIN`，`SIGTTOU` 和 `SIGWINCH`。

SIGHUP
: 默认行为：**Terminate**
: 可能行为：Terminate，Ignore，Function call
: 检测到挂起（hangup）的情况时，UART driver 会将 `SIGHUP` 信号发送到整个会话。这一般会杀掉会话中的所用进程。有些进程。如 `nohub(1)` 和 `screen(1)` 会从它们的进程（和 TTY）中分离，所以它们的子进程不会注意到挂起。

SIGINT
: 默认行为：**Terminate**
: 可能行为：Terminate，Ignore，Function call
: 当交互式注意字符（interactive attention character，通常是 `^C`，ASCII 码为 3）出现在输入流中时，TTY driver 会将 `SIGINT` 发送给当前的前台作业，除非此行为被关闭。只要有 TTY device 的访问（access）权限，就能改变交互式注意字符和开关这一特性。另外，session leader (manager) 会跟踪每个作业的 TTY 配置，并在每次切换作业时更新 TTY （配置）。

SIGQUIT
: 默认行为：**Core dump**
: 可能行为：Core dump，Ignore，Function call
: `SIGQUIT` 和 `SIGINT` 的工作方式相似，但是对应字符为 `^\` 并且默认行为不同。

SIGPIPE
: 默认行为：**Terminate**
: 可能行为：Terminate，Ignore，Function call
: 内核会将 `SIGPIPE` 发送给任何〔尝试写入〔没有读取器的管道〕的〕进程。这是很有用的，因为如果没有这个，`yes | head` 这样的进程将永远不会中止。

> `yes` 不断输出一个字符串，直到被杀死为止。

SIGCHLD
: 默认行为：**Ignore**
: 可能行为：Ignore，Function call
: 当一个进程死亡或改变状态（停止或继续），内核会将一个 `SIGCHLD` 信号发送给其父进程。`SIGCHLD` 携带了额外信息，如该进程的 ID，用户 ID，exit status （或 termination signal）和其他 执行时间数据（execution time statistics）。Session leader 借助此信号追踪其作业。

SIGSTOP
: 默认行为：**Suspend**
: 可能行为：Suspend
: 此信号会无条件地将接受者挂起，也就是说，其行为无法被重新设置。然而，请注意内核不会在作业控制过程中发送 `SIGSTOP` 信号。作为替代，`^Z` 通常会触发 `SIGTSTP`。`SIGTSTP` 可以被应用程序拦截。应用程序可以将光标移动到屏幕底部，或者将终端置于已知状态（known state），然后使用 `SIGSTOP` 让自己进入睡眠状态。

SIGCONT
: 默认行为：**Wake up**
: 可能行为：Wake up，Wake up + Function call
: `SIGCONT` 会将一个被停止的进程取消挂起（un-suspend）。用户调用 `fg` 命令时，它会被显式发送给指定进程。

SIGTSTP
: 默认行为：**Suspend**
: 可能行为：Suspend，Ignore，Function call
: `SIGTSTP` 的工作方式和 `SIGINT` 与 `SIGQUIT` 类似，但是 “magic” 字符一般是 `^Z` 并且默认行为是挂起进程。

SIGTTIN
: 默认行为：**Suspend**
: 可能行为：Suspend，Ignore，Function call
: 如果一个后台作业中的进程尝试从 TTY device 读取，TTY 会向整个作业发送 `SIGTTIN`。这通常会挂起该作业。

SIGTTOU
: 默认行为：**Suspend**
: 可能行为：Suspend，Ignore，Function call
: 如果一个后台作业中的进程尝试写入到 TTY device，TTY 会向整个作业发送 `SIGTOUT`。这通常会挂起该作业。可以为单独的 TTY 关闭此功能。

SIGWINCH
: 默认行为：**Ignore**
: 可能行为：Ignore，Function call
: 如前所述，TTY device 会追踪终端的大小（行列数），但是此信息需要手动更新。每当终端窗口大小改变时，TTY 会发送一个 `SIGWINCH` 信号给前台作业。行为良好的（well-behaving）交互式应用程序，如编辑器，会对此做出回应——获取新的终端大小并重新绘制自己。

# A Example

假设您正在（基于终端的）编辑器（如 Vim）中编辑文件。光标位于屏幕中间的某处，编辑器正忙于执行一些处理器密集型任务（processor intensive task），例如对大文件的搜索和替换操作。现在你按 `^Z`。由于已将 line discipline 配置为拦截该字符（`^Z` 是一个字节，ASCII 码为 26），因此你不必等待编辑器完成其任务再开始从 TTY device 读取。取而代之的是，line discipline 子系统立即将 `SIGTSTP` 发送到前台进程组。该进程组包含编辑器以及由它创建的任何子进程。

编辑器已为 `SIGTSTP` 安装了信号处理程序（即源代码中对 `SIGTSTP` 进行拦截并进行处理），因此内核将进程转到执行信号处理程序（signal handler，源码中的一个函数）的代码。该代码通过将相应的控制序列写入 TTY device，将光标移动到屏幕的最后一行。由于编辑器仍在前台，因此将按要求发送控制序列。但是随后，编辑器将 `SIGSTOP` 发送到其自己的进程组。

编辑器进程现在被挂起了。这件事，包括被挂起的进程的 ID，由内核通过 `SIGCHLD` 报告给 session leader （一般为 shell）。当前台作业的所有进程都被挂起时，session leader 会从 TTY device 读取当前配置，并将其保存以供以后（恢复被挂起的前台作业时）检索。Session leader 接下来使用 `ioctl` 调用将自己安装为 TTY 的前台进程组。然后，它会打印 `[1]+ Stopped` 之类的信息来提示用户有一个作业刚刚被挂起了。

此时，`ps(1)`（`ps l`）将告诉你编辑器处于停止状态（T）。如果我们尝试通过使用 shell 内置命令 `bg` 或使用 `kill(1)` 将 `SIGCONT` 发送到进程来唤醒它，则编辑器将开始执行其 `SIGCONT` 信号处理程序（signal handler）。该信号处理程序可能会通过写入 TTY 设备来尝试重绘编辑器 GUI（其实是 TUI）。但是由于编辑器现在是后台作业，因此 TTY device 将不允许这样（唤醒后台作业）。相反，TTY 会将 `SIGTTOU` 发送给编辑器，然后再次停止它。这件事将会通过 `SIGCHLD` 信号传达给 session leader，并且 shell 程序将再次向终端写入 `[1]+ Stopped` 之类的信息。

然而当我们使用 `fg` 命令（如 `fg %vim`）时，shell 首先会恢复原先保存的 line discipline 配置。这告诉 TTY driver 编辑器作业从现在起应该被视为前台作业。最后，shell 会向编辑器进程组发送 `SIGCONT` 信号。编辑器进程将会尝试重绘其 GUI （其实是 TUI）。因为它现在是前台进程组的一部分，所以这次它不会被 `SIGTTOU` 信号中断。

# Flow control and blocking I/O

（流控制和阻断式 I/O）

在 `xterm` 中运行 `yes`，你会看见许多只有一个 “y” 的行从你眼前掠过。自然，`yes` 生成这些行的速度会比 `xterm` 应用程序解析它们、更新其帧缓存、与 X server 沟通以滚动屏幕等工作的速度更快。这些程序是怎样协同合作的？

答案就在阻塞式 I/O。伪终端只能在其内核缓冲区中保留一定数量的数据，并且当该缓冲区已满并且 `yes` 尝试调用 `write(2)` 时，`write(2)` 将阻塞（block），将 `yes` 进程移至 interruptible sleep 状态，其中它会一直保持此状态直到 `xterm` 进程有机会读取一些缓冲的字节为止。

如果将 TTY 连接到串行端口，会发生相同的情况。`yes` 将能够以比 9600 波特高得多的速率传输数据，但是如果串行端口被限制为该速度，则内核缓冲区很快就会填满，并且随后所有的 `write(2)` 调用都会阻塞该进程（如果进程已请求非阻塞式 I/O，则以错误码 `EAGAIN` 失败）。

如果我告诉您，即使内核缓冲区中有剩余空间，也可以将 TTY 显式置于阻塞状态，会怎么样？除非另行通知（further notice），否则所有尝试将对 TTY `write(2)` 的进程都会自动阻塞。这种功能的用途是什么？

假设我们正在操作 9600 波特的旧 VT-100 硬件。我们刚刚发送了一个复杂的控制序列，要求终端滚动显示。此时，终端由于某些问题无法进行滚动操作，以致无法以 9600 波特的满速率接收新数据。嗯，实际上，终端 UART 仍以 9600 波特运行，但是终端中没有足够的缓冲区空间来保留所接收字符的积压。这是将 TTY 置于阻塞状态的好时机。但是我们如何从终端上做到这一点呢？

我们已经看到，可以将 TTY 设备配置为对某些数据字节进行特殊处理。例如，在默认配置中，接收到的 `^C` 字节不会通过 `read(2)` 传递给应用程序，而是将 `SIGINT` 传递给前台作业。以类似的方式，可以将 TTY 配置为对 停止流字节（stop flow byte，通常是 `^S`，ASCII 码为 19） 和 开始流字节（start flow byte，通常是 `^Q`，ASCII 码为 17）作出反应。旧的终端硬件会自动发送这些字节，并期望操作系统相应地调节其数据流。这称为流控制。

这里有一个重要的区别：对 TTY 的写入操作可能会由于流控制，也可能会由于内核缓冲区空间不足而停止，这会阻塞你的进程，但是因为后台作业尝试写入 TTY 而引发的 `SIGTTOU` 会挂起整个进程组。我不知道为什么 UNIX 的设计者会发明 `SIGTTOU` 和 `SIGTTIN` 而不是依靠阻塞式 I/O，但是我的猜测是负责作业控制的 TTY driver 旨在监控和操控整个作业，而不是其中的单个进程。

# Configuring the TTY Device

要找出当前 shell 的控制 TTY（controlling terminal）是什么，可以参考前面所述的 `ps l` 输出中的 TTY 列，或者可以简单地运行 `tty(1)` 命令。

进程可以使用 `ioctl(2)` 读取或修改打开的 TTY device 的配置。该 API 在 `tty_ioctl(4)`（ioctls for terminals and serial lines）中进行了描述。由于它是 Linux 应用程序和内核之间的二进制接口的一部分，因此它将在各 Linux 版本之间保持稳定。但是，该接口是不可移植的，应用程序应该使用 `termios(3)` 手册页中描述的 POSIX 封装（wrapper）。

我不会详细介绍 `termios(3)` 接口，但是如果您正在编写 C 程序，并且想在 `^C` 被转换为 `SIGINT` 之前对其进行拦截，请禁用行编辑（line editing）和字符回显（character echoing），更改串行端口的波特率，关闭流控制等。然后您将在上述手册页中找到所需的内容。

还有一个命令行工具，称为 `stty(1)`，用于操纵 TTY 设备。它使用了 `termios(3)` API。

Let's try it!

```bash
$ stty -a
speed 38400 baud; rows 73; columns 238; line = 0;
intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = <undef>; eol2 = <undef>; swtch = <undef>; start = ^Q; stop = ^S; susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V; flush = ^O; min = 1; time = 0;
-parenb -parodd cs8 -hupcl -cstopb cread -clocal -crtscts
-ignbrk brkint ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl ixon -ixoff -iuclc -ixany imaxbel -iutf8
opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt echoctl echoke
```

```bash
> stty -a
speed 38400 baud; rows 15; columns 92; line = 0;
intr = ^C; quit = ^\; erase = ^?; kill = ^U; eof = ^D; eol = <undef>; eol2 = <undef>;
swtch = <undef>; start = ^Q; stop = ^S; susp = ^Z; rprnt = ^R; werase = ^W; lnext = ^V;
discard = ^O; min = 1; time = 0;
-parenb -parodd -cmspar cs8 -hupcl -cstopb cread -clocal -crtscts
-ignbrk -brkint -ignpar -parmrk -inpck -istrip -inlcr -igncr icrnl ixon -ixoff -iuclc -ixany
-imaxbel iutf8
opost -olcuc -ocrnl onlcr -onocr -onlret -ofill -ofdel nl0 cr0 tab0 bs0 vt0 ff0
isig icanon iexten echo echoe echok -echonl -noflsh -xcase -tostop -echoprt echoctl echoke
-flusho -extproc
```

该 `-a` 标志告诉 `stty` 显示的*所有*设置。默认情况下，它将查看连接到你的 Shell 的 TTY device，但是你可以使用 `-F` 指定另一个设备。

其中一些设置取决于 UART 参数，有些会影响 line discipline，有些则与作业控制有关。全部放在一起供君取用。让我们先看一下第一行：


| 项目          | 相关            | 描述                                                                                                                                                                 |
|---------------|-----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| speed         | UART            | 波特率（baud rate）。伪终端无视。                                                                                                                                    |
| rows, columns | TTY driver      | 连接到此 TTY device 的终端的大小（以字符为单位）。基本上，这只是内核空间中的一对变量，你可以自由设置和获取。设置它们将导致 TTY driver 将 `SIGWINCH` 发送给前台作业。 |
| line          | Line discipline | 连接到 TTY 的 line discipline。0 表示 `N_TTY`，所有的有效数字都在 `/proc/tty/ldisc` 中被列出。为列出的数字好像是 `N_TTY` 的别名，但是不要依赖这点。                  |

请尝试以下操作：启动 xterm。记下其 TTY device（由 `tty` 显示）和
其大小（由 `stty -a` 显示）。在 xterm 中启动 vim （或其他全屏终端应用程序） 。编辑器向
TTY device 查询当前的终端大小，以填满整个窗口。现在，从另一个 shell 窗口中键入：

```bash
stty -F X rows Y
```

其中 *X* 是前面记录的 TTY device， *Y* 是前面记录的终端高度的一半。这会更新内核内存中的 TTY 数据结构体（TTY data structure），并向编辑器发送 `SIGWINCH` 信号，该编辑器将仅使用可用窗口区域的上半部分立即重绘自身。

`stty -a` 输出的第二列列出了所有特殊字符。启动一个新的 xterm 并尝试以下操作：

```bash
stty intr o
```

现在，`o` 而不是 `^C`，会向前台进程发送 `SIGINT`。尝试启动什么程序，如 `cat`，并且验证你现在无法用 `^C` 杀掉它。然后，试着向其中输入 "hello"。

有时，您可能会遇到在 UNIX 系统中退格键不起作用的情况。当终端模拟器发送的退格键（ASCII 码为 8 或 127）与 TTY device 中设置的 `erase` 不匹配时，就会发生这种情况。要解决此问题，通常可以键入 `stty erase ^H`（设置为 ASCII 8）或 `stty erase ^?`（设置为 ASCII 127）。但是请记住，许多终端应用程序都使用 `readline`，这会将线路规则置于原始模式。这些应用程序不受前面 `stty` 命令的影响。

最后，`stty -a` 列出了一堆开关。不出所料，它们没有按特定顺序列出。其中一些与 UART 相关，一些影响 line discipline 的行为，一些用于流控制，一些用于作业控制。破折号（-）表示开关已关闭；否则打开。所有开关都在 `stty(1)` 手册页中进行了说明，因此，我仅简要介绍其中的一些：

`icanon` 开启或关闭 canonical (line-based) 模式。在新的 xterm 中尝试以下命令：

```bash
stty -icanon; cat
```

注意所有的行编辑字符，如退格和 `^U`，都无法正常使用。另外注意 `cat` 一次性接收和输出一个字符，而不是（一次性接收和输出）一行。

```bash
> stty -icanon; cat
aassddff^?^?^?^?^?^?^?aassddffaassdd^?^?^?^C
>
```

`echo` 启用字符回显，默认开启。重新回到 canonical 模式（`stty icanon`），然后尝试：

```bash
stty -echo; cat
```

你输入的同时，终端模拟器将信息发送给内核。通常，内核会显示相同的信息到终端模拟器，使你能够看到你输入了什么。没有字符回显，你看不到你输入了什么，但是我们在 cooked 模式，所以行编辑功能任然能发挥作用。一旦你按下回车键，line discipline 就会将编辑缓冲发送给 `cat`，这将显示你写入的内容。

`tostop` 控制后台作业是否被允许写入终端。先来试试这个：

```bash
stty tostop; (sleep 5; echo hello, world) &
```

`&` 使括号中的命令在后台运行。五秒后，作业中的 `cat` 尝试写入 TTY。TTY device 会使用 `SIGTOUT`将其挂起，你的 shell 可能会对此做出报告，可能会是立即，也可能会是在将要发送新的提示符时（这种情况可以按下回车以获得新的提示符）。现在杀死后台进程，尝试以下命令：

```bash
stty -tostop; (sleep 5; echo hello, world) &
```

你会先得到提示符，但是在五秒后，后台作业在你输入的途中会发送 `hello world` 到终端。

最后，`stty sane` 会重置你的 TTY device 到合理的设置（一般为默认设置）。

# Conclusion

我希望本文为您提供了足够的信息，以使您能够熟悉 TTY drivers 和 line disciplines，以及它们与终端、line editing 和作业控制的关系。可以在我提到的各种手册页以及 glibc 手册（`info libc` 中的 “Job Control”）中找到更多详细信息。

最后，尽管我没有足够的时间回答所有问题，但我欢迎您对此网站以及本网站其他页面的反馈。
谢谢阅读！

