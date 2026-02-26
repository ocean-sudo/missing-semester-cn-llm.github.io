---
layout: lecture
title: "命令行环境"
description: >
  学习命令行程序如何工作，包括输入/输出流、环境变量以及通过 SSH 连接远程机器。
thumbnail: /static/assets/thumbnails/2026/lec2.png
date: 2026-01-13
ready: true
video:
  aspect: 56.25
  id: ccBGsPedE9Q
---

正如我们在上一讲中所介绍的，大多数 shell 不仅仅是启动其他程序的启动器，
但实际上，它们提供了一种完整的编程语言，其中充满了常见的模式和抽象。
然而，与大多数编程语言不同，在 shell 脚本中，一切都是围绕运行程序并让它们简单有效地相互通信而设计的。

特别是，shell 脚本受到_约定_的严格约束。为了让命令行界面 (CLI) 程序在更广泛的 shell 环境中良好运行，需要遵循一些常见模式。
现在，我们将介绍理解命令行程序如何工作所需的许多概念，以及如何使用和配置它们的普遍约定。

# 命令行界面

在大多数编程语言中编写函数类似于：

```
def add(x: int, y: int) -> int:
    return x + y
```

在这里我们可以明确地看到程序的输入和输出。
相比之下，shell 脚本乍一看可能看起来完全不同。

```shell
#!/usr/bin/env bash

if [[ -f $1 ]]; then
    echo "Target file already exists"
    exit 1
else
    if $DEBUG; then
        grep 'error' - | tee $1
    else
        grep 'error' - > $1
    fi
    exit 0
fi
```

为了正确理解像这样的脚本中发生的事情，我们首先需要介绍一些在 shell 程序相互通信或与 shell 环境通信时经常出现的概念：

- 参数
- 流
- 环境变量
- 返回代码
- 信号

## 参数

Shell 程序在执行时会收到一个参数列表。
参数在 shell 中是纯字符串，由程序如何解释它们。
例如，当我们执行`ls -l folder/`时，我们正在执行带有参数`['-l', 'folder/']`的程序`/bin/ls`。

在 shell 脚本中，我们通过特殊的 shell 语法访问它们。
要访问第一个参数，我们访问变量 `$1`，第二个参数 `$2` 等等，直到 `$9`。为了以列表的形式访问所有参数，我们使用 `$@` 并检索参数的数量 `$#`。此外，我们还可以使用 `$0` 访问程序的名称。

对于大多数程序，参数将由 _flags_ 和常规字符串混合组成。
可以识别标志，因为它们前面有破折号 (`-`) 或双破折号 (`--`)。
标志通常是可选的，它们的作用是修改程序的行为。
例如 `ls -l` 更改 `ls` 格式化其输出的方式。

您将看到具有长名称的双破折号标志（如 `--all`）和单破折号标志（如 `-a`），它们通常后面跟着一个字母。
可以在两种格式中指定相同的选项，`ls -a` 和 `ls --all` 是等效的。
单破折号标志通常分组，因此 `ls -l -a` 和 `ls -la` 也是等效的。
标志的顺序通常也不重要， `ls -la` 和 `ls -al` 产生相同的结果。
有些标志非常普遍，当您更加熟悉 shell 环境时，您将直观地使用它们，例如（`--help`、`--verbose`、`--version`）。

> 标志是 shell 约定的第一个很好的例子。 shell 语言不要求我们的程序以这种特定方式使用 `-` 或 `--`。
没有什么可以阻止我们使用语法 `myprogram +myoption myfile` 编写程序，但这会导致混乱，因为我们期望使用破折号。
> 实际上，大多数编程语言都提供 CLI 标志解析库（例如 python 中的 `argparse` 用于使用破折号语法解析参数）。

CLI 程序中的另一个常见约定是程序接受数量可变的相同类型的参数。当以这种方式给出参数时，该命令对每个参数执行相同的操作。

```shell
mkdir src
mkdir docs
# is equivalent to
mkdir src docs
```

这种语法糖乍一看似乎没有必要，但与 _globbing_ 结合使用时它会变得非常强大。
通配符或通配符是 shell 在调用程序之前将扩展的特殊模式。

假设我们要非递归地删除当前文件夹中的所有 .py 文件。根据我们在上一讲中学到的知识，我们可以通过运行来实现这一点

```shell
for file in $(ls | grep -P '\.py$'); do
    rm "$file"
done
```

但我们可以用 `rm *.py` 替换它！

当我们在终端中输入 `rm *.py` 时，shell 不会使用参数 `['*.py']` 调用 `/bin/rm` 程序。
相反，shell 将在当前文件夹中搜索与模式 `*.py` 匹配的文件，其中 `*` 可以匹配任何类型的零个或多个字符的任何字符串。
因此，如果我们的文件夹有 `main.py` 和 `utils.py` 那么 `rm` 程序将接收参数 `['main.py', 'utils.py']`。

您会发现的最常见的通配符是通配符 `*` （零个或多个任何内容）、`?` （恰好是任何内容之一）和花括号。
大括号 `{}` 将逗号分隔的模式列表扩展为多个参数。

在实践中，通过激励性示例可以最好地理解 glob。

```shell
touch folder/{a,b,c}.py
# Will expand to
touch folder/a.py folder/b.py folder/c.py

convert image.{png,jpg}
# Will expand to
convert image.png image.jpg

cp /path/to/project/{setup,build,deploy}.sh /newpath
# Will expand to
cp /path/to/project/setup.sh /path/to/project/build.sh /path/to/project/deploy.sh /newpath

# Globbing techniques can also be combined
mv *{.py,.sh} folder
# Will move all *.py and *.sh files
```

> 某些 shell（例如 zsh）支持更高级的通配形式，例如 `**` ，它将扩展为包含递归路径。所以 `rm **/*.py` 将递归删除所有 .py 文件。


## 流

每当我们执行像这样的程序管道时

```shell
cat myfile | grep -P '\d+' | uniq -c
```

我们看到 `grep` 程序正在与 `cat` 和 `uniq` 程序通信。

这里的一个重要观察是所有三个程序都同时执行。
也就是说，shell 并不是先调用 cat，然后调用 grep，然后调用 uniq。
相反，所有三个程序都被生成，并且 shell 将 cat 的输出连接到 grep 的输入，并将 grep 的输出连接到 uniq 的输入。
使用管道运算符 `|` 时，shell 对从一个程序流向链中下一个程序的数据流进行操作。

我们可以演示这种并发性，管道中的所有命令都会立即启动：

```console
$ (sleep 15 && cat numbers.txt) | grep -P '^\d$' | sort | uniq  &
[1] 12345
$ ps | grep -P '(sleep|cat|grep|sort|uniq)'
  32930 pts/1    00:00:00 sleep
  32931 pts/1    00:00:00 grep
  32932 pts/1    00:00:00 sort
  32933 pts/1    00:00:00 uniq
  32948 pts/1    00:00:00 grep
```

我们可以看到除了 `cat` 之外的所有进程都在立即运行。 shell 会生成所有进程并在其中任何进程完成之前连接它们的流。 `cat` 只有在 sleep 完成后才会开始，并且 `cat` 的输出将被发送到 grep 等等。

每个程序都有一个输入流，标记为 stdin（用于标准输入）。管道传输时，标准输入会自动连接。在脚本中，许多程序接受 `-` 作为文件名，表示“从标准输入读取”：

```shell
# These are equivalent when data comes from a pipe
echo "hello" | grep "hello"
echo "hello" | grep "hello" -
```

类似地，每个程序都有两个输出流：stdout 和 stderr。
标准输出是最常遇到的输出，它用于将程序的输出传输到管道中的下一个命令。
标准错误是一种替代流，旨在供程序报告警告和其他类型的问题，而不会由链中的下一个命令解析该输出。

```console
$ ls /nonexistent
ls: cannot access '/nonexistent': No such file or directory
$ ls /nonexistent | grep "pattern"
ls: cannot access '/nonexistent': No such file or directory
# The error message still appears because stderr is not piped
$ ls /nonexistent 2>/dev/null
# No output - stderr was redirected to /dev/null
```

shell 提供了用于重定向这些流的语法。以下是一些说明性示例。

```shell
# Redirect stdout to a file (overwrite)
echo "hello" > output.txt

# Redirect stdout to a file (append)
echo "world" >> output.txt

# Redirect stderr to a file
ls foobar 2> errors.txt

# Redirect both stdout and stderr to the same file
ls foobar &> all_output.txt

# Redirect stdin from a file
grep "pattern" < input.txt

# Discard output by redirecting to /dev/null
cmd > /dev/null 2>&1
```

另一个体现 Unix 哲学的强大工具是 [`fzf`](https://github.com/junegunn/fzf)，一个模糊查找器。它从标准输入读取行并提供一个交互式界面来过滤和选择：

```console
$ ls | fzf
$ cat ~/.bash_history | fzf
```

`fzf` 可以与许多 shell 操作集成。当我们讨论 shell 定制时，我们会看到它的更多用途。


## 环境变量

要在 bash 中分配变量，我们使用语法 `foo=bar`，然后使用 `$foo` 语法访问变量的值。
请注意，`foo = bar` 是无效语法，因为 shell 会将其解析为使用参数 `['=', 'bar']` 调用程序 `foo`。
在 shell 脚本中，空格字符的作用是执行参数分割。
这种行为可能会令人困惑且难以适应，因此请记住这一点。

Shell 变量没有类型，它们都是字符串。
请注意，在 shell 中编写字符串表达式时，单引号和双引号不可互换。
用 `'` 分隔的字符串是文字字符串，不会扩展变量、执行命令替换或处理转义序列，而 `"` 分隔的字符串则会。

```shell
foo=bar
echo "$foo"
# prints bar
echo '$foo'
# prints $foo
```

为了将命令的输出捕获到变量中，我们使用_命令替换_。
当我们执行
```shell
files=$(ls)
echo "$files" | grep README
echo "$files" | grep ".py"
```
ls 的输出（具体来说是 stdout）被放入变量 `$files` 中，我们稍后可以访问该变量。
`$files` 变量的内容确实包含 ls 输出中的换行符，这就是 `grep` 这样的程序如何知道独立地对每个项目进行操作。

一个鲜为人知的类似功能是_进程替换_，`<( CMD )` 将执行 `CMD` 并将输出放入临时文件中，并用该文件名替换 `<()`。
当命令期望值通过文件而不是 STDIN 传递时，这非常有用。
例如，`diff <(ls src) <(ls docs)` 将显示目录 `src` 和 `docs` 中的文件之间的差异。

每当 shell 程序调用另一个程序时，它都会传递一组通常称为“环境变量”的变量。
在 shell 中，我们可以通过运行 `printenv` 找到当前的环境变量。
要显式传递环境变量，我们可以在命令前面加上变量赋值

> 环境变量通常以全部大写字母书写（例如 `HOME`、`PATH`、`DEBUG`）。这是一个约定，而不是技术要求，但遵循它有助于区分环境变量和通常为小写的本地 shell 变量。

```shell
TZ=Asia/Tokyo date  # prints the current time in Tokyo
echo $TZ  # this will be empty, since TZ was only set for the child command
```

或者，我们可以使用 `export` 内置函数来修改当前环境，因此所有子进程都将继承该变量：

```shell
export DEBUG=1
# All programs from this point onwards will have DEBUG=1 in their environment
bash -c 'echo $DEBUG'
# prints 1
```

要删除变量，请使用 `unset` 内置命令，例如`unset DEBUG`。

> 环境变量是另一个 shell 约定。它们可用于隐式而不是显式地修改许多程序的行为。例如，shell 将 `$HOME` 环境变量设置为当前用户的主文件夹的路径。然后程序可以访问该变量来获取该信息，而不需要显式的 `--home /home/alice`。另一个常见的例子是 `$TZ`，许多程序使用它根据指定的时区来格式化日期和时间。

## 返回码

正如我们之前看到的，shell 程序的主要输出是通过 stdout/stderr 流和文件系统副作用来传达的。

默认情况下，shell 脚本将返回退出代码零。
惯例是零意味着一切顺利，而非零意味着遇到了一些问题。
要返回非零退出代码，我们必须使用内置的 `exit NUM` shell。
我们可以通过访问特殊变量 `$?` 来访问最后运行的命令的返回码。

shell 有布尔运算符 `&&` 和 `||` 分别用于执行 AND 和 OR 运算。
与常规编程语言中遇到的不同，shell 中的语言对程序的返回码进行操作。
这两个都是[短路](https://en.wikipedia.org/wiki/Short-circuit_evaluation) 运算符。
这意味着它们可用于根据先前命令的成功或失败有条件地运行命令，其中成功是根据返回代码是否为零来确定的。一些例子：

```shell
# echo will only run if grep succeeds (finds a match)
grep -q "pattern" file.txt && echo "Pattern found"

# echo will only run if grep fails (no match)
grep -q "pattern" file.txt || echo "Pattern not found"

# true is a shell program that always succeeds
true && echo "This will always print"

# and false is a shell program that always fails
false || echo "This will always print"
```

同样的原则也适用于 `if` 和 `while` 语句，它们都使用返回码来做出决定：

```shell
# if uses the return code of the condition command (0 = true, nonzero = false)
if grep -q "pattern" file.txt; then
    echo "Found"
fi

# while loops continue as long as the command returns 0
while read line; do
    echo "$line"
done < file.txt
```

## 信号

在某些情况下，您需要在程序执行时中断程序，例如，如果命令需要很长时间才能完成。
中断程序的最简单方法是按 `Ctrl-C`，命令可能会停止。
但这实际上是如何运作的以及为什么它有时无法阻止该过程？

```console
$ sleep 100
^C
$
```

> 注意，这里 `^C` 是在终端中键入时 `Ctrl` 的显示方式。

在幕后，这里发生的事情如下：

1.我们按下`Ctrl-C`
2. shell识别特殊字符组合
3. shell进程向`sleep`进程发送SIGINT信号
4.信号中断了`sleep`进程的执行

信号是一种特殊的通信机制。
当进程接收到信号时，它会停止执行，处理该信号，并可能根据信号传递的信息更改执行流程。因此，信号是_软件中断_。


在我们的例子中，当输入 `Ctrl-C` 时，这会提示 shell 向进程传递 `SIGINT` 信号。
下面是一个 Python 程序的最小示例，该程序捕获 `SIGINT` 并忽略它，不再停止。要终止该程序，我们现在可以通过键入 `Ctrl-\` 使用 `SIGQUIT` 信号。

```python
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```

如果我们向该程序发送 `SIGINT` 两次，然后发送 `SIGQUIT`，则会发生以下情况。请注意，`^` 是在终端中键入时显示 `Ctrl` 的方式。

```console
$ python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.py
```

虽然 `SIGINT` 和 `SIGQUIT` 通常都与终端相关的请求相关联，但要求进程正常退出的更通用信号是 `SIGTERM` 信号。
要发送此信号，我们可以使用 [`kill`](https://www.man7.org/linux/man-pages/man1/kill.1.html) 命令，语法为 `kill -TERM <PID>`。

除了杀死进程之外，信号还可以做其他事情。例如，`SIGSTOP` 暂停进程。在终端中，输入 `Ctrl-Z` 将提示 shell 发送 `SIGTSTP` 信号，该信号是 Terminal Stop 的缩写（即终端版本的 `SIGSTOP`）。

然后，我们可以分别使用 [`fg`](https://www.man7.org/linux/man-pages/man1/fg.1p.html) 或 [`bg`](https://man7.org/linux/man-pages/man1/bg.1p.html) 在前台或后台继续暂停的作业。

The [`jobs`](https://www.man7.org/linux/man-pages/man1/jobs.1p.html) command lists the unfinished jobs associated with the current terminal session.
您可以使用它们的 pid 来引用这些作业（您可以使用 [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) 来查找）。
更直观地，您还可以使用百分号后跟其作业编号（由 `jobs` 显示）来引用进程。要引用最后一个后台作业，您可以使用 `$!` 特殊参数。

另一件需要知道的事情是，命令中的 `&` 后缀将在后台运行该命令，给您返回提示，尽管它仍然会使用 shell 的 STDOUT，这可能很烦人（在这种情况下使用 shell 重定向）。同样，要使已运行的程序后台运行，您可以执行 `Ctrl-Z` ，然后执行 `bg` 。


请注意，后台进程仍然是终端的子进程，如果您关闭终端，后台进程就会终止（这将发送另一个信号 `SIGHUP`）。
为了防止这种情况发生，您可以使用 [`nohup`](https://www.man7.org/linux/man-pages/man1/nohup.1.html) （忽略 `SIGHUP` 的包装器）运行程序，或者如果进程已经启动，则使用 `disown` 。
或者，您可以使用终端多路复用器，我们将在下一节中看到。

以下是展示其中一些概念的示例会话。

```
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ kill -SIGHUP %2   # nohup protects from SIGHUP

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000
```

一个特殊的信号是 `SIGKILL` ，因为它不能被进程捕获，并且它总是会立即终止它。但是，它可能会产生严重的副作用，例如留下孤儿进程。

您可以了解有关这些信号和其他信号的更多信息 [此处](https://en.wikipedia.org/wiki/Signal_(IPC)) 或输入 [`man signal`](https://www.man7.org/linux/man-pages/man7/signal.7.html) 或 `kill -l`。

在 shell 脚本中，您可以使用 `trap` 内置函数在收到信号时执行命令。这对于清理操作很有用：

```shell
#!/usr/bin/env bash
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/mytemp.*
}
trap cleanup EXIT  # Run cleanup when script exits
trap cleanup SIGINT SIGTERM  # Also on Ctrl-C or kill
```
{% comment %}
### 用户、文件和权限

最后，程序之间间接通信的另一种方式是使用文件。
为了使程序能够正确读取/写入/删除文件和文件夹，文件权限必须允许该操作。

列出特定文件将给出以下输出

```console
$ ls -l notes.txt
-rw-r--r--  1 alice  users  12693 Jan 11 23:05 notes.txt
```

这里 `ls` 列出了文件的所有者、用户 `alice` 和组 `users`。然后 `rw-r--r--` 是权限的简写符号。
在本例中，文件 `notes.txt` 对用户 alice `rw-` 具有读/写权限，而对该组和文件系统中的其他用户仅具有读权限。

```console
$ ./script.sh
# permission denied
$ chmod +x script.sh
$ ls -l script.sh
-rwxr-xr-x  1 alice  users  3125 Jan 11 23:07 script.sh
$ ./script.sh
```

为了使脚本可执行，必须设置可执行权限，因此我们必须使用 `chmod` （更改模式）程序。
`chmod` 语法虽然直观，但在第一次遇到时并不明显。
如果您像我一样喜欢通过示例进行学习，那么这是 `tldr` 工具的一个很好的用例（请注意，您需要先安装它）。

```console
❯ tldr chmod
  Change the access permissions of a file or directory.
  More information: <https://www.gnu.org/software/coreutils/chmod>.

  Give the [u]ser who owns a file the right to e[x]ecute it:

      chmod u+x path/to/file

  Give the [u]ser rights to [r]ead and [w]rite to a file/directory:

      chmod u+rw path/to/file_or_directory

  Give [a]ll users rights to [r]ead and e[x]ecute:

      chmod a+rx path/to/file
```

运行 `tldr chmod` 查看更多示例，包括递归操作和组权限。

> 您的 shell 可能会显示类似 `command not found: tldr` 的内容。这是因为它是一个更现代的工具，并且大多数系统中都没有预安装。有关如何安装工具的一个很好的参考是 [https://command-not-found.com](https://command-not-found.com) 网站。它包含针对流行操作系统发行版的大量 CLI 工具的说明。

每个程序都作为系统中的特定用户运行。我们可以使用 `whoami` 命令查找用户名，使用 `id -u` 命令查找 UID（用户 ID），它是操作系统与用户关联的整数值。

运行`sudo command`时，`command`以root用户身份运行，可以绕过系统中的大多数权限。
尝试运行 `sudo whoami` 和 `sudo id -u` 以查看输出如何变化（系统可能会提示您输入密码）。
要更改文件或文件夹的所有者，我们使用 `chown` 命令。

您可以了解有关 UNIX 文件权限的更多信息 [此处](https://en.wikipedia.org/wiki/File-system_permissions#Traditional_Unix_permissions)

到目前为止，我们关注的是本地计算机，但是当使用远程服务器时，其中许多技能变得更有价值。

{% endcomment %}

# 远程机器

程序员在日常工作中使用远程服务器已经变得越来越普遍。这里最常用的工具是 SSH（安全 Shell），它将帮助我们连接到远程服务器并提供现在熟悉的 shell 界面。我们使用如下命令连接到服务器：

```bash
ssh alice@server.mit.edu
```

在这里，我们尝试以用户 `alice` 身份在服务器 `server.mit.edu` 中进行 ssh。

`ssh` 的一个经常被忽视的功能是能够以非交互方式运行命令。 `ssh` 正确处理发送标准输入和接收命令的标准输出，因此我们可以将其与其他命令结合起来

```shell
# here ls runs in the remote, and wc runs locally
ssh alice@server ls | wc -l

# here both ls and wc run in the server
ssh alice@server 'ls | wc -l'

```

> 尝试安装 [Mosh](https://mosh.org/) 作为 SSH 替代品，它可以处理断开连接、进入/退出睡眠、更改网络和处理高延迟链接。

为了让 `ssh` 让我们在远程服务器中运行命令，我们需要证明我们有权这样做。
我们可以通过密码或 ssh 密钥来完成此操作。
基于密钥的身份验证利用公钥加密技术向服务器证明客户端拥有秘密私钥，而无需泄露密钥。
基于密钥的身份验证既更方便又更安全，因此您应该更喜欢它。
请注意，私钥（通常是 `~/.ssh/id_rsa`，最近是 `~/.ssh/id_ed25519`）实际上是您的密码，因此请像这样对待它，并且永远不要共享其内容。

要生成一对，您可以运行 [`ssh-keygen`](https://www.man7.org/linux/man-pages/man1/ssh-keygen.1.html)。
```bash
ssh-keygen -a 100 -t ed25519 -f ~/.ssh/id_ed25519
```

如果您曾经配置过使用 SSH 密钥推送到 GitHub，那么您可能已经完成了[此处](https://help.github.com/articles/connecting-to-github-with-ssh/) 概述的步骤，并且已经拥有有效的密钥对。要检查您是否有密码并验证它，您可以运行 `ssh-keygen -y -f /path/to/key`。

在服务器端 `ssh` 将查看 `.ssh/authorized_keys` 以确定应该允许哪些客户端进入。要复制公钥，您可以使用：

```bash
cat .ssh/id_ed25519.pub | ssh alice@remote 'cat >> ~/.ssh/authorized_keys'

# or more simply (if ssh-copy-id is available)

ssh-copy-id -i .ssh/id_ed25519 alice@remote
```

除了运行命令之外，ssh 建立的连接还可用于安全地与服务器传输文件。 [`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html) 是最传统的工具，语法为 `scp path/to/local_file remote_host:path/to/remote_file`。 [`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html) 通过检测本地和远程中的相同文件并防止再次复制它们，对 `scp` 进行了改进。它还提供了对符号链接、权限的更细粒度的控制，并具有额外的功能，例如可以从先前中断的副本中恢复的 `--partial` 标志。 `rsync` 的语法与 `scp` 类似。

SSH 客户端配置位于 `~/.ssh/config` ，它允许我们声明主机并为其设置默认设置。该配置文件不仅可以被 `ssh` 读取，还可以被其他程序读取，例如 `scp`、`rsync`、`mosh` 等。

```bash
Host vm
    User alice
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

# Configs can also take wildcards
Host *.mit.edu
    User alice
```




# 终端多路复用器

使用命令行界面时，您通常会希望一次运行多个任务。
例如，您可能希望并行运行编辑器和程序。
尽管这可以通过打开新的终端窗口来实现，但使用终端多路复用器是一种更通用的解决方案。

像 [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html) 这样的终端多路复用器允许您使用窗格和选项卡多路复用终端窗口，以便您可以有效地与多个 shell 会话进行交互。
此外，终端多路复用器允许您分离当前的终端会话并在稍后的某个时间点重新连接。
因此，终端多路复用器在使用远程计算机时非常方便，因为它避免了使用 `nohup` 和类似技巧。

目前最流行的终端多路复用器是 [`tmux`](https://www.man7.org/linux/man-pages/man1/tmux.1.html)。 `tmux` 具有高度可配置性，通过使用关联的键绑定，您可以创建多个选项卡和窗格并快速浏览它们。

`tmux` 希望您知道它的键绑定，它们都具有 `<C-b> x` 形式，这意味着 (1) 按 `Ctrl+b`，(2) 释放 `Ctrl+b`，然后 (3) 按 `x`。 `tmux` 具有以下对象层次结构：
- **会话** - 会话是具有一个或多个窗口的独立工作区
    + `tmux` 开始一个新会话。
    + `tmux new -s NAME` 以该名称开头。
    + `tmux ls` 列出当前会话
    + 在 `tmux` 中输入 `<C-b> d` 分离当前会话
    + `tmux a` 附上最后一次会议。您可以使用 `-t` 标志来指定哪个

- **Windows** - 相当于编辑器或浏览器中的选项卡，它们在视觉上是同一会话的独立部分
+ `<C-b> c` 创建一个新窗口。要关闭它，你可以终止 shell 执行 `<C-d>`
    + `<C-b> N` 转到第 _N_ 个窗口。请注意它们已编号
    + `<C-b> p` 转到上一个窗口
    + `<C-b> n` 转到下一个窗口
    + `<C-b> ,` 重命名当前窗口
    + `<C-b> w` 列出当前窗口

- **窗格** - 与 vim 分割一样，窗格允许您在同一视觉显示中拥有多个 shell。
    + `<C-b> "` 水平分割当前窗格
    + `<C-b> %` 垂直分割当前窗格
    + `<C-b> <direction>` 移动到指定方向的窗格。这里的方向指的是方向键。
    + `<C-b> z` 切换当前窗格的缩放
    + `<C-b> [` 开始回滚。然后，您可以按 `<space>` 开始选择，并按 `<enter>` 复制该选择。
    + `<C-b> <space>` 循环浏览窗格排列。

> 要了解有关 tmux 的更多信息，请考虑阅读 [this](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) 快速教程和 [this](https://linuxcommand.org/lc3_adv_termmux.php) 更详细的解释。

通过工具包中的 tmux 和 SSH，您将希望让您的环境在任何计算机上都感觉像在家一样。这就是 shell 定制的用武之地。

# 自定义 shell

使用称为 _dotfiles_ 的纯文本文件配置各种命令行程序
（因为文件名以 `.` 开头，例如 `~/.vimrc`，因此它们是
默认情况下隐藏在列出 `ls` 的目录中）。

> 点文件是另一种 shell 约定。前面的点是在列出时“隐藏”它们（是的，另一个约定）。

Shell 是使用此类文件配置的程序的示例之一。启动时，您的 shell 将读取许多文件来加载其配置。
根据 shell 以及您是否启动登录和/或交互式会话，整个过程可能非常复杂。
[这里](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html) 是有关该主题的优秀资源。

对于 `bash`，编辑 `.bashrc` 或 `.bash_profile` 将在大多数系统中工作。
可以通过点文件配置的其他一些工具示例包括：

- `bash` - `~/.bashrc`, `~/.bash_profile`
- `git` - `~/.gitconfig`
- `vim` - `~/.vimrc` 和 `~/.vim` 文件夹
- `ssh` - `~/.ssh/config`
- `tmux` - `~/.tmux.conf`

常见的配置更改是为 shell 添加新位置以查找程序。安装软件时你会遇到这种模式：

```shell
export PATH="$PATH:path/to/append"
```

在这里，我们告诉 shell 将 $PATH 变量的值设置为其当前值加上新路径，并让所有子进程继承这个新的 PATH 值。
这将允许子进程找到位于 `path/to/append` 下的程序。


自定义 shell 通常意味着安装新的命令行工具。包管理器使这一切变得简单。他们负责下载、安装和更新软件。不同的操作系统有不同的包管理器：macOS 使用 [Homebrew](https://brew.sh/)，Ubuntu/Debian 使用 `apt`，Fedora 使用 `dnf`，Arch 使用 `pacman`。我们将在交付代码讲座中更深入地介绍包管理器。

以下是如何在 macOS 上使用 Homebrew 安装两个有用的工具：

```shell
# ripgrep: a faster grep with better defaults
brew install ripgrep

# fd: a faster, user-friendly find
brew install fd
```

安装这些后，您可以使用 `rg` 代替 `grep`，使用 `fd` 代替 `find`。

> **关于 `curl | bash`** 的警告：您经常会看到类似 `curl -fsSL https://example.com/install.sh | bash` 的安装说明。这种模式下载脚本并立即执行，方便但有风险；您正在运行尚未检查的代码。更安全的方法是先下载，查看，然后执行：
> ```shell
>curl -fsSL https://example.com/install.sh -o install.sh
> less install.sh # 查看脚本
> bash 安装.sh
> ```
> 一些安装程序使用稍微更安全的变体：`/bin/bash -c "$(curl -fsSL https://url)"`，它至少确保 bash 解释脚本而不是当前的 shell。

当您尝试运行未安装的命令时，您的 shell 将显示 `command not found`。网站 [command-not-found.com](https://command-not-found.com) 是一个有用的资源，您可以使用它来搜索任何命令，以了解如何跨不同的包管理器和发行版安装它。

另一个有用的工具是 [`tldr`](https://tldr.sh/)，它提供了简化的、以示例为中心的手册页。您无需阅读冗长的文档，而是可以快速查看常见的使用模式：

```console
$ tldr fd
  An alternative to find.
  Aims to be faster and easier to use than find.

  Recursively find files matching a pattern in the current directory:
      fd "pattern"

  Find files that begin with "foo":
      fd "^foo"

  Find files with a specific extension:
      fd --extension txt
```

有时您不需要一个全新的程序，而只需要具有特定标志的现有命令的快捷方式。这就是别名的用武之地。

我们还可以使用内置的 `alias` shell 创建自己的命令别名。
shell 别名是另一个命令的缩写形式，shell 在计算表达式之前会自动替换该命令。
例如，bash 中的别名具有以下结构：

```bash
alias alias_name="command_to_alias arg1 arg2"
```

> 请注意，等号 `=` 周围没有空格，因为 [`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html) 是一个采用单个参数的 shell 命令。

别名有许多方便的功能：

```bash
# Make shorthands for common flags
alias ll="ls -lh"

# Save a lot of typing for common commands
alias gs="git status"
alias gc="git commit"

# Save you from mistyping
alias sl=ls

# Overwrite existing commands for better defaults
alias mv="mv -i"           # -i prompts before overwrite
alias mkdir="mkdir -p"     # -p make parent dirs as needed
alias df="df -h"           # -h prints human readable format

# Alias can be composed
alias la="ls -A"
alias lla="la -l"

# To ignore an alias run it prepended with \
\ls
# Or disable an alias altogether with unalias
unalias la

# To get an alias definition just call it with alias
alias ll
# Will print ll='ls -lh'
```

别名有局限性：它们不能在命令中间接受参数。对于更复杂的行为，您应该使用 shell 函数。

大多数 shell 支持 `Ctrl-R` 进行反向历史搜索。输入 `Ctrl-R` 并开始输入以搜索以前的命令。之前我们介绍了 `fzf` 作为模糊查找器；配置了 fzf 的 shell 集成后，`Ctrl-R` 成为对整个历史记录的交互式模糊搜索，比默认的功能强大得多。

您应该如何组织点文件？它们应该位于自己的文件夹中，
在版本控制下，并使用脚本**符号链接**到位。这有
的好处：

- **轻松安装**：如果您登录到新机器，应用您的
定制只需要一分钟。
- **可移植性**：您的工具在任何地方都可以以相同的方式工作。
- **同步**：您可以在任何地方更新您的点文件并保留它们
同步。
- **更改跟踪**：您可能会维护您的点文件
对于你的整个编程生涯来说，版本历史记录是很好的
长期项目。

您应该在点文件中放入什么？
您可以通过阅读在线文档或
[手册页](https://en.wikipedia.org/wiki/Man_page)。另一个好方法是
在互联网上搜索有关特定程序的博客文章，作者将在其中
告诉您他们喜欢的定制。另一种学习方式
定制就是查看其他人的点文件：你可以找到很多
[点文件
存储库](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories)
在 GitHub 上 --- 查看最受欢迎的一个
[此处](https://github.com/mathiasbynens/dotfiles)（建议您不要盲目
不过复制配置）。
[这里](https://dotfiles.github.io/) 是关于该主题的另一个很好的资源。

课程教员的 dotfiles 都公开在 GitHub 上：[Anish](https://github.com/anishathalye/dotfiles)，
[乔恩](https://github.com/jonhoo/configs),
[何塞](https://github.com/jjgo/dotfiles)。

**框架和插件**也可以改进您的 shell。一些流行的通用框架是 [prezto](https://github.com/sorin-ionescu/prezto) 或 [oh-my-zsh](https://ohmyz.sh/)，以及专注于特定功能的较小插件：

- [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) - 输入时用颜色显示有效/无效命令
- [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) - 在您键入时建议历史记录中的命令
- [zsh-completions](https://github.com/zsh-users/zsh-completions) - 附加完成定义
- [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search) - 类似鱼的历史搜索
- [powerlevel10k](https://github.com/romkatv/powerlevel10k) - 快速、可定制的提示主题

像 [fish](https://fishshell.com/) 这样的 shell 默认包含许多这样的功能。

> 您不需要像 oh-my-zsh 这样的大型框架来获得这些功能。安装单独的插件通常更快，并且给您更多的控制权。大型框架会显着减慢 shell 启动时间，因此请考虑仅安装您实际使用的框架。


# Shell 中的人工智能

将 AI 工具整合到 shell 中的方法有很多。以下是不同集成级别的几个示例：

**命令生成**：像 [`simonw/llm`](https://github.com/simonw/llm) 这样的工具可以帮助从自然语言描述生成 shell 命令：

```console
$ llm cmd "find all python files modified in the last week"
find . -name "*.py" -mtime -7
```

**管道集成**：LLM 可以集成到 shell 管道中以处理和转换数据。当您需要从不一致的格式中提取信息时，它们特别有用，而正则表达式会很痛苦：

```console
$ cat users.txt
Contact: john.doe@example.com
User 'alice_smith' logged in at 3pm
Posted by: @bob_jones on Twitter
Author: Jane Doe (jdoe)
Message from mike_wilson yesterday
Submitted by user: sarah.connor
$ INSTRUCTIONS="Extract just the username from each line, one per line, nothing else"
$ llm "$INSTRUCTIONS" < users.txt
john.doe
alice_smith
bob_jones
jdoe
mike_wilson
sarah.connor
```

请注意我们如何使用 `"$INSTRUCTIONS"` （带引号），因为变量包含空格，并使用 `< users.txt` 将文件内容重定向到标准输入。

**AI shell**：像 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 这样的工具充当元 shell，接受英语命令并将其翻译为 shell 操作、文件编辑和更复杂的多步骤任务。

# 终端模拟器

除了自定义您的 shell 之外，还值得花一些时间来确定您选择的 **终端仿真器** 及其设置。
终端仿真器是一个 GUI 程序，它提供运行 shell 的基于文本的界面。
有很多终端模拟器。

由于您可能会在终端上花费数百至数千小时，因此检查其设置是值得的。您可能想要在终端中修改的一些方面包括：

- 字体选择
- 配色方案
- 键盘快捷键
- 选项卡/窗格支持
- 回滚配置
- 性能（一些较新的终端，如 [Alacritty](https://github.com/alacritty/alacritty) 或 [Ghostty](https://ghostty.org/) 提供 GPU 加速）。



# 练习

## 参数和 Glob

1. 您可能会看到类似 `cmd --flag -- --notaflag` 的命令。 `--` 是一个特殊参数，它告诉程序停止解析标志。 `--` 之后的所有内容都被视为位置参数。为什么这可能有用？尝试运行 `touch -- -myfile`，然后在不运行 `--` 的情况下将其删除。

1. 读取 [`man ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html) 并写入 `ls` 命令，该命令按以下方式列出文件：
    - 包括所有文件，包括隐藏文件
    - 大小以人类可读的格式列出（例如 454M 而不是 454279954）
    - 文件按新近度排序
    - 输出是彩色的

    示例输出如下所示：

    ```
    -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
    drwxr-xr-x   5 user group  160 Jan 14 09:53 .
    -rw-r--r--   1 user group  514 Jan 14 06:42 bar
    -rw-r--r--   1 user group 106M Jan 13 12:12 foo
    drwx------+ 47 user group 1.5K Jan 12 18:08 ..
    ```

{% comment %}
ls -lath --color=auto
{% endcomment %}

1. 进程替换 `<(command)` 允许您像使用文件一样使用命令的输出。使用带有进程替换的 `diff` 来比较 `printenv` 和 `export` 的输出。为什么它们不同？ （提示：尝试 `diff <(printenv | sort) <(export | sort)`）。

## 环境变量

1. 编写 bash 函数 `marco` 和 `polo` 执行以下操作：每当执行 `marco` 时，应以某种方式保存当前工作目录，然后当您执行 `polo` 时，无论您在哪个目录中，`polo` 都应将 `cd` 返回到执行 `marco` 的目录。为了便于调试，您可以在文件 `marco.sh` 中编写代码，并通过执行 `source marco.sh` 将定义（重新）加载到 shell。

{% comment %}
马可（）{
    导出 MARCO=$(pwd)
}

马球（）{
    cd“$马可”
}
{% endcomment %}

## 返回代码

1. 假设您有一个很少失败的命令。为了调试它，您需要捕获它的输出，但是运行失败可能会很耗时。编写一个 bash 脚本，运行以下脚本直至失败，并将其标准输出和错误流捕获到文件中，并在最后打印所有内容。如果您还可以报告脚本失败所需的运行次数，则会加分。

    ```bash
    #!/usr/bin/env bash

    n=$(( RANDOM % 100 ))

    if [[ n -eq 42 ]]; then
       echo "Something went wrong"
       >&2 echo "The error was using magic numbers"
       exit 1
    fi

    echo "Everything went according to plan"
    ```

{% comment %}
#!/usr/bin/env bash

计数=0
直到[[“$？” -ne 0]];
做
  计数=$((计数+1))
  ./random.sh &> out.txt
完成

echo“$count运行后发现错误”
猫出.txt
{% endcomment %}

## 信号和作业控制

1. 在终端中启动 `sleep 10000` 作业，使用 `Ctrl-Z` 将其置于后台，并使用 `bg` 继续执行。现在使用 [`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html) 查找其 pid，并使用 [`pkill`](https://man7.org/linux/man-pages/man1/pgrep.1.html) 杀死它，而无需输入 pid 本身。 （提示：使用 `-af` 标志）。

1. 假设您不想启动一个进程，直到另一个进程完成为止。你会怎样做呢？在本练习中，我们的限制过程将始终是 `sleep 60 &`。实现此目的的一种方法是使用 [`wait`](https://www.man7.org/linux/man-pages/man1/wait.1p.html) 命令。尝试启动 sleep 命令并让 `ls` 等待后台进程完成。

    但是，如果我们在不同的 bash 会话中启动，此策略将会失败，因为 `wait` 仅适用于子进程。我们在注释中没有讨论的一个功能是 `kill` 命令的退出状态在成功时为零，否则为非零。 `kill -0` 不发送信号，但如果进程不存在，则会给出非零退出状态。编写一个名为 `pidwait` 的 bash 函数，它接受 pid 并等待给定进程完成。您应该使用 `sleep` 以避免不必要地浪费 CPU。

## 文件和权限

1.（高级）编写命令或脚本以递归地查找目录中最近修改的文件。更一般地说，您可以按新近度列出所有文件吗？

## 终端多路复用器

1. 按照`tmux` [教程](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) 进行操作，然后学习如何按照[这些步骤](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/) 进行一些基本自定义。

## 别名和点文件

1. 创建一个别名 `dc`，当您输入错误时，该别名会解析为 `cd`。

1. 运行 `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10` 获取最常用的 10 个命令，并考虑为它们编写较短的别名。注意：这适用于 Bash；如果您使用 ZSH，请使用 `history 1` 而不是仅 `history`。

1. 为点文件创建一个文件夹并设置版本控制。

1. 添加至少一个程序的配置，例如你的 shell，进行一些自定义（首先，它可以像通过设置 `$PS1` 自定义 shell 提示符一样简单）。

1. 设置一种在新机器上快速（无需手动操作）安装点文件的方法。这可以像为每个文件调用 `ln -s` 的 shell 脚本一样简单，或者您可以使用[专用实用程序](https://dotfiles.github.io/utilities/)。

1. 在新的虚拟机上测试您的安装脚本。

1. 将所有当前工具配置迁移到 dotfiles 存储库。

1. 在 GitHub 上发布您的点文件。

## 远程计算机 (SSH)

安装 Linux 虚拟机（或使用现有的虚拟机）来进行这些练习。如果您不熟悉虚拟机，请查看 [this](https://hibbard.eu/install-ubuntu-virtual-box/) 安装虚拟机教程。

1. 转到 `~/.ssh/` 并检查那里是否有一对 SSH 密钥。如果没有，则使用 `ssh-keygen -a 100 -t ed25519` 生成它们。建议您使用密码并使用 `ssh-agent`，更多信息[此处](https://www.ssh.com/ssh/agent)。

1. 编辑 `.ssh/config` 以获得如下条目：

    ```bash
    Host vm
        User username_goes_here
        HostName ip_goes_here
        IdentityFile ~/.ssh/id_ed25519
        LocalForward 9999 localhost:8888
    ```

1. 使用 `ssh-copy-id vm` 将 ssh 密钥复制到服务器。

1. 通过执行 `python -m http.server 8888` 在虚拟机中启动 Web 服务器。通过导航到计算机中的 `http://localhost:9999` 来访问 VM Web 服务器。

1. 通过执行 `sudo vim /etc/ssh/sshd_config` 编辑 SSH 服务器配置，并通过编辑 `PasswordAuthentication` 的值禁用密码身份验证。通过编辑 `PermitRootLogin` 的值来禁用 root 登录。使用 `sudo service sshd restart` 重新启动 `ssh` 服务。再次尝试 ssh 登录。

1.（挑战）在虚拟机中安装 [`mosh`](https://mosh.org/) 并建立连接。然后断开服务器/虚拟机的网络适配器。 mosh能正常康复吗？

1.（挑战）研究 `-N` 和 `-f` 标志在 `ssh` 中的作用，并找出实现后台端口转发的命令。
