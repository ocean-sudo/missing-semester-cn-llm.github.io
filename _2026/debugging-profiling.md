---
layout: lecture
title: "调试与性能分析"
description: >
  学习如何使用日志与调试器调试程序，以及如何进行性能分析。
thumbnail: /static/assets/thumbnails/2026/lec4.png
date: 2026-01-15
ready: true
panopto: "https://mit.hosted.panopto.com/Panopto/Pages/Viewer.aspx?id=a72c48e3-5eb2-46fa-aa03-b3b700e1ca8d"
video:
  aspect: 56.25
  id: 8VYT9TcUmKs
---

编程的一条黄金法则是，代码不会做你期望它做的事情，而是做你告诉它做的事情。弥合这一差距有时可能是一项相当困难的壮举。在本次讲座中，我们将介绍处理有错误和资源匮乏的代码的有用技术：调试和分析。

# 调试

## Printf 调试和日志记录

> “最有效的调试工具仍然是仔细的思考，加上明智地放置打印语句” - Brian Kernighan，_Unix for Beginners_。

调试程序的第一种方法是在检测到问题的位置添加打印语句，并不断迭代，直到提取足够的信息来了解导致问题的原因。

第二种方法是在程序中使用日志记录，而不是临时打印语句。日志记录本质上是“更加小心地打印”，通常通过日志记录框架完成，该框架包括对以下内容的内置支持：

- 将日志（或日志子集）定向到其他输出位置的能力；
- 设置严重性级别（例如 INFO、DEBUG、WARN、ERROR 等）并允许您根据这些级别过滤输出；和
- 支持与日志条目相关的数据的结构化记录，然后可以在事后更轻松地提取这些数据。

记录您通常还会主动放入 while 的语句
编程，以便您需要调试的数据可能已经存在！
事实上，一旦您使用打印发现并解决了问题
语句，通常值得将这些打印转换为正确的
在删除语句之前记录它们。这样，如果出现类似的bug
将来，您将已经拥有所需的诊断信息
无需修改代码。

> **第三方日志**：许多程序支持 `-v` 或 `--verbose` 标志，以便在运行时打印更多信息。这对于发现给定命令失败的原因非常有用。有些甚至允许重复该标志以获取更多详细信息。调试服务（数据库、Web 服务器等）问题时，请检查其日志 — 通常位于 Linux 上的 `/var/log/` 中。使用 `journalctl -u <service>` 查看 systemd 服务的日志。对于第三方库，检查它们是否支持通过环境变量或配置进行调试日志记录。

## 调试器

当您知道要打印什么并且可以轻松修改和重新运行代码时，打印调试效果很好。当您不确定需要什么信息时，当错误仅在难以重现的情况下出现时，或者当修改和重新启动程序成本高昂（启动时间长、重新创建的状态复杂等）时，调试器就变得很有价值。

调试器是一种程序，可让您在程序执行时与其进行交互，从而使您能够：

- 当到达某一行时停止执行。
- 一次逐步执行一项指令。
- 崩溃后检查变量值。
- 当满足给定条件时有条件地停止执行。
- 以及许多更高级的功能。

大多数编程语言都支持（或附带）某种形式的调试器。最通用的是**通用调试器**，例如 [`gdb`](https://www.gnu.org/software/gdb/) (GNU 调试器) 和 [`lldb`](https://lldb.llvm.org/) (LLVM 调试器)，它们可以调试任何本机二进制文件。许多语言还具有**特定于语言的调试器**，它们与运行时集成更紧密（例如 Python 的 pdb 或 Java 的 jdb）。

`gdb` 是 C、C++、Rust 和其他编译语言事实上的标准调试器。它可以让您探测几乎任何进程并获取其当前的机器状态：寄存器、堆栈、程序计数器等等。

一些有用的 GDB 命令：

- `run` - 启动程序
- `b {function}` 或 `b {file}:{line}` - 设置断点
- `c` - 继续执行
- `step` / `next` / `finish` - 步入/跨过/走出
- `p {variable}` - 打印变量值
- `bt` - 显示回溯（调用堆栈）
- `watch {expression}` - 值更改时中断

> 考虑使用 GDB 的 TUI 模式（`gdb -tui` 或按 GDB 内的 `Ctrl-x a`），以在命令提示符旁边显示源代码的分屏视图。

### 记录重放调试

一些最令人沮丧的错误是_Heisenbugs_：当您尝试观察它们时，这些错误似乎会消失或改变行为。竞争条件、与时间相关的错误以及仅在某些系统条件下出现的问题都属于这一类。传统的调试在这里通常是无用的，因为再次运行程序会产生不同的行为（例如，打印语句可能会充分减慢代码速度，从而不再发生竞争）。

**记录重放调试** 通过记录程序的执行并允许您根据需要多次确定地重放它来解决这个问题。更好的是，您可以“反向”执行，以准确找到出错的地方。

[rr](https://rr-project.org/) 是一个强大的 Linux 工具，可以记录程序执行并允许确定性重放并具有完整的调试功能。它与 GDB 一起使用，因此您已经了解该接口。

基本用法：

```bash
# Record a program execution
rr record ./my_program

# Replay the recording (opens GDB)
rr replay
```

奇迹发生在重播期间。由于执行是确定性的，因此可以使用**反向调试**命令：

- `reverse-continue` (`rc`) - 向后运行直到遇到断点
- `reverse-step` (`rs`) - 向后退一行
- `reverse-next` (`rn`) - 向后退一步，跳过函数调用
- `reverse-finish` - 向后运行直到进入当前函数

这对于调试来说非常强大。假设您发生了崩溃，您可以：而不是猜测错误在哪里并设置断点：

1. 跑到崩溃处
2.检查损坏状态
3. 在损坏的变量上设置观察点
4. `reverse-continue` 准确查找损坏的位置

**何时使用 rr:**
- 间歇性失败的片状测试
- 竞争条件和线程错误
- 难以重现的崩溃
- 任何你希望“回到过去”的错误

> 注意：rr 仅适用于 Linux，并且需要硬件性能计数器。它不适用于不公开这些计数器的虚拟机（例如大多数 AWS EC2 实例），并且不支持 GPU 访问。对于 macOS，请查看 [Warpspeed](https://warpspeed.dev/)。

> **rr 和并发**：因为 rr 确定性地记录执行，所以它串行化线程调度。这意味着如果某些竞争条件取决于特定的时间，则它们可能不会在 rr 下显现。 rr 对于调试竞赛仍然有用 - 一旦捕获失败的运行，您就可以可靠地重播它 - 但您可能需要多次记录尝试才能捕获间歇性错误。对于不涉及并发的错误，rr 表现最出色：您始终可以重现确切的执行情况并使用反向调试来查找损坏。

## 系统调用跟踪

有时您需要了解程序如何与操作系统交互。程序通过[系统调用](https://en.wikipedia.org/wiki/System_call) 向内核请求服务——打开文件、分配内存、创建进程等等。跟踪这些调用可以揭示程序挂起的原因、它试图访问哪些文件，或者它在哪里等待。

### strace (Linux) 和 dtruss (macOS)

[`strace`](https://www.man7.org/linux/man-pages/man1/strace.1.html) 可让您观察程序进行的每个系统调用：

```bash
# Trace all system calls
strace ./my_program

# Trace only file-related calls
strace -e trace=file ./my_program

# Follow child processes (important for programs that start other programs)
strace -f ./my_program

# Trace a running process
strace -p <PID>

# Show timing information
strace -T ./my_program
```

> 在 macOS 和 BSD 上，使用 [`dtruss`](https://www.manpagez.com/man/1/dtruss/)（包装 `dtrace`）来实现类似的功能：

> 要更深入地了解 `strace`，请查看 Julia Evans 的优秀 [strace zine](https://jvns.ca/strace-zine-unfolded.pdf)。

### bpftrace 和 eBPF

[eBPF](https://ebpf.io/)（扩展伯克利数据包过滤器）是一项强大的 Linux 技术，允许在内核中运行沙盒程序。 [`bpftrace`](https://github.com/iovisor/bpftrace) 提供了用于编写 eBPF 程序的高级语法。这些是在内核中运行的任意程序，因此具有巨大的表达能力（尽管也是有点笨拙的类似 awk 的语法）。它们最常见的用例是调查正在调用哪些系统调用，包括聚合（如计数或延迟统计）或内省（甚至过滤）系统调用参数。

```bash
# Trace file opens system-wide (prints immediately)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args->filename)); }'

# Count system calls by name (prints summary on Ctrl-C)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* { @[probe] = count(); }'
```

但是，您也可以使用 [`bcc`](https://github.com/iovisor/bcc) 等工具链直接用 C 语言编写 eBPF 程序，该工具链还附带[许多方便的工具](https://www.brendangregg.com/blog/2015-09-22/bcc-linux-4.3-tracing.html)，例如用于打印磁盘操作延迟分布的 `biosnoop` 或用于打印所有打开文件的 `opensnoop` 。

`strace` 很有用，因为它很容易“启动并运行”，而 `bpftrace` 是当您需要较低开销、想要跟踪内核函数、需要进行任何类型的聚合等时应该使用的。请注意，`bpftrace` 必须作为 `root` 运行，并且它通常监视整个内核，而不仅仅是特定进程。要定位特定程序，您可以按命令名称或 PID 进行过滤：

```bash
# Filter by command name (prints summary on Ctrl-C)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /comm == "bash"/ { @[probe] = count(); }'

# Trace a specific command from startup using -c (cpid = child PID)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /pid == cpid/ { @[probe] = count(); }' -c 'ls -la'
```

`-c` 标志运行指定的命令并将 `cpid` 设置为其 PID，这对于从程序启动的那一刻起跟踪程序很有用。当跟踪命令退出时，bpftrace 会打印聚合结果。

### 网络调试

对于网络问题，[`tcpdump`](https://www.man7.org/linux/man-pages/man1/tcpdump.1.html) 和 [Wireshark](https://www.wireshark.org/) 可让您捕获和分析网络数据包：

```bash
# Capture packets on port 80
sudo tcpdump -i any port 80

# Capture and save to file for Wireshark analysis
sudo tcpdump -i any -w capture.pcap
```

对于 HTTPS 流量，加密会降低 tcpdump 的用处。 [mitmproxy](https://mitmproxy.org/) 等工具可以充当拦截代理来检查加密流量。浏览器开发人员工具（“网络”选项卡）通常是调试来自 Web 应用程序的 HTTPS 请求的最简单方法 - 它们显示解密的请求/响应数据、标头和计时。

## 内存调试

内存错误（缓冲区溢出、释放后使用、内存泄漏）是最危险且最难调试的错误。它们通常不会立即崩溃，但会以稍后导致问题的方式破坏内存。

### 消毒剂

查找内存错误的一种方法是使用**清理程序**，它们是编译器功能，可在运行时检测代码以检测错误。例如，广泛使用的 **AddressSanitizer (ASan)** 检测：
- 缓冲区溢出（堆栈、堆和全局）
- 免费后使用
- 归还后使用
- 内存泄漏

```bash
# Compile with AddressSanitizer
gcc -fsanitize=address -g program.c -o program
./program
```

有多种有用的消毒剂：

- **ThreadSanitizer (TSan)**：检测多线程代码中的数据争用 (`-fsanitize=thread`)
- **MemorySanitizer (MSan)**：检测未初始化内存的读取 (`-fsanitize=memory`)
- **UndefinedBehaviorSanitizer (UBSan)**：检测未定义的行为，例如整数溢出 (`-fsanitize=undefined`)

Sanitizers 需要重新编译，但速度足够快，可以在 CI 管道和常规开发过程中使用。

### Valgrind：当你无法重新编译时

[Valgrind](https://valgrind.org/) 相反，在类似于虚拟机的东西中运行您的程序以检测内存错误。它比消毒剂慢，但不需要重新编译：

```bash
valgrind --leak-check=full ./my_program
```

在以下情况下使用 Valgrind：
- 你没有源代码
- 无法重新编译（第三方库）
- 您需要无法用作消毒剂的特定工具

Valgrind 实际上是一个非常强大的受控执行环境，稍后当我们进行分析时我们会看到更多它！

## AI 调试

大型语言模型已成为非常有用的调试助手。他们擅长某些与传统工具相辅相成的调试任务。

**LLM的闪光点：**

- **解释神秘的错误消息**：编译器错误，尤其是来自 C++ 模板或 Rust 的借用检查器的错误，可能非常神秘。LLM可以将它们翻译成简单的英语并提出修复建议。

- **穿越语言和抽象边界**：如果您正在调试跨多种语言的问题（例如，通过 Python 绑定体现的 C 库中的错误），LLM 可以帮助导航不同的层。他们特别擅长理解 FFI 边界、构建系统问题和跨语言调试（例如，我的程序错误，但我相信这是因为我的依赖项之一中的错误）。

- **将症状与根本原因相关联**：“我的程序运行良好，但使用的内存比预期多 10 倍”是LLM可以帮助调查的一种模糊症状，建议可能的原因以及要查找的内容。

- **分析故障转储和堆栈跟踪**：粘贴堆栈跟踪并询问可能导致它的原因。

> **关于调试符号的注意事项**：为了获得有意义的堆栈跟踪和调试，请确保您的二进制文件（以及任何链接库）是使用调试符号（`-g` 标志）进行编译的。调试信息通常以 DWARF 格式存储。此外，使用帧指针 (`-fno-omit-frame-pointer`) 进行编译使堆栈跟踪更加可靠，尤其是对于分析工具而言。如果没有这些，堆栈跟踪可能仅显示内存地址或不完整。这对于本机编译的程序（C++、Rust）来说比 Python 或 Java 更重要。

**要记住的限制：**
- LLM可能会产生看似合理但错误的解释
- 他们可能会提出修复建议来掩盖错误而不是修复它
- 始终使用实际调试工具验证建议
- 它们最好作为理解代码的补充，而不是替代

> 这与开发环境讲座中介绍的[通用人工智能编码功能](/2026/development-environment/#ai-powered-development) 不同。在这里，我们专门讨论使用LLM作为调试辅助工具。

# 分析

即使您的代码在功能上表现符合您的预期，但如果它占用了进程中的所有 CPU 或内存，那么这可能还不够好。算法课程通常教授大_O_符号，但不教授如何在程序中查找热点。由于[过早的优化是万恶之源](https://wiki.c2.com/?PrematureOptimization)，您应该了解分析器和监控工具。它们将帮助您了解程序的哪些部分占用了大部分时间和/或资源，以便您可以专注于优化这些部分。

## 时间安排

衡量绩效的最简单方法就是计时。在许多情况下，只需打印代码在两点之间花费的时间就足够了。

但是，挂钟时间可能会产生误导，因为您的计算机可能同时运行其他进程或等待事件发生。 `time` 命令区分 _Real_、_User_ 和 _Sys_ 时间：

- **真实** - 从开始到结束的挂钟时间，包括等待时间
- **用户** - CPU 运行用户代码所花费的时间
- **Sys** - CPU 运行内核代码所花费的时间

```bash
$ time curl https://missing.csail.mit.edu &> /dev/null
real	0m0.272s
user	0m0.079s
sys	    0m0.028s
```

这里的请求花费了近 300 毫秒（实时），但仅占用了 107 毫秒的 CPU 时间（用户 + 系统）。剩下的就等网络了。

## 资源监控

有时，分析程序性能的第一步是了解其实际资源消耗是多少。当资源有限时，程序通常运行缓慢。

- **一般监控**：[`htop`](https://htop.dev/) 是 `top` 的改进版本，可显示当前正在运行的进程的各种统计信息。有用的键绑定：`<F6>` 用于对进程进行排序，`t` 用于显示树层次结构，`h` 用于切换线程。还有 [`btop`](https://github.com/aristocratos/btop) 可以监控_way_更多的东西。

- **I/O 操作**：[`iotop`](https://www.man7.org/linux/man-pages/man8/iotop.8.html) 显示实时 I/O 使用信息。

- **内存使用情况**：[`free`](https://www.man7.org/linux/man-pages/man1/free.1.html) 显示可用和已用内存总量。

- **打开文件**：[`lsof`](https://www.man7.org/linux/man-pages/man8/lsof.8.html) 列出有关进程打开的文件的文件信息。对于检查哪个进程打开了特定文件很有用。

- **网络连接**：[`ss`](https://www.man7.org/linux/man-pages/man8/ss.8.html) 可让您监控网络连接。一个常见的用例是确定哪个进程正在使用给定端口：`ss -tlnp | grep :8080`。

- **网络使用情况**：[`nethogs`](https://github.com/raboof/nethogs) 和 [`iftop`](https://pdw.ex-parrot.com/iftop/) 是很好的交互式 CLI 工具，用于监视每个进程的网络使用情况。

## 可视化性能数据

人类在图表中发现模式的速度比在数字表格中快得多。分析性能时，绘制数据图通常会揭示原始数据中不可见的趋势、峰值和异常情况。

**使数据可绘图**：添加打印或日志语句进行调试时，请考虑格式化输出，以便以后可以轻松绘制图表。 CSV 格式 (`1705012345,42.5`) 的简单时间戳和值比散文句子更容易绘制。 JSON 结构的日志也可以轻松解析和绘制。换句话说，[以整洁的方式]记录您的数据(https://vita.had.co.nz/papers/tidy-data.pdf)。

**使用 gnuplot 快速绘图**：对于简单的命令行绘图，[`gnuplot`](http://www.gnuplot.info/) 可以直接从数据文件生成图形：

```bash
# Plot a simple CSV with timestamp,value
gnuplot -e "set datafile separator ','; plot 'latency.csv' using 1:2 with lines"
```

**使用 matplotlib 和 ggplot2 进行迭代探索**：为了进行更深入的分析，Python 的 [`matplotlib`](https://matplotlib.org/) 和 R 的 [`ggplot2`](https://ggplot2.tidyverse.org/) 支持迭代探索。与一次性绘图不同，这些工具可让您快速切片和转换数据以研究假设。 ggplot2 的分面图特别强大 - 您可以按类别将单个数据集拆分为多个子图（例如，按端点或一天中的时间分面请求延迟），以梳理出本来会隐藏的模式。

**用例示例：**
- 绘制随时间变化的请求延迟可以揭示原始百分位数所掩盖的周期性减速（垃圾收集、cron 作业、流量模式）
- 可视化不断增长的数据结构的插入时间可以暴露算法复杂性问题——当支持数组大小加倍时，向量插入图将显示特征峰值
- 按不同维度（请求类型、用户群体、服务器）划分指标通常会揭示“系统范围”的问题实际上被隔离到一个类别

## CPU 分析器

大多数时候，当人们提到_分析器_时，他们指的是_CPU分析器_。主要有两种类型：

- **跟踪分析器** 记录程序进行的每个函数调用
- **采样分析器**定期探测您的程序（通常每毫秒）并记录程序的堆栈

采样分析器的开销较低，通常更适合生产使用。

### perf：采样分析器

[`perf`](https://www.man7.org/linux/man-pages/man1/perf.1.html) 是标准 Linux 分析器。它可以分析任何程序而无需重新编译：

`perf stat` 让您快速了解时间花在哪里：

```bash
$ perf stat ./slow_program

 Performance counter stats for './slow_program':

         3,210.45 msec task-clock                #    0.998 CPUs utilized
               12      context-switches          #    3.738 /sec
                0      cpu-migrations            #    0.000 /sec
              156      page-faults               #   48.587 /sec
   12,345,678,901      cycles                    #    3.845 GHz
    9,876,543,210      instructions              #    0.80  insn per cycle
    1,234,567,890      branches                  #  384.532 M/sec
       12,345,678      branch-misses             #    1.00% of all branches
```

现实世界程序的探查器输出将包含大量信息。人类是视觉动物，不擅长阅读大量数字。 [火焰图](https://www.brendangregg.com/flamegraphs.html) 是一种可视化形式，可以使分析数据更容易理解。

火焰图显示 Y 轴上的函数调用层次结构以及与 X 轴成比例的所用时间。它们是交互式的——您可以单击以放大程序的特定部分。

[![FlameGraph](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)

要从 `perf` 数据生成火焰图：

```bash
# Record profile
perf record -g ./my_program

# Generate flame graph (requires flamegraph scripts)
perf script | stackcollapse-perf.pl | flamegraph.pl > flamegraph.svg
```

> 考虑使用 [Speedscope](https://www.speedscope.app/) 进行基于 Web 的交互式火焰图查看器，或使用 [Perfetto](https://perfetto.dev/) 进行全面的系统级分析。

### Valgrind 的 Callgrind：跟踪分析器

[`callgrind`](https://valgrind.org/docs/manual/cl-manual.html) 是一个分析工具，用于记录程序的调用历史记录和指令计数。与采样分析器不同，它提供精确的调用计数，并可以显示调用者和被调用者之间的关系：

```bash
# Run with callgrind
valgrind --tool=callgrind ./my_program

# Analyze with callgrind_annotate (text) or kcachegrind (GUI)
callgrind_annotate callgrind.out.<pid>
kcachegrind callgrind.out.<pid>
```

Callgrind 比采样分析器慢，但提供精确的调用计数，并且如果您需要该信息，可以选择模拟缓存行为（使用 `--cache-sim=yes`）。

> 如果您使用特定语言，可能会有更专业的分析器。例如，Python 有 [`cProfile`](https://docs.python.org/3/library/profile.html) 和 [`py-spy`](https://github.com/benfred/py-spy)，Go 有 [`go tool pprof`](https://pkg.go.dev/cmd/pprof)，Rust 有 [`cargo-flamegraph`](https://github.com/flamegraph-rs/flamegraph)。

## 内存分析器

内存分析器可帮助您了解程序如何随时间使用内存并查找内存泄漏。

### 瓦尔格瑞德高地

[`massif`](https://valgrind.org/docs/manual/ms-manual.html) 配置文件堆内存使用情况：

```bash
valgrind --tool=massif ./my_program
ms_print massif.out.<pid>
```

这显示了一段时间内堆的使用情况，有助于识别内存泄漏和过度分配。

> 对于 Python，[`memory-profiler`](https://pypi.org/project/memory-profiler/) 提供逐行内存使用信息。

## 基准测试

当您需要比较不同实现或工具的性能时， [`hyperfine`](https://github.com/sharkdp/hyperfine) 非常适合对命令行程序进行基准测试：

```bash
$ hyperfine --warmup 3 'fd -e jpg' 'find . -iname "*.jpg"'
Benchmark #1: fd -e jpg
  Time (mean ± σ):      51.4 ms ±   2.9 ms    [User: 121.0 ms, System: 160.5 ms]
  Range (min … max):    44.2 ms …  60.1 ms    56 runs

Benchmark #2: find . -iname "*.jpg"
  Time (mean ± σ):      1.126 s ±  0.101 s    [User: 141.1 ms, System: 956.1 ms]
  Range (min … max):    0.975 s …  1.287 s    10 runs

Summary
  'fd -e jpg' ran
   21.89 ± 2.33 times faster than 'find . -iname "*.jpg"'
```

> 对于 Web 开发，浏览器开发人员工具包括出色的分析器。请参阅 [Firefox Profiler](https://profiler.firefox.com/docs/) 和 [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/rendering-tools) 文档。

# 练习

## 调试

1. **调试排序算法**：以下伪代码实现了合并排序，但包含一个错误。使用您选择的语言实现它，然后使用调试器（gdb、lldb、pdb 或 IDE 的调试器）来查找并修复错误。

   ```
   function merge_sort(arr):
       if length(arr) <= 1:
           return arr
       mid = length(arr) / 2
       left = merge_sort(arr[0..mid])
       right = merge_sort(arr[mid..end])
       return merge(left, right)

   function merge(left, right):
       result = []
       i = 0, j = 0
       while i < length(left) AND j < length(right):
           if left[i] <= right[j]:
               append result, left[i]
               i = i + 1
           else:
               append result, right[i]
               j = j + 1
       append remaining elements from left and right
       return result
   ```

   测试向量：`merge_sort([3, 1, 4, 1, 5, 9, 2, 6])` 应返回 `[1, 1, 2, 3, 4, 5, 6, 9]`。使用断点并单步执行合并功能来查找选择了错误元素的位置。

1. 安装 [`rr`](https://rr-project.org/) 并使用反向调试来查找损坏错误。将此程序保存为 `corruption.c`：

   ```c
   #include <stdio.h>

   typedef struct {
       int id;
       int scores[3];
   } Student;

   Student students[2];

   void init() {
       students[0].id = 1001;
       students[0].scores[0] = 85;
       students[0].scores[1] = 92;
       students[0].scores[2] = 78;

       students[1].id = 1002;
       students[1].scores[0] = 90;
       students[1].scores[1] = 88;
       students[1].scores[2] = 95;
   }

   void curve_scores(int student_idx, int curve) {
       for (int i = 0; i < 4; i++) {
           students[student_idx].scores[i] += curve;
       }
   }

   int main() {
       init();
       printf("=== Initial state ===\n");
       printf("Student 0: id=%d\n", students[0].id);
       printf("Student 1: id=%d\n", students[1].id);

       curve_scores(0, 5);

       printf("\n=== After curving ===\n");
       printf("Student 0: id=%d\n", students[0].id);
       printf("Student 1: id=%d\n", students[1].id);

       if (students[1].id != 1002) {
           printf("\nERROR: Student 1's ID was corrupted! Expected 1002, got %d\n",
                  students[1].id);
           return 1;
       }
       return 0;
   }
   ```

使用 `gcc -g corruption.c -o corruption` 编译并运行它。学生 1 的 ID 被损坏，但损坏发生在仅涉及学生 0 的函数中。使用 `rr record ./corruption` 和 `rr replay` 找到罪魁祸首。在 `students[1].id` 上设置一个观察点，并在损坏后使用 `reverse-continue` 来准确查找哪一行代码覆盖了它。

1. 使用 AddressSanitizer 调试内存错误。将其保存为 `uaf.c`：

   ```c
   #include <stdlib.h>
   #include <string.h>
   #include <stdio.h>

   int main() {
       char *greeting = malloc(32);
       strcpy(greeting, "Hello, world!");
       printf("%s\n", greeting);

       free(greeting);

       greeting[0] = 'J';
       printf("%s\n", greeting);

       return 0;
   }
   ```

   首先在没有消毒剂的情况下编译并运行：`gcc uaf.c -o uaf && ./uaf`。它可能看起来有效。现在使用 AddressSanitizer 进行编译：`gcc -fsanitize=address -g uaf.c -o uaf && ./uaf`。阅读错误报告。 ASan 发现了什么错误？修复它所识别的问题。

1. 使用 `strace` (Linux) 或 `dtruss` (macOS) 来跟踪 `ls -l` 等命令所进行的系统调用。它进行什么系统调用？尝试跟踪一个更复杂的程序并查看它打开了哪些文件。

1. 使用 LLM 帮助调试神秘的错误消息。尝试复制编译器错误（尤其是来自 C++ 模板或 Rust）并要求解释和修复。尝试将 `strace` 或地址清理器的一些输出放入其中。

## 分析

1. 使用 `perf stat` 获取您选择的程序的基本性能统计信息。不同的计数器代表什么意思？

1. 带有 `perf record` 的配置文件。将其保存为 `slow.c`：

   ```c
   #include <math.h>
   #include <stdio.h>

   double slow_computation(int n) {
       double result = 0;
       for (int i = 0; i < n; i++) {
           for (int j = 0; j < 1000; j++) {
               result += sin(i * j) * cos(i + j);
           }
       }
       return result;
   }

   int main() {
       double r = 0;
       for (int i = 0; i < 100; i++) {
           r += slow_computation(1000);
       }
       printf("Result: %f\n", r);
       return 0;
   }
   ```

   使用调试符号进行编译：`gcc -g -O2 slow.c -o slow -lm`。运行 `perf record -g ./slow`，然后运行 ​​`perf report` 以查看时间花费在哪里。尝试使用火焰图脚本生成火焰图。

1. 使用 `hyperfine` 对同一任务的两种不同实现进行基准测试（例如，`find` 与 `fd`、`grep` 与 `ripgrep` 或您自己代码的两个版本）。

1. 在运行资源密集型程序时使用 `htop` 监视系统。尝试使用 `taskset` 来限制进程可以使用哪些 CPU：`taskset --cpu-list 0,2 stress -c 3`。为什么`stress`不使用三个CPU？

1. 一个常见的问题是您要侦听的端口已被另一个进程占用。 Learn how to discover that process: First execute `python -m http.server 4444` to start a minimal web server on port 4444. On a separate terminal run `ss -tlnp | grep 4444` to find the process.使用 `kill <PID>` 终止它。
