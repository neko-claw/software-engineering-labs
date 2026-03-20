# 2026年春季学期 计算学部《软件工程》课程
# Lab 1实验报告

姓名：__________  
班级/学号：2023112455  
联系方式：__________

## 1 实验要求
本实验要求完成一个基于英文文本的图处理程序。程序需要从文本文件中读取内容，对文本进行规范化处理，并根据单词的相邻关系构建带权有向图。在构图基础上，需要实现图展示、桥接词查询、根据桥接词生成新文本、最短路径计算、PageRank值计算和随机游走等功能。同时，本实验还要求结合大模型辅助完成需求分析、算法设计、编码和文档整理，并利用 Git / GitHub 完成代码版本管理、分支管理与提交记录维护。

我理解这次实验不只是完成几个函数，而是要把“需求理解、程序设计、算法实现、测试验证、版本管理”串成一个完整的软件工程小项目。因此，除了程序功能本身是否正确，还需要关注代码组织是否清晰、测试是否充分、Git 过程是否可追溯。

## 2 待求解问题描述
本实验要解决的问题是：把英文文本中的单词顺序关系转化为带权有向图，并在该图上完成一系列操作。

### 输入数据
输入是英文文本文件。文件中可能包含字母、标点符号、空格、换行，以及重复单词和重复相邻关系。程序需要将文本中的非字母字符当作分隔符，将所有单词统一转换为小写，然后按原顺序提取单词序列。

### 输出数据
输出主要包括以下几类：
- 构建后的有向图；
- 查询桥接词得到的字符串结果；
- 插入桥接词后生成的新文本；
- 两单词之间的最短路径与路径长度；
- 指定单词的 PageRank 值；
- 随机游走得到的路径和保存到文件中的结果；
- Git 实验部分的命令执行记录、截图和远程仓库结果。

### 约束条件
- 只处理英文单词；
- 非字母字符统一视为分隔符；
- 所有单词统一转成小写；
- 图为带权有向图，边权表示相邻关系出现次数；
- 对不存在的单词或不存在的路径需要给出提示；
- 随机游走遇到重复边或无后继边时结束。

我的理解是，这个实验的关键在于“把线性文本映射成图结构”。一旦这个抽象建立起来，后续桥接词、最短路径、PageRank 和随机游走都可以在统一的数据结构上实现，程序逻辑会更清晰，也更方便测试。

## 3 算法与数据结构设计

### 3.1 设计思路与算法流程图
程序先读取文本文件，再对文本进行规范化处理，得到单词序列。之后遍历所有相邻单词对，构建带权有向图。图构建完成后，再分别调用不同模块完成图展示、桥接词查询、新文本生成、最短路径、PageRank 和随机游走等功能。

整体流程可以概括为：

```text
读取文本文件
  ↓
文本规范化（去除非字母、转小写、切分单词）
  ↓
得到单词序列
  ↓
根据相邻单词生成带权有向图
  ↓
调用不同功能模块进行图操作
```

如果分模块来看：
1. **根据文本生成图**：对相邻单词建立有向边，重复出现则边权加一；
2. **展示有向图**：将图进行布局后渲染为 PNG 文件；
3. **查询桥接词**：寻找满足 `word1 -> x -> word2` 的中间结点；
4. **根据桥接词生成新文本**：遍历输入新句子的相邻单词对，若存在桥接词则插入；
5. **计算最短路径**：基于 Dijkstra 算法求最短路径；
6. **计算 PR 值**：采用 PageRank 迭代算法；
7. **随机游走**：从随机结点开始，沿随机出边行走，遇到重复边或无出边时停止。

### 大模型辅助提示词与分析
在设计算法时，可以使用如下提示词辅助思考：
- “请用 Java 设计一个带权有向图数据结构，结点是单词，边权是相邻出现次数。”
- “请给出从英文文本生成单词图的处理流程。”
- “请为桥接词、最短路径、PageRank 和随机游走分别设计对应算法。”

在实际使用中，大模型给出的算法框架通常比较快，但存在两个问题：
1. 容易只给出通用算法，而没有完全贴合本实验的输入输出格式；
2. 容易忽略边界情况，例如图为空、单词不存在、随机游走终止条件等。

因此，在得到大模型结果后，还需要结合实验要求对接口、字符串输出、停止条件和测试样例进行人工修正。

### 3.2 数据结构设计
本实验中最核心的数据结构是带权有向图。当前仓库中图结构的核心表示为：

```java
Map<String, Map<String, Integer>> adjacency
```

含义如下：
- 外层 `Map` 的键表示起点单词；
- 内层 `Map` 的键表示终点单词，值表示边权；
- 边权表示该相邻关系在文本中出现的次数。

除此之外，还使用了以下数据结构：
- `List<String>`：存放规整化后的单词序列；
- `PriorityQueue<PathNode>`：在 Dijkstra 中维护当前最短路径候选；
- `Map<String, String>`：记录最短路径前驱；
- `Map<String, Double>`：保存 PageRank 值；
- `Set<Edge>`：记录随机游走中已访问的边；
- `Map<String, NodePosition>`：图渲染时保存结点坐标。

### 大模型辅助提示词与分析
在数据结构设计部分，可以使用提示词：
- “请设计一个适合表示单词带权有向图的 Java 数据结构。”
- “如果需要实现 Dijkstra、PageRank 和随机游走，分别还需要哪些辅助数据结构？”

大模型在这部分给出的建议通常合理，但有时会推荐过于复杂的图封装方式。结合本实验规模和需求，采用 `Map<String, Map<String, Integer>>` 更直接，也更容易调试和验证。

### 3.3 算法时间复杂度分析
设文本规整化后有 `n` 个单词，图中结点数为 `V`，边数为 `E`。

- 根据文本生成图：需要遍历所有单词和相邻单词对，时间复杂度为 `O(n)`；
- 展示有向图：需要布局和绘制所有结点、边，复杂度约为 `O(V + E)`；
- 查询桥接词：枚举 `word1` 的后继并检查到 `word2` 的可达关系，复杂度约为 `O(V)`；
- 根据桥接词生成新文本：若新输入有 `k` 个单词，则约为 `O(kV)`；
- 计算最短路径：使用 Dijkstra，复杂度为 `O((V+E)logV)`；
- 计算 PageRank：若迭代 `T` 轮，则复杂度约为 `O(T(V+E))`；
- 随机游走：在最坏情况下访问边的数量不超过 `E`，复杂度可记为 `O(E)`。

### 大模型辅助提示词与分析
在复杂度分析部分，可以使用提示词：
- “请分析单词图各个功能模块的时间复杂度。”
- “如果图使用邻接表结构实现，Dijkstra 和 PageRank 的复杂度应如何表示？”

大模型能够给出大致正确的复杂度形式，但仍然需要人工确认变量含义，并结合代码里的实际数据结构来判断。

## 4 算法代码的生成
在编码阶段，可以让大模型帮助生成代码框架，例如：
- “请用 Java 写一个文本规整化函数，将非字母字符替换为空格并统一转小写。”
- “请实现一个带权有向图类，支持添加结点、添加边和查询边权。”
- “请实现 Dijkstra、PageRank 和随机游走的 Java 版本。”

结合当前仓库内容看，代码最终被组织为多个类和模块，而不是简单堆在一个主函数里，说明大模型产生的初稿经过了人工整理与重构。

我对大模型生成代码的评价是：
1. **优势**：能快速给出标准算法和样板代码，节省编码时间；
2. **问题**：字符串输出、边界情况、实验要求细节往往需要人工修正；
3. **修改方式**：结合真实输入、测试结果和实验要求逐步修改，最终保留符合题意的实现。

## 5 实验与测试
实验手册要求使用 `Easy Test.txt` 和 `Cursed Be The Treasure.txt` 两个文件进行测试；同时，仓库中还保留了 `sample.txt` 作为核心样例。我在整理报告时同时参考了仓库里的测试代码、仓库中的截图证据，以及我本次重新运行得到的输出结果。

### 5.1 读取文本文件并展示有向图
下面以 `sample.txt` 为例说明程序行为，因为当前仓库中的功能测试主要围绕该样例展开。

文本文件中包含的内容：

```text
To @ explore strange new worlds,
To seek out new life and new civilizations
```

期望生成的图（手工计算得到）：
- 10 个结点：`to, explore, strange, new, worlds, seek, out, life, and, civilizations`
- 12 条边：`to->explore, explore->strange, strange->new, new->worlds, worlds->to, to->seek, seek->out, out->new, new->life, life->and, and->new, new->civilizations`

程序实际生成结果：
- `Words: 10`
- `Edges: 12`
- 图像文件已生成到 `out/directed-graph.png`

二者是否一致：一致。

给出实际运行得到结果的界面截图：
- `screenshots/screenshot_graph_text.png`
- `artifacts/directed-graph-latest.png`
- `screenshots/app_output_latest.txt`

### 5.2 查询桥接词
| 序号 | 输入（2个单词） | 期望输出 | 实际输出 | 运行是否正确 |
|---|---|---|---|---|
| 1 | explore, new | The bridge words from "explore" to "new" is: "strange". | The bridge words from "explore" to "new" is: "strange". | 正确 |
| 2 | to, explore | No bridge words from "to" to "explore"! | No bridge words from "to" to "explore"! | 正确 |
| 3 | exciting, new | No "exciting" in the graph! | No "exciting" in the graph! | 正确 |

给出实际运行得到结果的界面截图：
- `screenshots/screenshot_bridge.png`
- `screenshots/app_output_latest.txt`

### 5.3 根据桥接词生成新文本
| 序号 | 输入（一行文本） | 期望输出 | 实际输出 | 运行是否正确 |
|---|---|---|---|---|
| 1 | Seek to explore new and exciting synergies | seek to explore strange new life and exciting synergies | seek to explore strange new life and exciting synergies | 正确 |
| 2 | Hello world | hello world | hello world | 正确 |
| 3 | Seek out new civilizations | seek out new civilizations | seek out new civilizations | 正确 |

给出实际运行得到结果的界面截图：
- `screenshots/screenshot_new_text.png`
- `screenshots/app_output_latest.txt`

### 5.4 计算最短路径
| 序号 | 输入（两个单词、或一个单词） | 期望输出 | 实际输出 | 运行是否正确 |
|---|---|---|---|---|
| 1 | to, and | 两条长度为 5 的最短路径之一 | 测试允许任一合法最短路径 | 正确 |
| 2 | civilizations, to | No path from "civilizations" to "to". | No path from "civilizations" to "to". | 正确 |
| 3 | to, civilizations | 长度为 4 的最短路径 | to -> explore -> strange -> new -> civilizations (length: 4) | 正确 |

给出实际运行得到结果的界面截图：
- `screenshots/screenshot_shortest_path.png`
- `screenshots/app_output_latest.txt`

### 5.5 计算PageRank值
| 序号 | 单词 | 期望输出 | 实际输出 | 运行是否正确 |
|---|---|---|---|---|
| 1 | new | 接近 0.24 | 0.2405888935099065 | 正确 |
| 2 | missing | 不存在提示或空值 | 主程序输出 `Word not found in graph.` | 正确 |
| 3 | sink node 场景 | 分值和保持为 1 | 仓库测试验证通过 | 正确 |

给出实际运行得到结果的界面截图：
- `screenshots/screenshot_pagerank.png`
- `screenshots/app_output_latest.txt`

### 5.6 随机游走
该功能无输入，让程序执行多次，分别记录结果。

| 序号 | 实际输出 | 程序运行是否正确 |
|---|---|---|
| 1 | seek out new worlds to seek | 正确 |
| 2 | alpha beta gamma beta | 正确 |
| 3 | a b c a | 正确 |

给出实际运行得到结果的界面截图：
- `screenshots/screenshot_random_walk.png`
- `screenshots/app_output_latest.txt`
- `artifacts/random-walk-latest.txt`

### 补充测试证据
仓库中还保留了完整测试输出，本次重新运行结果为：

```text
Passed 13 tests.
```

对应证据：
- `screenshots/screenshot_tests.png`
- `screenshots/test_output_latest.txt`

## 6 编程语言与开发环境
- 编程语言：Java
- JDK 版本：仓库 README 推荐 JDK 23
- 实际验证环境：当前环境能够成功编译和运行全部程序及测试
- IDE：IntelliJ IDEA（仓库中包含 `.idea`、`.iml` 和运行配置）
- 采用的大模型：用于辅助梳理模块结构、算法框架、测试思路和报告撰写

## 7 Git操作过程

### 7.1 实验场景(1)：仓库创建与提交
仓库中已经保留了完整 Git 操作证据，主要位于 `out/git-evidence` 目录。对应操作包括：
- 仓库初始化；
- 首次提交；
- 修改后再次提交；
- 撤销最近一次提交；
- 查看完整历史；
- 配置远程并推送到 GitHub。

对应命令和截图证据可引用：
- `screenshots/03_r2_first_commit.png`
- `screenshots/04_r3_diff_after_first_edit.png`
- `screenshots/07_r6_undo_last_commit.png`
- `screenshots/10_r8_remote_setup.png`
- `screenshots/11_r9_push_master.png`
- `screenshots/git_evidence_record.md`

GitHub 上的仓库界面证明提交成功：
- 仓库地址：`https://github.com/shizhenneko/Lab1-2023112455`

### 7.2 实验场景(2)：分支管理
仓库中的 Git 实验证据表明，已经完成了以下过程：
- 创建 `B1`、`B2` 分支；
- 从 `B2` 创建 `C4` 分支；
- 分支上分别修改并提交；
- 合并 `C4` 到 `B1` 并处理冲突；
- 将未合并内容继续合并到学号分支 `2023112455`；
- 推送学号分支到 GitHub。

对应截图证据：
- `screenshots/17_r6_merge_c4_into_b1.png`
- `screenshots/19_r8_branch_merge_status.png`
- `screenshots/21_r10_push_student_branch.png`
- `screenshots/22_r11_version_tree.png`
- `screenshots/github-2023112455.png`

## 8 在IDE中使用Git Plugin
仓库本身包含 IntelliJ IDEA 工程配置，因此可以在 IDEA 中直接使用 Git Plugin 查看文件修改、提交本地仓库以及推送到 GitHub。虽然当前仓库保留的主要是命令行截图和日志，但配合 IDEA 项目结构，可以证明该项目能够在 IDE 内进行 Git 管理。

如需在 Word 正文中插图，可优先插入：
- `screenshots/screenshot_git.png`
- `screenshots/github-2023112455.png`
- `screenshots/22_r11_version_tree.png`

## 9 小结
和传统的“自己一点点查资料、自己从零写代码”的方式相比，引入大模型辅助后，最大优势是能明显提高开发效率，尤其是在模块拆分、常见算法实现和文档整理方面。对于这次实验中的文本规整化、图结构、Dijkstra、PageRank 和随机游走，大模型都能快速给出可用框架。

但同时，大模型带来的额外工作也很明显：
1. 需要不断核对它的输出是否真正符合实验要求；
2. 需要人工补足边界条件和测试样例；
3. 需要人工检查字符串输出、文件路径、图结构细节和随机行为是否符合题意；
4. 最终还需要人工整理代码结构、Git 证据和实验报告。

因此，我认为大模型最适合扮演“开发助手”的角色。它能大幅减少机械性工作，但不能替代人工判断。只有把大模型辅助、人工修正、测试验证和 Git 管理结合起来，才能真正形成可靠的软件工程实践流程。
