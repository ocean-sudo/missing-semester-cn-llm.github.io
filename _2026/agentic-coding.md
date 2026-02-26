---
layout: lecture
title: "智能体编程"
description: >
  学习如何高效使用 AI 编码智能体完成软件开发任务。
thumbnail: /static/assets/thumbnails/2026/lec7.png
date: 2026-01-21
ready: true
video:
  aspect: 56.25
  id: sTdz6PZoAnw
---

编码智能体是对话式人工智能模型，可以访问读/写文件、网络搜索和调用 shell 命令等工具。它们要么存在于 IDE 中，要么存在于独立的命令行或 GUI 工具中。编码智能体是高度自主且功能强大的工具，可实现多种用例。

本讲座以[开发环境和工具](/2026/development-environment/) 讲座中的人工智能驱动的开发材料为基础。作为一个快速演示，让我们继续使用 [AI 支持的开发](/2026/development-environment/#ai-powered-development) 部分中的示例：

```python
from urllib.request import urlopen

def download_contents(url: str) -> str:
    with urlopen(url) as response:
        return response.read().decode('utf-8')

def extract(content: str) -> list[str]:
    import re
    pattern = r'\[.*?\]\((.*?)\)'
    return re.findall(pattern, content)

print(extract(download_contents("https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/development-environment.md")))
```

我们可以尝试提示编码智能体执行以下任务：

```
Turn this into a proper command-line program, with argparse for argument parsing. Add type annotations, and make sure the program passes type checking.
```

代理将读取文件以理解它，然后进行一些编辑，最后调用类型检查器以确保类型注释正确。如果它犯了一个错误，导致类型检查失败，它可能会迭代，尽管这是一个简单的任务，因此不太可能发生。由于编码智能体可以访问可能有害的工具，因此默认情况下，代理工具会提示用户确认工具调用。

> 如果编码智能体犯了错误 --- 例如，如果您在 `$PATH` 上直接提供了 `mypy` 二进制文件，但代理尝试调用 `python -m mypy` --- 您可以向其提供文本反馈以帮助其纠正。

编码智能体支持多轮交互，因此您可以通过与代理的来回对话来迭代工作。如果代理走错了路，您甚至可以打断它。一个有用的思维模型可能是实习生经理的思维模型：实习生会做具体的工作，但需要指导，偶尔会做错事并需要纠正。

> 要获得更具说明性的演示，请尝试要求代理作为后续操作来运行生成的脚本。观察输出，并尝试要求它进行更改（例如，要求它仅包含绝对 URL）。

# AI 模型和代理如何工作

全面解释现代[大语言模型 (LLM)](https://en.wikipedia.org/wiki/Large_language_model) 和代理运行框架等基础设施的内部运作超出了本课程的范围。然而，对一些关键思想有一个高层次的理解有助于有效地使用这种前沿技术并理解其局限性。

LLM 可以被视为对给定提示字符串（输入）的完成字符串（输出）的概率分布进行建模。 LLM 推理（例如，当您向对话式聊天应用程序提供查询时会发生什么）从该概率分布_样本_。 LLM 有一个固定的_上下文窗口_，即输入和输出字符串的最大长度。

{% comment %}
> 在数学符号中，LLM 对提示 $x$ 条件下完成 $y$ 的概率分布 $\pi_\theta$ 进行建模，并且我们从该分布中采样：$\hat{y} \sim \pi_\theta(\cdot \mid x)$。
{% endcomment %}

对话式聊天和编码智能体等人工智能工具建立在这个原语之上。对于多回合交互，聊天应用程序和代理使用回合标记，并在每次出现新用户提示时提供整个对话历史记录作为提示字符串，每个用户提示调用一次 LLM 推理。对于工具调用代理，工具将某些 LLM 输出解释为调用工具的请求，并且工具将工具回调的结果作为提示字符串的一部分提供给模型（因此每次有工具调用/响应时，LLM 推理都会再次运行）。工具调用代理的核心概念可以[用 200 行代码实现](https://www.mihaileric.com/The-Emperor-Has-No-Clothes/)。

## 隐私

大多数标准配置的人工智能编码工具都会将大量数据发送到云端。有时，运行框架在本地运行，而 LLM 推理在云中运行，有时更多的软件在云中运行（例如，服务提供商可能会有效地获得整个存储库的副本以及您与 AI 工具的所有交互）。

有一些开源 AI 编码工具和开源 LLM 相当不错（尽管不如专有模型），但目前对于大多数用户来说，由于硬件限制，在本地运行前沿的开放 LLM 是不可行的。

# 用例

编码智能体可以帮助完成各种任务。一些例子：

- **实现新功能。** 如上例所示，您可以要求编码智能体实现某个功能。在这一点上，给出一个好的规范更多的是一门艺术，而不是一门科学。您希望代理的输入具有足够的描述性，以便代理执行您希望它执行的操作（至少朝着正确的方向前进，以便您可以迭代），但不要过度描述，以免您自己做太多工作。测试驱动开发可能特别有效：编写测试（或使用编码智能体帮助您编写测试），审核它们以确保它们捕获您想要的内容，然后要求编码智能体实现该功能。模型在不断改进，因此您必须保持对模型功能的最新直觉。
    > 我们使用 Claude Code 来[实现](https://github.com/missing-semester/missing-semester/pull/345) 这些 Tufte 风格的旁注。
{%- comment %}
无需演示，因为讲座的介绍只是添加新功能的小型演示。
{% endcomment %}
- **修复错误。** 如果您的编译器、linter、类型检查器或测试出现错误，您可以要求您的代理更正它们，例如使用“使用 mypy 修复问题”之类的提示。当您可以将模型纳入反馈循环时，编码模型特别有效，因此请尝试进行设置，以便模型可以直接运行失败检查，这将使其自主迭代。如果这不切实际，您可以手动向模型提供反馈。
    > 在提交缺失学期存储库的 [f552b55](https://github.com/missing-semester/missing-semester/commit/f552b5523462b22b8893a8404d2110c4e59613dd) 时，我们提示 Claude Code“查看代理编码讲座中的打字错误和语法问题”，并随后要求其修复发现的问题，这些问题是在 [f1e1c41](https://github.com/missing-semester/missing-semester/commit/f1e1c417adba6b4149f7eef91ff5624de40dc637) 中提交的。
{%- comment %}
在 https://github.com/anishathalye/dotbot/commit/cef40c902ef0f52f484153413142b5154bbc5e99 中演示修复该错误的编码智能体。

编写失败的测试来演示错误，然后要求代理修复。在分支 demo-bugfix 中准备。

可以使用以下命令运行失败的测试：

    孵化测试测试/test_cli.py::test_issue_357

可以提示编码智能体：

    我为一个错误编写了一个失败的测试，您可以使用 `hatch test tests/test_cli.py::test_issue_357` 重现它。修复错误。

让它提交更改。
{% endcomment %}
- **重构。** 您可以使用编码智能体以各种方式重构代码，从简单的任务（例如重命名方法）（这种重构也受 [代码智能](/2026/development-environment/#code-intelligence-and-language-servers) 支持）到更复杂的任务（例如将功能分解为单独的模块）。
> 我们使用 Claude Code 将(https://github.com/missing-semester/missing-semester/pull/344) 代理编码[split]到它自己的讲座中。
{%- comment %}
显示 Missing Semester 中的用法，指出代理确实犯了一些错误。
{% endcomment %}
- **代码审查。** 您可以要求编码智能体审查代码。您可以为他们提供基本指导，例如“查看我尚未提交的最新更改”。如果您想查看拉取请求并且您的编码智能体支持网络获取，或者您安装了 [GitHub CLI](https://cli.github.com/) 等命令行工具，您甚至可以要求编码智能体“查看拉取请求 {link}”，它会从那里处理它。
{%- comment %}
在 Porcupine 存储库中，提示代理：

    查看此 PR：https://github.com/anishathalye/porcupine/pull/39
{% endcomment %}
- **代码理解。** 您可以向编码智能体询问有关代码库的问题，这对于入门特别有帮助。
{%- comment %}
在缺少学期的存储库中尝试的一些提示：

    我如何在本地运行该网站？

    社交预览卡是如何实施的？
{% endcomment %}
- **作为 shell。** 您可以要求编码智能体使用特定工具来解决任务，这样您就可以使用自然语言调用 shell 命令，例如“使用 find 命令查找所有超过 30 天的文件”或“使用 mogrify 将所有 jpg 大小调整为原始大小的 50%”。
{%- comment %}
在 Dotbot 存储库中，提示代理：

    使用 ag 命令查找所有 Python 重命名导入
{% endcomment %}
- **Vibe 编码。** 代理足够强大，您无需自己编写一行代码即可实现某些应用程序。
    > [这里是一个示例](https://github.com/cleanlab/office-presence-dashboard) 是一个真实世界项目，由一位讲师进行了 vivi 编码。
{%- comment %}
在缺少学期的存储库中，提示代理：

    让这个网站看起来复古。
{% endcomment %}

# 高级代理

在这里，我们简要概述了编码智能体的一些更高级的使用模式和功能。

- **可重复使用的提示。** 创建可重复使用的提示或模板。例如，您可以编写详细的提示以特定方式进行代码审查，并将其保存为可重用的提示。
    > 代理工具发展迅速。在某些工具中，不推荐将可重复使用的提示作为独立功能。例如，在 Codex 和 Claude Code 中，它们被[技能](https://code.claude.com/docs/en/skills)[包含](https://developers.openai.com/codex/custom-prompts)。
- **并行代理。** 编码智能体可能会很慢：您可以提示代理，它可能会在问题上工作数十分钟。您可以同时运行多个代理副本，要么处理同一任务（LLM 是随机的，因此多次运行同一事物并采取最佳解决方案可能会有所帮助），要么处理不同的任务（例如，同时实现两个不重叠的功能）。为了防止不同代理的更改相互干扰，您可以使用 [git worktrees](https://git-scm.com/docs/git-worktree)，我们在 [版本控制](/2026/version-control/) 讲座中对此进行了介绍。
- **MCP。** MCP 代表_模型上下文协议_，是一种开放协议，可用于将编码智能体与工具连接起来。例如，这个 [Notion MCP 服务器](https://github.com/makenotion/notion-mcp-server) 可以让您的代理读取/写入 Notion 文档，从而启用诸如“读取 {Notion doc} 中链接的规范，起草实施计划作为 Notion 中的新页面，然后实现原型”之类的用例。为了发现 MCP，您可以使用 [Pulse](https://www.pulsemcp.com/servers) 和 [Glama](https://glama.ai/mcp/servers) 等目录。
- **上下文管理。** 正如我们[上文](#how-ai-models-and-agents-work)所指出的，编码智能体底层的 LLM 具有有限的_上下文窗口_。高效使用编码智能体的关键是做好上下文管理：既要确保代理拿到完成任务所需的信息，也要避免无关上下文导致窗口溢出或模型性能下降（即便未溢出，上下文过大也常会降低效果）。代理工具会自动提供和管理一部分上下文，但用户仍然需要主动把控。
    - **清除上下文窗口。** 最基本的控制，编码智能体支持清除上下文窗口（开始新的对话），您应该为不相关的查询执行此操作。
    - **倒带对话。** 某些编码智能体支持撤消对话历史记录中的步骤。在“撤消”更有意义的情况下，这可以更有效地管理上下文，而不是给出引导代理走向不同方向的后续消息。
{%- comment %}
制作一个快速演示。
{% endcomment %}
- **压缩。** 为了实现无限长度的对话，编码智能体支持上下文_压缩_：如果对话历史记录变得太长，它们将自动调用LLM来总结对话的前缀，并用摘要替换对话历史记录。一些代理允许用户在需要时调用压缩。
{%- comment %}
在 Claude 代码中显示 `/compact`，显示完整摘要。
{% endcomment %}
    - **llms.txt.** `/llms.txt` 文件是建议的[标准](https://llmstxt.org/) 位置，用于供LLM在推理时使用的文档。产品（例如，[cursor.com/llms.txt](https://cursor.com/llms.txt)）、软件库（例如，[ai.pydantic.dev/llms.txt](https://ai.pydantic.dev/llms.txt)）和 API（例如，[apify.com/llms.txt](https://apify.com/llms.txt)）可能具有便于开发的 `llms.txt` 文件。此类文档的每个标记的信息密度更高，因此它们比要求编码智能体获取和读取 HTML 页面更具上下文效率。当编码智能体没有关于您尝试使用的依赖项的内置知识时（例如，因为它是在 LLM 知识截止后发布的），外部文档会很方便。
{%- comment %}
在空存储库中进行并排比较（在桌面或其他一些独立的位置，其中运行 `git init`）：

    使用 semlib 在 demo.py 中编写一个单文件 Python 程序示例，根据“Ilya Sutskever”、“Soumith Chintala”和“Donald Knuth”作为 AI 研究人员的名气对他们进行排序。

    使用 semlib 在 demo.py 中编写一个单文件 Python 程序示例，根据“Ilya Sutskever”、“Soumith Chintala”和“Donald Knuth”作为 AI 研究人员的名气对他们进行排序。请参阅 https://semlib.anish.io/llms.txt。单击 llms.txt 文件中链接的任何页面的 Markdown 版本的链接。

不知道为什么代理默认情况下不这样做。您可能会将最后一句话放在 CLAUDE.md 文件中。
{% endcomment %}
    - **AGENTS.md.** 大多数编码智能体支持 [AGENTS.md](https://agents.md/) 或类似的（例如，Claude Code 查找 `CLAUDE.md`）作为编码智能体的自述文件。当代理启动时，它会使用 `AGENTS.md` 的全部内容预先填充上下文。您可以使用它向代理提供跨会话常见的建议（例如，指示它在进行代码更改后始终运行类型检查器，解释如何运行单元测试，或提供代理可以浏览的第三方文档的链接）。某些编码智能体可以自动生成此文件（例如 Claude Code 中的 `/init` 命令）。有关 `AGENTS.md` 的真实示例，请参阅[此处](https://github.com/pydantic/pydantic-ai/blob/main/CLAUDE.md)。
{%- comment %}
Dotbot 示例 CLAUDE.md 包含 @DEVELOPMENT.md 并表示在对 Python 代码进行任何更改后始终运行类型检查器和代码格式化程序。

示例提示，来自 master：

    删除“--version”命令行标志。

出于演示目的，这会很快。
{% endcomment %}
    - **技能。** `AGENTS.md` 中的内容始终会完整加载到代理的上下文窗口中。 _技能_添加一级间接以避免上下文膨胀：您可以向代理提供技能列表以及描述，并且代理可以根据需要“打开”技能（将其加载到其上下文窗口中）。
    - **子代理。** 某些编码智能体允许您定义子代理，它们是特定于任务的工作流程的代理。顶级编码智能体可以调用​​子代理来完成特定任务，这使得顶级代理和子代理能够更有效地管理上下文。顶级代理的上下文不会因为子代理看到的所有内容而变得臃肿，并且子代理可以仅获取其任务所需的上下文。举一个例子，一些编码智能体将网络研究实现为子代理：顶级代理将向子代理提出查询，子代理将运行网络搜索、检索各个网页、分析它们，并向顶级代理提供查询的答案。这样，顶级代理的上下文就不会因为所有检索到的网页的完整内容而变得臃肿，并且子代理的上下文中也不会包含顶级代理的其余对话历史记录。

对于许多需要编写提示的高级功能（例如技能或子代理），您可以使用LLM来帮助您入门。一些编码智能体甚至内置了对此的支持。例如，Claude Code 可以通过简短的提示生成子代理（调用 `/agents` 并创建新代理）。尝试使用以下提示创建子代理：

```
A Python code checking agent that uses `mypy` and `ruff` to type-check, lint, and format *check* any files that have been modified from the last git commit.
```

然后，您可以使用顶级代理通过诸如“使用代码检查器子代理”之类的消息显式调用子代理。您还可以让顶级代理在适当的时候自动调用子代理，例如在修改任何 Python 文件后。

# 需要注意什么

人工智能工具可能会犯错误。它们建立在 LLM 的基础上，而 LLM 只是概率下一个标记预测模型。它们并不像人类那样“聪明”。检查 AI 输出的正确性和安全错误。有时验证代码可能比自己编写代码更困难；对于关键代码，请考虑手动编写。人工智能可以钻进兔子洞并试图给你加油；注意调试螺旋。不要把人工智能当作拐杖，警惕过度依赖或理解浅薄。人工智能仍然无法完成大量的编程任务。计算思维仍然有价值。

# 推荐软件

许多 IDE / AI 编码扩展都包含编码智能体（请参阅[开发环境讲座](/2026/development-environment/) 中的建议）。其他流行的编码智能体包括 Anthropic 的 [Claude Code](https://www.claude.com/product/claude-code)、OpenAI 的 [Codex](https://openai.com/codex/) 以及 [opencode](https://github.com/anomalyco/opencode) 等开源代理。

# 练习

1. 通过执行四次相同的编程任务，比较手动编码、使用人工智能自动完成、内联聊天和代理的体验。最好的候选者是您已经在处理的项目中的小型功能。如果您正在寻找其他想法，您可以考虑在 GitHub 上的开源项目中完成“好第一期”风格的任务，或者 [Advent of Code](https://adventofcode.com/) 或 [LeetCode](https://leetcode.com/) 问题。
1. 使用人工智能编码智能体来导航不熟悉的代码库。最好在想要调试或向您真正关心的项目添加新功能的情况下完成此操作。如果您没有想到，请尝试使用 AI 代理来了解安全相关功能如何在 [opencode](https://github.com/anomalyco/opencode) 代理中工作。
1. Vibe 从头开始编写一个小应用程序。不要手写一行代码。
1. 对于您选择的编码智能体，创建并测试 `AGENTS.md`（或与您选择的代理类似的工具，例如 `CLAUDE.md`）、一项技能（例如，[Claude Code 中的技能](https://code.claude.com/docs/en/skills) 或 [Codex 中的技能](https://developers.openai.com/codex/skills/)）和一个子代理（例如，[Claude Code 中的子代理](https://code.claude.com/docs/en/sub-agents)）。考虑一下您何时想使用其中一种而不是另一种。请注意，您选择的编码智能体可能不支持其中某些功能；您可以跳过它们，或者尝试使用支持的其他编码智能体。
1. 使用编码智能体来完成与[代码质量讲座](/2026/code-quality/) 中的 Markdown 要点正则表达式练习相同的目标。它是否通过直接文件编辑来完成任务？代理直接编辑文件来完成此类任务有哪些缺点和限制？了解如何提示代理，使其不会通过直接文件编辑来完成任务。提示：要求代理使用[第一讲](/2026/course-shell/)中提到的命令行工具之一。
1. 大多数编码智能体支持某种形式的“yolo 模式”（例如，在 Claude Code 中，`--dangerously-skip-permissions`）。直接使用这种模式并不安全，但在虚拟机或容器等隔离环境中运行编码智能体然后启用自治操作可能是可以接受的。在您的计算机上运行此设置。 [Claude Code devcontainers](https://code.claude.com/docs/en/devcontainer) 或 [Docker Sandboxes / Claude Code](https://docs.docker.com/ai/sandboxes/agents/claude-code/) 等文档可能会派上用场。有不止一种方法可以进行设置。
