---
layout: page
title: 计算机教育中缺失的一课
description: >
  学习能显著提升你开发效率的工具与工程实践。
subtitle: IAP 2026
nositetitle: true
excerpt: '' # work around a bug
---

# The Missing Semester of Your CS Education 中文版

大学里的计算机课程通常专注于讲授操作系统、数据库、机器学习等“学科内容”，但对“如何高效使用工具”常常着墨不多。  
在这个系列课程中，我们会系统讲解命令行、编辑器、版本控制、调试与性能分析、工程质量、代码交付，以及与 AI 协作开发。

学生在学习和工作中会花费大量时间使用这些工具。尽早建立正确的方法，能显著减少摩擦、降低重复劳动，并帮助你解决原本看起来很复杂的问题。

关于 [开设此课程的动机](/about/)。

# 日程 <span style="float:right"><img src = "https://img.shields.io/badge/文档同步时间-2026--02--26-blue"></span>

<ul>
{% assign lectures = site['2026'] | sort: 'date' %}
{% for lecture in lectures %}
    {% if lecture.phony != true %}
        <li>
        <strong>{{ lecture.date | date: '%-m/%d/%y' }}</strong>:
        {% if lecture.ready %}
            <a href="{{ lecture.url }}">{{ lecture.title }}</a><span style="float:right"><img src = "https://img.shields.io/badge/Chinese-✔-green"></span>
        {% else %}
            {{ lecture.title }} {% if lecture.noclass %}[no class]{% endif %}<span style="float:right"><img src = "https://img.shields.io/badge/Chinese-✘-orange"></span>
        {% endif %}
        </li>
    {% endif %}
{% endfor %}
</ul>

讲座视频可以在 [YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQunmnnTXrNbZnBaCA-ieK4L) 上找到。  
往期内容可访问 [2020 版本](/2020/)。

# 关于本课程

**教员**：本课程由 [Anish](https://anish.io/)、[Jon](https://thesquareplanet.com/) 和 [Jose](https://josejg.com/) 讲授。  
**问题**：请通过 [missing-semester@mit.edu](mailto:missing-semester@mit.edu) 联系课程团队。

# 在 MIT 之外

课程在 MIT 之外也有大量讨论与传播：

 - Hacker News（[2026](https://news.ycombinator.com/item?id=47124171), [2020](https://news.ycombinator.com/item?id=22226380), [2019](https://news.ycombinator.com/item?id=19078281)）
 - Lobsters（[2026](https://lobste.rs/s/q4ykw7/missing_semester_your_cs_education_2026), [2020](https://lobste.rs/s/ti1k98/missing_semester_your_cs_education_mit), [2019](https://lobste.rs/s/h6157x/mit_hacker_tools_lecture_series_on)）
 - r/learnprogramming（[2026](https://www.reddit.com/r/learnprogramming/comments/1r93yk6/the_missing_semester_of_your_cs_education_2026/), [2020](https://www.reddit.com/r/learnprogramming/comments/eyagda/the_missing_semester_of_your_cs_education_mit/)）
 - r/programming（[2020](https://www.reddit.com/r/programming/comments/eyagcd/the_missing_semester_of_your_cs_education_mit/)）
 - X（[2026](https://x.com/anishathalye/status/2024521145777848588)）
 - YouTube（[2026](https://www.youtube.com/playlist?list=PLyzOVJj3bHQunmnnTXrNbZnBaCA-ieK4L), [2020](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J), [2019](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuiujH1lpn8cA9dsyulbYRv)）

# 译文

- [Arabic](https://missing-semester-ar.github.io/)
- [Bengali](https://missing-semester-bn.github.io/)
- [繁体中文](https://missing-semester-zh-hant.github.io/)
- [German](https://missing-semester-de.github.io/)
- [Italian](https://missing-semester-it.github.io/)
- [Japanese](https://missing-semester-jp.github.io/)
- [Korean](https://missing-semester-kr.github.io/)
- [Portuguese](https://missing-semester-pt.github.io/)
- [Russian](https://missing-semester-rus.github.io/)
- [Spanish](https://missing-semester-esp.github.io/)
- [Turkish](https://missing-semester-tr.github.io/)
- [Vietnamese](https://missing-semester-vn.github.io/)

注意：上述链接为社区翻译，课程官方未逐一审校。

## 致谢

感谢 Elaine Mello 与 [MIT Open Learning](https://openlearning.mit.edu/) 对课程录制工作的支持。  
感谢 Luis Turino / [SIPB](https://sipb.mit.edu/) 对 [SIPB IAP 2026](https://sipb.mit.edu/iap/) 的支持。

---

<div class="small center">
<p><a href="https://github.com/missing-semester-cn/missing-semester-cn.github.io">Source code</a>.</p>
<p>Licensed under CC BY-NC-SA.</p>
<p>See <a href="/license">here</a> for contribution &amp; translation guidelines.</p>
</div>
