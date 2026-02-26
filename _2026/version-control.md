---
layout: lecture
title: "版本控制与 Git"
description: >
  学习 Git 的数据模型，以及如何用 Git 进行版本控制与协作。
thumbnail: /static/assets/thumbnails/2026/lec5.png
date: 2026-01-16
ready: true
video:
  aspect: 56.25
  id: 9K8lB61dl3Y
---

版本控制系统（VCS）用于跟踪源代码（以及其他文件集合）的变化。  
它不仅能保留完整历史，也能支持多人协作。  
从抽象层面看，VCS 会把目录状态记录为一系列“快照（snapshots）”，每个快照都描述了某一时刻项目目录的完整状态，并附带作者、提交说明等元数据。

为什么版本控制重要？  
即便你单人开发，它也能帮你回看历史、追踪改动动机、并行维护多个分支。  
在团队场景下，它更是协作基础：你可以查看他人的改动并处理并发开发带来的冲突。

现代 VCS 还能轻松（很多时候是自动地）回答这类问题：

- 谁写了这个模块？
- 该特定文件的该特定行何时被编辑？由谁来？为什么
  被编辑了吗？
- 在过去 1000 次修订中，特定单元测试何时/为何停止
工作？

虽然存在其他 VCS，但 **Git** 是版本控制的事实标准。
这个 [XKCD 漫画](https://xkcd.com/1597/) 捕获了 Git 的声誉：

![xkcd 1597](https://imgs.xkcd.com/comics/git.png)

因为 Git 的接口是一个有漏洞的抽象，所以自上而下地学习 Git（从
及其界面/命令行界面）可能会导致很多混乱。
记住一些命令并将其视为魔法是可能的
咒语，并按照上面漫画中的方法进行操作
错了。

虽然 Git 的界面确实很丑陋，但它的底层设计和想法却很不错。
美丽的。虽然丑陋的界面必须_记住_，但漂亮的设计
可以_理解_。为此，我们对Git进行自下而上的解释，
从它的数据模型开始，然后涵盖命令行界面。
一旦理解了数据模型，就可以更好地理解命令
他们如何操纵底层数据模型。

# Git 的数据模型

Git 的独创性在于其经过深思熟虑的数据模型，它使所有美好的事情成为可能
版本控制的功能，例如维护历史记录、支持分支，以及
促进协作。

## 快照

Git 对某些文件和文件夹集合的历史进行建模
作为一系列快照的顶级目录。在 Git 术语中，文件是
称为“blob”，它只是一堆字节。一个目录被称为
“tree”，它将名称映射到 blob 或树（因此目录可以包含其他
目录）。快照是正在跟踪的顶级树。对于
例如，我们可能有一棵树，如下所示：

```
<root> (tree)
|
+- foo (tree)
|  |
|  + bar.txt (blob, contents = "hello world")
|
+- baz.txt (blob, contents = "git is wonderful")
```

顶层树包含两个元素，一棵树“foo”（它本身包含
一个元素、一个 blob“bar.txt”）和一个 blob“baz.txt”。

## 建模历史：相关快照

版本控制系统应如何关联快照？一个简单的模型是
具有线性历史。历史记录是按时间顺序排列的快照列表。
由于多种原因，Git 不使用这样的简单模型。

在 Git 中，历史记录是快照的有向无环图 (DAG)。那可能
听起来像是一个花哨的数学词，但不要被吓倒。这一切意味着
Git 中的每个快照都引用一组“父级”，即之前的快照
它。这是一群父母而不是单亲（就像在
线性历史），因为快照可能来自多个父项，例如
例如，由于组合（合并）两个并行的开发分支。

Git 将这些快照称为“提交”。可视化提交历史记录可能看起来
像这样的东西：

```
o <-- o <-- o <-- o
            ^
             \
              --- o <-- o
```

在上面的 ASCII 艺术中，`o` 对应于单独的提交（快照）。
箭头指向每个提交的父级（这是一个“先于”关系，
不是“之后”）。第三次提交后，历史记录分为两部分
单独的分支机构。例如，这可能对应于两个单独的功能
并行开发，彼此独立。未来，
这些分支可以合并以创建一个包含两个分支的新快照
这些功能，产生了一个新的历史，看起来像这样，新的
创建的合并提交以粗体显示：

<pre class="highlight">
<代码>
o <-- o <-- o <-- o <---- <strong>o</strong>
            ^            /
             \v
              --- o <-- o
</code>
</前>

Git 中的提交是不可变的。这并不意味着错误不会发生
然而已更正；只是对提交历史记录的“编辑”实际上是
创建全新的提交，并将引用（见下文）更新为指向
到新的。

## 数据模型，作为伪代码

看看用伪代码写下的 Git 数据模型可能会很有启发：

```
// a file is a bunch of bytes
type blob = array<byte>

// a directory contains named files and directories
type tree = map<string, tree | blob>

// a commit has parents, metadata, and the top-level tree
type commit = struct {
    parents: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

这是一个干净、简单的历史模型。

## 对象和内容寻址

“对象”是一个 blob、树或提交：

```
type object = blob | tree | commit
```

在 Git 的数据存储中，所有对象都通过其 [SHA-1
哈希](https://en.wikipedia.org/wiki/SHA-1)。

```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blob、树和提交以这种方式统一：它们都是对象。当
它们引用其他对象，但它们实际上并不_包含_它们
磁盘上的表示形式，但可以通过哈希值对其进行引用。

例如，示例目录结构的树[上面](#snapshots)
（使用 `git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d` 可视化），
看起来像这样：

```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```

树本身包含指向其内容的指针，`baz.txt`（一个 blob）和 `foo`
（一棵树）。如果我们查看对应的哈希所寻址的内容
baz.txt 与 `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`，我们得到
以下：

```
git is wonderful
```

## 参考文献

现在，所有快照都可以通过其 SHA-1 哈希值来识别。这样不方便啊
因为人类不擅长记住 40 个十六进制字符的字符串。

Git 解决这个问题的方法是为 SHA-1 哈希值提供人类可读的名称，称为
“参考文献”。引用是指向提交的指针。与对象不同的是，
不可变，引用是可变的（可以更新以指向新的提交）。
例如， `master` 引用通常指向最新的提交
发展的主要分支。

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

这样，Git 就可以使用人类可读的名称（例如“master”）来指代
历史记录中的特定快照，而不是长的十六进制字符串。

一个细节是，我们经常希望在信息中了解“我们当前所处的位置”的概念。
历史记录，这样当我们拍摄新快照时，我们就知道它与什么相关
（我们如何设置提交的 `parents` 字段）。在 Git 中，“我们
current are”是一个称为“HEAD”的特殊引用。

## 存储库

最后，我们可以（粗略地）定义什么是 Git _repository_：它是数据
`objects` 和 `references`。

在磁盘上，所有 Git 存储都是对象和引用：这就是 Git 的全部内容
数据模型。所有 `git` 命令映射到提交 DAG 的某些操作
添加对象并添加/更新引用。

每当您输入任何命令时，请考虑一下该命令的操作方式
命令正在对底层图形数据结构进行操作。相反，如果你是
尝试对提交 DAG 进行特定类型的更改，例如“丢弃
未提交的更改并使“主”引用指向提交 `5d83f9e`”，有
可能是执行此操作的命令（例如，在本例中，`git checkout master; git reset
--硬 5d83f9e`)。

# 暂存区

这是另一个与数据模型正交的概念，但它是
创建提交的界面。

您可能想象实现上述快照的一种方法是
“创建快照”命令，根据 _current 创建新快照
工作目录的 state_。一些版本控制工具的工作原理是这样的，但是
不是 Git。我们想要干净的快照，并且制作一个可能并不总是理想的
当前状态的快照。例如，想象一个场景，您已经
实现了两个单独的功能，并且您想要创建两个单独的提交，
其中第一个介绍第一个功能，下一个介绍
第二个特点。或者想象一下您正在调试 print 语句的场景
添加到您的所有代码中，并修复了错误；你想提交错误修复
同时丢弃所有打印语句。

Git 通过允许您指定哪些修改来适应这种情况
应通过称为“暂存”的机制包含在下一个快照中
区”。

# Git 命令行界面

为了避免重复信息，我们不会解释下面的命令
这些讲义中有详细说明。查看强烈推荐的【Pro
Git](https://git-scm.com/book/en/v2) 了解更多信息，或观看讲座
视频。

## 基础知识

- `git help <command>`：获取 git 命令的帮助
- `git init`：创建一个新的git repo，数据存储在`.git`目录中
- `git status`：告诉你发生了什么
- `git add <filename>`：将文件添加到暂存区
- `git commit`：创建一个新的提交
    - 写[好的提交消息](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)！
    - 更多的理由来写[好的提交消息](https://chris.beams.io/posts/git-commit/)！
- `git log`：显示历史记录的扁平化日志
- `git log --all --graph --decorate`：将历史可视化为 DAG
- `git diff <filename>`：显示您相对于暂存区域所做的更改
- `git diff <revision> <filename>`：显示快照之间文件的差异
- `git checkout <revision>`：更新 HEAD（如果检出分支则更新当前分支）

## 分支与合并

- `git branch`：显示分支
- `git branch <name>`：创建一个分支
- `git switch <name>`：切换到分支
- `git checkout -b <name>`：创建一个分支并切换到它
    - 与 `git branch <name>; git switch <name>` 相同
- `git merge <revision>`：合并到当前分支
- `git mergetool`：使用精美的工具来帮助解决合并冲突
- `git rebase`：将补丁集重新设置到新的基础上

## 遥控器

- `git remote`：列出遥控器
- `git remote add <name> <url>`：添加遥控器
- `git push <remote> <local branch>:<remote branch>`：发送对象到远程，并更新远程引用
- `git branch --set-upstream-to=<remote>/<remote branch>`：设置本地和远程分支的对应关系
- `git fetch`：从远程检索对象/引用
- `git pull`：与 `git fetch; git merge` 相同
- `git clone`：从远程下载存储库

## 撤消

- `git commit --amend`：编辑提交的内容/消息
- `git reset <file>`：取消暂存文件
- `git restore`：放弃更改

# 高级 Git

- `git config`：Git [高度可定制](https://git-scm.com/docs/git-config)
- `git clone --depth=1`：浅克隆，没有完整的版本历史记录
- `git add -p`：交互式舞台
- `git rebase -i`：交互式变基
- `git blame`：显示谁最后编辑了哪一行
- `git stash`：暂时删除对工作目录的修改
- `git bisect`：二分搜索历史（例如回归）
- `git revert`：创建一个新的提交来反转早期提交的效果
- `git worktree`：同时检查多个分支
- `.gitignore`：[指定](https://git-scm.com/docs/gitignore) 故意未跟踪的文件以忽略

# 杂项

- **GUI**：有很多 [GUI 客户端](https://git-scm.com/downloads/guis)
Git 就在那里。我们个人不使用它们并使用命令行
接口代替。
- **Shell 集成**：将 Git 状态作为您的一部分非常方便
shell 提示符 ([zsh](https://github.com/olivierverdier/zsh-git-prompt),
[bash](https://github.com/magicmonty/bash-git-prompt))。经常包含在
像 [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh) 这样的框架。
- **编辑器集成**：与上面类似，与许多
特点。 [fugitive.vim](https://github.com/tpope/vim-fugitive) 是标准
一份给 Vim 的。
- **工作流程**：我们教您数据模型，以及一些基本命令；我们
没有告诉您在处理大型项目时应遵循哪些实践（并且
有[很多](https://nvie.com/posts/a-successful-git-branching-model/)
[不同](https://www.endoflineblog.com/gitflow-considered-harmful)
[方法](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow))。
- **GitHub**：Git 不是 GitHub。 GitHub 有特定的贡献代码方式
到其他项目，称为 [pull
请求](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests)。
- **其他 Git 提供商**：GitHub 并不特别：有很多 Git 存储库
主机，例如 [GitLab](https://about.gitlab.com/) 和
[BitBucket](https://bitbucket.org/)。

# 资源

- [Pro Git](https://git-scm.com/book/en/v2) **强烈推荐阅读**。
阅读第 1--5 章应该可以教会您使用 Git 所需的大部分内容
既然您已经了解了数据模型，那么就可以熟练地进行操作了。后面的章节有
一些有趣的、先进的材料。
- [Oh Shit, Git!?!](https://ohshitgit.com/) 是关于如何恢复的简短指南
一些常见的 Git 错误。
- [计算机版 Git
科学家](https://eagain.net/articles/git-for-computer-scientists/) 是
Git 数据模型的简短解释，伪代码更少，更花哨
图表比这些讲义更重要。
- [Git 自下而上](https://jwiegley.github.io/git-from-the-bottom-up/)
除了数据之外，还详细解释了 Git 的实现细节
模型，供好奇的人使用。
- 【如何简单解释git
字](https://smusamashah.github.io/blog/2017/10/14/explain-git-in-simple-words)
- [学习 Git 分支](https://learngitbranching.js.org/) 是基于浏览器的
教你 Git 的游戏。

# 练习

1. 如果您过去没有任何 Git 经验，请尝试阅读第一篇
   [Pro Git](https://git-scm.com/book/en/v2) 的几章或通过
   类似 [学习 Git 分支](https://learngitbranching.js.org/) 的教程。作为
   当你正在处理它时，将 Git 命令与数据模型联系起来。
1. 克隆[课程网站仓库](https://github.com/missing-semester/missing-semester)。
    1. 通过将其可视化为图表来探索版本历史记录。
    1. 最后修改 `README.md` 的人是谁？（提示：给 `git log` 传入一个参数）
    1. 与上次修改相关的提交消息是什么
       `_config.yml` 的 `collections:` 行？ （提示：使用 `git blame` 和 `git
       显示`）。
1. 学习 Git 时常见的一个错误是提交大文件
   不由 Git 管理或添加敏感信息。尝试添加一个文件到
一个存储库，进行一些提交，然后从 _history_ 中删除该文件
   （不仅仅是最新的提交）。您可能想看看
   [此](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)。
1. 从 GitHub 克隆一些仓库，并修改其中一个已有文件。
   执行 `git stash` 会发生什么？运行 `git log --all --oneline` 会看到什么？
   再执行 `git stash pop` 撤销之前的 stash 操作。
   这在什么情况下有用？
1. 与许多命令行工具一样，Git 提供了配置文件（或点文件）
   称为 `~/.gitconfig`。在 `~/.gitconfig` 中创建一个别名，以便当您
   运行 `git graph` ，你会得到 `git log --all --graph --decorate 的输出
   --oneline`。您可以直接执行此操作
   [编辑](https://git-scm.com/docs/git-config#Documentation/git-config.txt-alias)
   `~/.gitconfig` 文件，或者您可以使用 `git config` 命令添加
   别名。可以找到关于git别名的信息
   [此处](https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases)。
1. 运行 `git config --global core.excludesfile ~/.gitignore_global` 后，
   你可以在 `~/.gitignore_global` 中定义全局忽略规则。这会设置
   Git 将使用的全局忽略文件的位置，但您仍然需要
   在该路径手动创建文件。将全局 gitignore 文件设置为
   忽略特定于操作系统或特定于编辑器的临时文件，例如 `.DS_Store`。
1. fork 这门课程的[网站仓库](https://github.com/missing-semester/missing-semester)，找一个拼写错误
   或您可以进行的其他一些改进，并在 GitHub 上提交拉取请求
   （您可能想查看 [this](https://github.com/firstcontributions/first-contributions)）。
请仅提交有用的 PR（请不要向我们发送垃圾邮件！）。如果你
   找不到要改进的地方，您可以跳过此练习。
1. 通过模拟协作场景练习解决合并冲突：
    1. 使用 `git init` 创建一个新存储库并创建一个名为
       `recipe.txt` 有几行（例如，一个简单的食谱）。
    1.提交，然后创建两个分支：`git branch salty`和`git分支
       甜甜`。
    1. 在 `salty` 分支中，修改一行（例如，将“1 cup Sugar”更改为“1
       杯盐”）并提交。
    1. 在 `sweet` 分支中，对同一行进行不同的修改（例如，更改“1
       一杯糖”改为“2 杯糖”）并提交。
    1. 现在切换到 `master` 并尝试 `git merge salty`，然后 `git merge
       sweet`. What happens? Look at the contents of `recipe.txt` - 做什么
       `<<<<<<<`、`=======` 和 `>>>>>>>` 标记的含义是什么？
    1.通过编辑文件来保留你想要的内容来解决冲突，
       删除冲突标记，并使用 `git add` 完成合并
       和 `git commit` （或 `git merge --continue`）。或者，尝试使用
       `git mergetool` 解决与图形或
       基于终端的合并工具。
    1. 使用 `git log --graph --oneline` 可视化刚刚的合并历史记录
       创建的。
