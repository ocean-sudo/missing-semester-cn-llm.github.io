---
layout: lecture
title: "课程概览与 Shell 入门"
description: >
  了解这门课的动机，并开始上手 Shell。
thumbnail: /static/assets/thumbnails/2026/lec1.png
date: 2026-01-12
ready: true
video:
  aspect: 56.25
  id: MSgoeuMqUmU
---

# 我们是谁？

本课程由 [Anish](https://anish.io/)、[Jon](https://thesquareplanet.com/) 和 [Jose](http://josejg.com/) 共同授课。  
我们都是 MIT 校友，这门课最早也是我们在 MIT 读书时发起的。  
如需联系课程团队，请发邮件到 [missing-semester@mit.edu](mailto:missing-semester@mit.edu)。

这门课是完全公益性质的：我们不收取报酬，也不会将课程商业化。  
所有[课程资料](https://missing.csail.mit.edu/)和[讲座视频](https://www.youtube.com/@MissingSemester)都会免费公开。  
如果你想支持我们，最好的方式是把课程分享给更多人。  
如果你所在的公司、学校或组织在更大范围使用了这套内容，也欢迎来信分享实践反馈。

# 动机

作为计算机专业学生，我们都知道计算机擅长处理重复任务。  
但我们常常忽略：这同样适用于“我们如何使用计算机”本身，而不仅仅是“程序要计算什么”。  
实际上，我们手边有大量工具可以让工作更顺畅、解决更复杂的问题，但很多人只用了其中很小一部分。

这门课的目标就是[补上这一块](/about/)。

我们希望教你把已有工具用到位、把新工具加入自己的工具箱，并且建立持续探索工具（甚至自己造工具）的兴趣。  
这正是我们认为很多计算机课程里“缺失的一学期”。

# 课程结构

本课程为非学分课程，共 9 次讲座，每次约 1 小时，分别聚焦一个[主题](/2026/)。  
各讲之间相对独立，但我们会默认你逐步掌握前面的内容。  
我们提供在线讲义，但课堂演示中的一部分内容可能不在讲义里；讲座录像会持续发布到[线上](https://www.youtube.com/@MissingSemester)。

由于时间紧、内容多，课程节奏会比较密集。每次讲座都配有练习，帮助你按自己的节奏巩固重点。  
课程没有固定 office hour，但你可以在 [OSSU Discord](https://ossu.dev/#community) 的 `#missing-semester-forum` 提问，或直接邮件联系 [missing-semester@mit.edu](mailto:missing-semester@mit.edu)。

受时长限制，我们无法把每个工具讲到专门课程那样细。  
我们会尽量给出继续深入的资料；如果你对某个方向特别感兴趣，欢迎直接联系课程团队。

如果你对课程有反馈，也欢迎随时发邮件给我们。

# 主题 1：Shell

{% comment %}
讲师：乔恩
{% endcomment %}

## Shell 是什么？

如今的计算机有多种命令输入接口：图形界面、语音接口、AR/VR，最近还有 LLM。
这些方式覆盖 80% 的场景没问题，但它们通常会从根本上限制你能做的事情：
你无法点击一个不存在的按钮，也无法对一个还没被设计的语音命令下指令。
要真正发挥计算机的能力，我们还是要回到最基础的文本接口：shell。

几乎所有平台都提供某种 shell，很多平台还提供多个 shell 可选。
虽然它们在细节上可能有所不同，但其核心都是大致相同的
相同：它们允许您运行程序、为其提供输入并进行检查
他们以半结构化的方式输出。

要打开 shell_提示符_（您可以在其中键入命令），您首先需要
_terminal_，这是 shell 的可视化界面。您的设备
可能会附带安装一个，或者您可以公平地安装一个
轻松：

- **Linux：**
  按 `Ctrl + Alt + T` （适用于大多数发行版）。或者搜索
  应用程序菜单中的“终端”。
- **Windows：**
  按 `Win + R`，输入 `cmd` 或 `powershell`，然后按 Enter。
  或者，在“开始”菜单中搜索“终端”或“命令提示符”。
- **macOS：**
  按 `Cmd + Space` 打开 Spotlight，输入“Terminal”，然后按 Enter。
  或者在应用程序 → 实用程序 → 终端中找到它。

在 Linux 和 macOS 上，这通常会打开 Bourne Again SHell，或者
简称“bash”。这是使用最广泛的 shell 之一，其
语法与您在许多其他 shell 中看到的类似。在 Windows 上，
您将受到“batch”或“powershell”shell 的欢迎，具体取决于
你运行了哪个命令。这些是 Windows 特定的，而不是我们将要做的
重点关注这门课，尽管它与大多数内容有类似之处
我们会教学。相反，您需要 [Windows 子系统
Linux](https://docs.microsoft.com/en-us/windows/wsl/) 或 Linux 虚拟
机。

还存在其他 shell，通常比 bash 有许多符合人体工程学的改进
（fish 和 zsh 是最常见的）。虽然这些很受欢迎
（所有的教练都使用一个），它们远没有像
bash，并且依赖于许多相同的概念，所以我们不会关注
本次讲座中的那些。

## 你为什么要关心它？

shell 不仅（通常）比“点击周围”快得多，它
还具有任何人都无法轻易找到的表现力
图形程序。正如我们将看到的，shell 使您能够
以创造性的方式_组合_程序来自动化几乎所有任务。

了解 shell 的使用方式对于导航也非常有用
开源软件的世界（通常随安装一起提供）
需要 shell 的指令），构建持续集成
对于您的软件项目（如[代码质量
讲座](/2026/code-quality/))，以及调试其他程序时的错误
失败。

## 在 shell 中导航

当您启动终端时，您将看到一个通常看起来像的_提示_
有点像这样：

```console
missing:~$
```

这是 shell 的主要文本界面。它告诉你，你
位于机器 `missing` 上并且您的“当前工作目录”，
或者您当前所在的位置是 `~`（“家”的缩写）。 `$` 告诉你
您不是 root 用户（稍后会详细介绍）。出现此提示时您
可以输入_命令_，然后 shell 将解释该命令。的
最基本的命令是执行一个程序：

```console
missing:~$ date
Fri 10 Jan 2020 11:49:31 AM EST
missing:~$
```

在这里，我们执行了 `date` 程序，这（也许并不奇怪）
打印当前日期和时间。然后 shell 会要求我们提供另一个
要执行的命令。我们还可以使用_arguments_执行命令：

```console
missing:~$ echo hello
hello
```

在本例中，我们告诉 shell 使用以下命令执行程序 `echo`
参数 `hello`。 `echo` 程序只是打印出它的参数。
shell 通过用空格分割命令来解析命令，然后
运行第一个字指示的程序，提供每个后续的
word 作为程序可以访问的参数。如果您想提供
包含空格或其他特殊字符的参数（例如，
名为“我的照片”的目录），您可以用 `'` 引用该参数
或 `"` (`"My Photos"`)，或使用 `\` 仅转义相关字符
（`My\ Photos`）。

也许刚开始时最重要的命令是 `man`，
“手册”的缩写。除其他外，`man` 程序可以让您查看
有关系统上任何命令的更多信息。例如，如果
你运行 `man date` ，它会解释 `date` 是什么，以及所有各种
您可以传递它来改变其行为的参数。通常你也可以
通过将 `--help` 作为参数传递给
大多数命令。

> 也建议在 `man` 之外安装并使用 [`tldr`](https://tldr.sh/)。
> 它会在终端里直接给出常见用法示例。LLM 通常也很擅长解释
> 命令的工作方式，以及如何正确调用它们来完成你的目标。

在 `man` 之后，要学习的最重要的命令是 `cd`，或“更改
目录”。该命令实际上内置于 shell 中，而不是
单独的程序（即 `which cd` 将表示“未找到 cd”）。你通过了
一个路径，该路径将成为您当前的工作目录。你会
还可以查看 shell 提示符中反映的工作目录：

```console
missing:~$ cd /bin
missing:/bin$ cd /
missing:/$ cd ~
missing:~$
```

> 请注意，shell 具有自动完成功能，因此您可以经常
> 按 `<TAB>` 可以更快地完成路径！

很多命令如果什么都不操作就对当前工作目录进行操作
否则已指定。如果你不确定自己在哪里，你可以跑步
`pwd` 或打印 `$PWD` 环境变量（使用 `echo $PWD`），两者
其中产生当前工作目录。

当前工作目录也很方便，因为它允许我们
使用相对路径。到目前为止我们看到的所有路径都是
_absolute_ --- 它们以 `/` 开头并提供完整的目录集
需要从文件系统的根导航到某个位置
（`/`）。在实践中，您将更常见地使用相对路径；所以
之所以调用，是因为它们是相对于当前工作目录的。在一个
相对路径（任何不以 `/` 开头的路径），第一个路径
在当前工作目录中查找组件，然后
组件照常遍历。例如：

```console
missing:~$ cd /
missing:/$ cd bin
missing:/bin$
```

每个目录中还存在两个“特殊”组件：
`.` 和 `..`。 `.` 是“本目录”，`..` 是“父目录”
目录”。所以：

```console
missing:~$ cd /
missing:/$ cd bin/../bin/../bin/././../bin/..
missing:/$
```

通常，您可以互换地使用绝对路径和相对路径
命令参数，只需记住您当前的工作目录
是在使用相对的时候！

> 考虑安装和使用
> [`zoxide`](https://github.com/ajeetdsouza/zoxide) 加快您的速度
> `cd`ing --- `z` 会记住你经常访问的路径并让
> 您可以通过更少的打字进行访问。

## shell 中有什么可用的？

但是 shell 如何知道如何查找像 `date` 或 `echo` 这样的程序呢？
如果要求 shell 执行命令，它会查阅 _environment
名为 `$PATH` 的variable_ 列出了 shell 应该使用的目录
当给出命令时搜索程序：

```console
missing:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
missing:~$ which echo
/bin/echo
missing:~$ /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

当我们运行 `echo` 命令时，shell 认为它应该执行
程序 `echo`，然后搜索以 `:` 分隔的列表
`$PATH` 中的目录中具有该名称的文件。当它找到它时，它
运行它（假设该文件是_可执行_；稍后会详细介绍）。我们可以
使用以下命令找出给定程序名执行的文件
`which` 程序。我们还可以通过给出完全绕过 `$PATH`
_path_ 到我们要执行的文件。

这也为我们如何确定_所有_我们正在执行的程序提供了线索
能够在shell中执行：通过列出所有的内容
`$PATH` 上的目录。我们可以通过传递给定的目录路径来做到这一点
到 `ls` 程序，其中列出了文件：

```console
missing:~$ ls /bin
```

> 考虑安装和使用 [`eza`](https://eza.rocks/) 以获取更多信息
> 人性化 `ls`。

在大多数计算机上，这将打印大量程序，但我们只会
这里重点关注一些最重要的问题。首先，一些简单的：

- `cat file`，打印`file`的内容。
- `sort file`，按排序顺序打印出 `file` 的行。
- `uniq file`，从 `file` 中消除连续的重复行。
- `head file` 和 `tail file`，分别打印第一个和
  `file` 的最后几行。

> 考虑安装和使用 [`bat`](https://github.com/sharkdp/bat)
> 超过 `cat` 用于语法突出显示和滚动。

还有 `grep pattern file`，它查找与 `pattern` 匹配的行
在 `file` 中。这个值得更多关注，因为它都_very_
有用，并且具有比人们想象的更广泛的功能。
`pattern` 实际上是一个_正则表达式_，它可以表达非常
复杂的模式——我们将[涵盖
那些](/2026/code-quality/#regular-expressions)
在代码质量讲座中。您还可以指定目录而不是
文件（或将其保留为 `.`）并传递 `-r` 以递归搜索所有
目录中的文件。

> 考虑安装和使用
> [`ripgrep`](https://github.com/BurntSushi/ripgrep) 超过 `grep` 的
> 更快、更人性化（但便携性较差）的替代方案。
> `ripgrep` 也会递归搜索当前工作目录
> 默认！

还有一些非常有用的工具，但稍微复杂一些
界面。其中第一个是 `sed`，它是一个编程文件
编辑。它有自己的编程语言来进行自动编辑
文件，但最常见的用途是：

```console
missing:~$ sed -i 's/pattern/replacement/g' file
```

这会将 `file` 中的 `pattern` 的所有实例替换为 `replacement`。
`-i` 表示我们希望替换发生内联（如
反对不修改 `file` 并打印替换的内容
内容）。 `s/`是sed编程中的表达方式
我们想要进行替换的语言。 `/` 分隔
替换的图案。尾随的 `/g` 表明我们
想要替换每一行中出现的所有内容，而不仅仅是替换
首先。与 `grep` 一样，这里的 `pattern` 是一个正则表达式，
给你显着的表达能力。正则表达式替换
还允许 `replacement` 引用回匹配模式的部分；
我们稍后会看到一个例子。

接下来，我们有 `find`，它可以让您（递归地）查找匹配的文件
某些条件。例如：

```console
missing:~$ find ~/Downloads -type f -name "*.zip" -mtime +30
```

查找下载目录中超过 30 天的 ZIP 文件。

```console
missing:~$ find ~ -type f -size +100M -exec ls -lh {} \;
```

在您的主目录中查找大于 100M 的文件并列出它们。注意事项
`-exec` 接受一个以独立的 `;` 终止的 _command_ （其中
我们需要像空格一样进行转义），其中 `{}` 被替换为 every
通过 `find` 匹配文件路径。

```console
missing:~$ find . -name "*.py" -exec grep -l "TODO" {} \;
```

查找其中包含 TODO 项的任何 `.py` 文件。

`find` 的语法可能有点令人畏惧，但希望这能给出
你会感觉到它有多么有用！

> 考虑安装和使用 [`fd`](https://github.com/sharkdp/fd)
> 而不是 `find` 更人性化（但便携性较差！）
> 经验。

接下来是 `awk`，它与 `sed` 一样，有自己的编程
语言。其中 `sed` 是为编辑文件而构建的， `awk` 是为编辑文件而构建的
解析它们。到目前为止，`awk` 最常见的用途是用于具有
常规语法（如 CSV 文件），您只想提取某些内容
每条记录的部分（即行）：

```console
missing:~$ awk '{print $2}' file
```

打印 `file` 每一行的第二个空格分隔列。
如果添加 `-F,`，它将打印每个的第二个逗号分隔列
线。 `awk` 可以做更多的事情——过滤行、计算聚合、
还有更多——请参阅练习来了解一下。

将这些工具组合在一起，我们可以做一些奇特的事情，例如：

```console
missing:~$ ssh myserver 'journalctl -u sshd -b-1 | grep "Disconnected from"' \
  | sed -E 's/.*Disconnected from .* user (.*) [^ ]+ port.*/\1/' \
  | sort | uniq -c \
  | sort -nk1,1 | tail -n10 \
  | awk '{print $2}' | paste -sd,
postgres,mysql,oracle,dell,ubuntu,inspur,test,admin,user,root
```

这会从远程服务器获取 SSH 日志（我们将在中详细讨论 `ssh`
下一讲），搜索断开消息，提取
每个此类消息中的用户名，并打印前 10 个用户名
以逗号分隔。一切尽在一个命令中！我们将把每个步骤剖析为
一个练习。

## shell 语言（bash）

前面的示例引入了一个新概念：管道 (`|`)。这些让
将一个程序的输出与另一个程序的输入串在一起。
这是可行的，因为大多数命令行程序都会在其上运行
如果没有 `file`，则为“标准输入”（通常击键的位置）
给出了论证。 `|` 采用“标准输出”（通常得到的内容）
打印到您的终端）在 `|` 之前的程序并使其成为
`|` 之后的程序的标准输入。这可以让您
_compose_ shell 程序，它是使 shell 如此出色的一部分
富有成效的工作环境！

事实上，大多数 shell 都实现了完整的编程语言（如 bash），
就像 Python 或 Ruby 一样。它有变量、条件、循环和
功能。当您在 shell 中运行命令时，您实际上是在编写一个
shell 解释的一小段代码。我们不会全部教给你
今天，您会发现有些内容特别有用：

首先，重定向：`>file` 可让您获取程序的标准输出
并将其写入 `file` 而不是您的终端。这使得它更容易
事后分析。 `>>file` 将附加到 `file` 而不是
覆盖它。还有 `<file` 告诉 shell 读取
`file` 而不是从键盘作为程序的标准输入。

> 现在是提及 `tee` 程序的好时机。 `tee` 将打印
> 标准输入到标准输出（就像 `cat`！），但也会 _also_
> 将其写入文件。所以 `verbose cmd | tee verbose.log | grep CRITICAL`
> 将保留完整的详细日志到文件中，同时保留您的
> 终端干净！

接下来，条件语句：`if command1; then command2; command3; fi` 将
执行`command1`，如果没有错误，就会运行
`command2` 和 `command3`。如果您满足以下条件，您还可以拥有 `else` 分支：
希望。用作 `command1` 的最常见命令是 `test`
命令，通常缩写为 `[`，它可以让您评估
诸如“文件是否存在”（`test -f file` / `[ -f file ]`）之类的条件或
“一个字符串是否等于另一个字符串”(`[ "$var" = "string" ]`)。在巴什中，
还有 `[[ ]]`，它是 `test` 的“更安全”内置版本
引用方面的奇怪行为较少。

Bash 还有两种形式的循环：`while` 和 `for`。 `而命令1；做
命令2；命令3；完成` functions just like the equivalent `if`
命令，只不过它会一遍又一遍地重新执行整个过程
只要 `command1` 不出错。 `对于 a b c d 中的 varname；做
命令；完成` executes `命令` four times, each time with `$varname`
设置为 `a`、`b`、`c` 和 `d` 之一。而不是列出项目
明确地说，您经常会使用“命令替换”，例如：

```bash
for i in $(seq 1 10); do
```

这将执行命令 `seq 1 10` （打印从 1 到
10（含 10）），然后将整个 `$()` 替换为该命令的
输出，为您提供 10 次迭代的 for 循环。在旧代码中你会
有时会看到文字反引号（例如``for i in `seq 1 10`; do``）
而不是 `$()`，但您应该强烈选择 `$()` 形式，因为它
可以嵌套。

当您_可以_直接在提示符中编写长 shell 脚本时，您将
通常想将它们写入 `.sh` 文件。例如，
这是一个将循环运行程序直到失败的脚本，
仅打印失败运行的输出，同时给 CPU 带来压力
背景（例如，可用于重现片状测试）：

```bash
#!/bin/bash
set -euo pipefail

# Start CPU stress in background
stress --cpu 8 &
STRESS_PID=$!

# Setup log file
LOGFILE="test_runs_$(date +%s).log"
echo "Logging to $LOGFILE"

# Run tests until one fails
RUN=1
while cargo test my_test > "$LOGFILE" 2>&1; do
    echo "Run $RUN passed"
    ((RUN++))
done

# Cleanup and report
kill $STRESS_PID
echo "Test failed on run $RUN"
echo "Last 20 lines of output:"
tail -n 20 "$LOGFILE"
echo "Full log: $LOGFILE"
```

这里面有很多新东西，我建议你花一些时间
深入研究，因为它们对于写出有用的 shell 脚本非常重要
调用后台作业 (`&`) 来同时运行程序，
更棘手的[shell
重定向](https://www.gnu.org/software/bash/manual/html_node/Redirections.html),
和[算术
扩展](https://www.gnu.org/software/bash/manual/html_node/Arithmetic-Expansion.html)。

值得在程序的前两行上花一点时间
不过。第一个是“shebang”——你会在顶部看到它
除了 shell 脚本之外还有其他文件。当文件以
执行魔法咒语 `#!/path` 时，shell 将启动
在 `/path` 处进行编程，并将文件的内容作为输入传递给它。在
对于 shell 脚本，这意味着传递 shell 的内容
脚本到 `/bin/bash`，但您也可以使用
`/usr/bin/python` 的 shebang 行！

第二行是使 bash “更严格”的一种方法，并减少一些
编写 shell 脚本时的 footgun。 `set` 可能需要很多
参数，但简单地说： `-e` 使得如果任何命令失败，
脚本提前退出； `-u` 使得可以使用未定义的变量
使脚本崩溃而不是仅仅使用空字符串；和`-o
pipelinefail` makes it so that if programs in a `|` 序列失败，
shell 脚本作为一个整体也会提前退出。

> Shell 编程是一个深奥的话题，就像任何编程语言一样
> 是，但请注意：bash 有异常数量的陷阱，就这一点而言
> 有[多个](https://tldp.org/LDP/abs/html/gotchas.html)
> 专门用于[列出它们](https://mywiki.wooledge.org/BashPitfalls) 的网站。
> 我强烈建议大量使用
> 编写它们时[shellcheck](https://www.shellcheck.net/)。LLM是
> 也擅长编写和调试 shell 脚本，以及
> 将它们翻译成“真正的”编程语言（比如Python）
> 它们对于 bash 来说已经变得太笨重了（100 多行）。

# 后续步骤

至此，您已经了解了足够的 shell 来完成任务
基本任务。您应该能够四处导航以查找以下文件
感兴趣并使用大多数程序的基本功能。在接下来的
讲座中，我们将讨论如何执行和自动化更复杂的
使用 shell 和许多方便的命令行程序执行任务
那里。

# 练习

本课程的所有课程都附有一系列练习。
有些给你一个特定的任务去做，而另一些则是开放式的，比如
“尝试使用 X 和 Y 程序”。我们强烈鼓励您尝试一下。

我们还没有为练习编写解决方案。如果你被困在
有什么特别的，请随时在 `#missing-semester-forum` 中发布
在 [Discord](https://ossu.dev/#community) 上或向我们发送电子邮件描述
到目前为止您已尝试过的操作，我们将尽力帮助您。这些
练习也可能作为初始提示起到很好的作用
与LLM对话，您可以交互式地深入了解
主题。这些练习的真正价值在于发现之旅
答案，而不是答案本身。我们鼓励您遵循切线
并在解决这些问题时问“为什么”，而不是仅仅寻找
解决方案的最短路径。

1. 对于本课程，您需要使用 Unix shell，例如 Bash 或 ZSH。如果
如果您使用的是 Linux 或 macOS，则无需执行任何特殊操作。如果你
   在 Windows 上，您需要确保没有运行 cmd.exe 或
   PowerShell；您可以使用 [Windows Subsystem for
   Linux](https://docs.microsoft.com/en-us/windows/wsl/) 或 Linux 虚拟机
   来使用 Unix 风格的命令行工具。确保你正在运行
   一个合适的 shell，你可以尝试命令 `echo $SHELL`。如果它说
   像 `/bin/bash` 或 `/usr/bin/zsh` 这样的东西，这意味着你正在运行
   正确的程序。

1. `-l` 标志到 `ls` 的作用是什么？运行 `ls -l /` 并检查输出。
   每行的前10个字符是什么意思？ （提示：`man ls`）

1. 在命令`find ~/Downloads -type f -name "*.zip" -mtime +30`中，
   `*.zip` 是一个“glob”。什么是全局？创建一个测试目录，其中包含一些
   文件并尝试 `ls *.txt`、`ls file?.txt` 等模式，以及
   `ls {a,b,c}.txt`。参见[模式
   匹配](https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html)
   在 Bash 手册中。

1. `'single quotes'`、`"double quotes"` 和 `$'ANSI quotes'` 有什么区别？
   编写一个命令来回显包含以下内容的字符串
   文字 `$`、`!` 和换行符。参见
   [引用](https://www.gnu.org/software/bash/manual/html_node/Quoting.html)。

1. shell有3个标准流：stdin(0)、stdout(1)、stderr
   （2）。运行 `ls /nonexistent /tmp` 并将 stdout 重定向到一个文件，然后
   stderr 到另一个。您如何将两者重定向到同一个文件？参见
   [重定向](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)。

1. `$?` 保存最后一个命令的退出状态（0 = 成功）。 `&&` 运行
   仅当上一个命令成功时才执行下一个命令； `||` 仅在以下情况下运行它
   前一个失败了。编写一行代码，仅在以下情况下创建 `/tmp/mydir`
   它还不存在。请参阅[退出
   状态](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html)。

1. 为什么 `cd` 必须内置到 shell 本身而不是一个
   独立程序？ （提示：考虑一下子进程可以做什么
   不能影响其父级。）

1. 编写一个脚本，将文件名作为参数 (`$1`) 并检查
   使用 `test -f` 或 `[ -f ... ]` 判断文件是否存在。它应该打印
   根据文件是否存在不同的消息。参见[重击
   有条件的
   表达式](https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html)。

1. 将上一个练习中的脚本保存到文件中（例如 `check.sh`）。
   尝试使用 `./check.sh somefile` 运行它。会发生什么？现在运行
   `chmod +x check.sh` 并重试。为什么需要这一步？ （提示：
   查看 `chmod` 之前和之后的 `ls -l check.sh`。）

1. 如果将 `-x` 添加到脚本中的 `set` 标志中，会发生什么情况？尝试一下
    一个简单的脚本并观察输出。参见[套装
    内置](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html)。

1. 编写一条命令，将文件复制到备份，其中包含今天的日期
    文件名（例如 `notes.txt` → `notes_2026-01-12.txt`）。 （提示：`$(日期
    +%Y-%m-%d)`)。请参阅[命令
    替换](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html)。

1. 修改lecture中的flaky测试脚本以接受测试命令
    作为参数而不是硬编码 `cargo test my_test`。 （提示：`$1`
    或 `$@`)。参见[特别
    参数](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html)。

1. 使用管道查找家中 5 个最常见的文件扩展名
    目录。 （提示：组合 `find`、`grep` 或 `sed` 或 `awk`、`sort`、
    `uniq -c` 和 `head`。）

1. `xargs` 将标准输入中的行转换为命令参数。使用 `find` 和
    `xargs` 一起（不是 `find -exec`）来查找 a 中的所有 `.sh` 文件
    目录并用 `wc -l` 计算每个目录中的行数。奖励：成功
    处理带空格的文件名。 （提示：`-print0` 和 `-0`）。见‘人’
    xargs`。

1. 使用`curl`获取课程网站的HTML
    (`https://missing.csail.mit.edu/`) 并将其通过管道传输到 `grep` 来计算
    列出了许多讲座。 （提示：寻找出现过一次的模式
    每堂课；使用 `curl -s` 来静音进度输出。）

1. [`jq`](https://jqlang.github.io/jq/)是一个强大的处理工具
    JSON 数据。获取样本数据
    `https://microsoftedge.github.io/Demos/json-dummy-data/64KB.json` 与
    `curl` 并使用 `jq` 只提取其版本的人员姓名
    大于 6。（提示：首先通过管道传输到 `jq .` 查看结构；
    然后尝试 `jq '.[] | select(...) | .name'`)

1. `awk` 可以根据列值过滤行并操作输出。
    例如， `awk '$3 ~ /pattern/ {$4=""; print}'` 仅打印行
其中第三列匹配 `pattern`，同时省略第四列
    列。编写一个 `awk` 命令，仅打印第二个所在的行
    列大于 100，交换第一列和第三列。测试
    与： `printf 'a 50 x\nb 150 y\nc 200 z\n'`

1. 剖析讲座中的 SSH 日志管道：每一步的作用是什么？
    然后构建类似的东西来查找最常用的 shell 命令
    `~/.bash_history`（或`~/.zsh_history`）。
