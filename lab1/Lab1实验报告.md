# 2026年春季学期 计算学部《软件工程》课程 Lab 1 实验报告

## 基本信息
- 仓库：`shizhenneko/Lab1-2023112455`
- 仓库地址：`https://github.com/shizhenneko/Lab1-2023112455`
- 学生学号：2023112455
- 实验主题：基于大模型的编程与 Git 实战
- 报告依据：仓库 `main` 分支当前内容
- 本次核对版本：`e68b86e new test source`

---

## 1 实验要求
本实验要求基于英文文本文件构建带权有向图，并在该图上完成若干图算法与文本处理操作。具体来说，需要实现：从文本中读取数据、对文本做预处理、构建有向图、展示图结构、查询桥接词、根据桥接词生成新文本、计算最短路径、计算 PageRank 值，以及执行随机游走并输出结果。

除程序实现外，实验还要求结合大模型完成辅助设计与编码，并使用 Git / GitHub 完成仓库初始化、提交、分支管理、冲突合并和推送等操作。因此，本次 Lab 既考查编程与算法能力，也考查软件工程实践能力。

从整体目标来看，本实验不是单纯完成某一个算法函数，而是要求围绕“需求分析—设计实现—测试验证—版本管理”形成一个较完整的软件工程小项目。

---

## 2 待求解问题描述
本实验的核心问题可以描述为：给定一段英文文本，将文本中的单词相邻关系抽象成带权有向图，再基于该图完成若干图计算与文本生成任务。

### 2.1 输入数据
输入是英文文本文件，文件中可能包含：
- 大小写字母；
- 标点符号；
- 空格和换行；
- 重复单词；
- 重复出现的相邻单词对。

程序需要先对原始文本进行规整化处理，将所有非字母字符视为分隔符，将单词统一转为小写，再按顺序提取为单词序列。

### 2.2 输出数据
程序输出包括以下几类：
1. **有向图结构**：结点表示单词，边表示两个单词在原文中相邻出现，边权表示该相邻关系出现的次数；
2. **桥接词查询结果**：给定 `word1` 和 `word2`，输出所有满足 `word1 -> bridge -> word2` 的桥接词；
3. **新文本生成结果**：对输入的新句子，在相邻单词之间尝试插入桥接词；
4. **最短路径结果**：给定两个单词，输出最短路径及路径长度；
5. **PageRank 值**：输出指定单词在图中的 PageRank 分数；
6. **随机游走结果**：随机从图中某结点出发，沿边游走直到满足停止条件，并输出游走路径；
7. **图像文件**：将构建的有向图渲染为 PNG 图片。

### 2.3 约束条件
- 只处理英文单词；
- 非字母字符统一视为分隔符；
- 所有单词统一转为小写；
- 图为带权有向图；
- 输入单词不存在时要给出明确提示；
- 不存在路径时要给出提示；
- 随机游走遇到重复边或当前结点无后继时终止。

### 2.4 本人理解
从本质上说，这是一个“文本序列建模为图结构”的问题。原始文本是线性的，而图结构能够更自然地表达单词之间的连接关系。完成图抽象之后，桥接词、最短路径、PageRank 和随机游走等操作都可以统一在图上实现。这种做法既体现了抽象建模能力，也有利于程序模块化设计。

---

## 3 算法与数据结构设计

### 3.1 总体设计思路
当前仓库中的程序采用分层设计，按照“文本预处理 → 建图 → 图上计算 → 命令行交互”的思路组织代码。

项目主入口为 `lab1.app.TextGraphApplication`，其负责：
- 读取输入文件；
- 构建图；
- 输出结点数和边数；
- 在命令行菜单中调用各项功能。

功能模块分布如下：
- `lab1.util.TextNormalizer`：文本规整化；
- `lab1.graph.GraphBuilder` 与 `DirectedGraph`：图结构与建图；
- `lab1.io.TextFileLoader`：文件读取；
- `lab1.service.BridgeWordService`：桥接词查询；
- `lab1.service.NewTextGenerator`：新文本生成；
- `lab1.service.ShortestPathService`：最短路径；
- `lab1.service.PageRankService`：PageRank；
- `lab1.service.RandomWalkService`：随机游走；
- `lab1.service.GraphRenderer`：图像渲染。

这种设计将数据表示、业务逻辑和交互逻辑分离开来，结构较清晰，便于测试和维护。

### 3.2 文本规整化与建图
程序中先通过 `TextNormalizer.normalizeToWords` 对文本进行处理，其核心步骤为：
1. 将所有非字母字符替换为空格；
2. 将文本统一转换为小写；
3. 去除首尾空白；
4. 按空白符切分成单词序列。

随后，`GraphBuilder.build(List<String> words)` 负责根据单词序列建图：
1. 先把所有单词加入图中作为结点；
2. 再遍历相邻单词对 `words[i]` 和 `words[i+1]`；
3. 对每一对相邻单词建立一条有向边；
4. 若同一条边重复出现，则边权加一。

其流程可概括为：

```text
读取文本
  ↓
清洗文本（去非字母、转小写）
  ↓
得到单词序列
  ↓
遍历相邻单词对
  ↓
建立带权有向边
  ↓
生成有向图
```

### 3.3 有向图展示
项目中的 `GraphRenderer` 负责图像渲染。实现方式并不是简单文本输出，而是实际生成 PNG 图像：
- 画布大小为 `1400 × 1000`；
- 结点采用圆周布局；
- 支持边、箭头和边权标签绘制；
- 支持自环边绘制；
- 最终输出到 `out/directed-graph.png`。

因此，程序不仅满足“展示图结构”的要求，而且具备较完整的可视化效果。

### 3.4 桥接词查询算法
`BridgeWordService` 的逻辑如下：
1. 将输入单词统一转为小写；
2. 检查 `word1` 和 `word2` 是否存在于图中；
3. 枚举 `word1` 的所有直接后继；
4. 若某个后继结点能够直接到达 `word2`，则该结点是桥接词；
5. 根据桥接词数量输出不同格式的结果字符串。

这种算法直接基于图的局部结构完成查询，逻辑清晰，且符合题意要求。

### 3.5 生成新文本算法
`NewTextGenerator` 负责根据桥接词生成新文本，其步骤为：
1. 用正则表达式从输入句子中提取英文单词；
2. 将提取出的单词全部转为小写；
3. 对每一对相邻单词调用桥接词查询逻辑；
4. 若存在桥接词，则随机插入其中一个；
5. 将结果重新拼接成新的句子返回。

这种实现将桥接词查询逻辑复用到了文本生成模块中，体现了良好的代码复用性。

### 3.6 最短路径算法
`ShortestPathService` 使用 **Dijkstra 算法** 完成最短路径计算。实现过程为：
1. 初始化距离表，源点距离为 0，其余结点为无穷大；
2. 使用优先队列选取当前距离最小的结点；
3. 对其出边进行松弛操作；
4. 用 `previous` 映射记录前驱；
5. 当搜索结束后，从目标结点逆向回溯得到完整路径。

本项目将边权定义为文本中相邻关系的出现次数，因此求得的是“边权和最小”的路径。

### 3.7 PageRank 算法
`PageRankService` 实现了迭代版 PageRank，主要参数如下：
- 阻尼系数：`0.85`
- 收敛阈值：`1.0E-9`
- 最大迭代次数：`200`

具体方法为：
1. 初始时将所有结点分配相同分值；
2. 每一轮根据当前得分向后继结点分发权重；
3. 对无出边结点（sink node）将其分值均匀分发给全图；
4. 当相邻两轮总差值足够小时停止。

当前实现还专门验证了 sink node 情况下分值总和仍保持为 1，说明实现较为完整。

### 3.8 随机游走算法
`RandomWalkService` 的主要逻辑如下：
1. 随机选取一个起始结点；
2. 若当前结点存在出边，则随机选择其中一条继续走；
3. 用 `Set<Edge>` 记录访问过的边；
4. 若下一条边已被访问，则立即停止；
5. 若当前结点无后继边，也停止；
6. 将游走路径写入输出文件并返回字符串。

这里采用的是“检测重复边”而不是“检测重复结点”，与实验手册中的描述更一致。

### 3.9 数据结构设计
本项目最核心的数据结构位于 `DirectedGraph` 中：

```java
private final Map<String, Map<String, Integer>> adjacency = new LinkedHashMap<>();
```

其含义是：
- 外层 `Map` 的键为源单词；
- 内层 `Map` 的键为目标单词，值为边权。

该设计支持以下操作：
- `addNode`：添加结点；
- `addEdge`：添加边并自动累加权值；
- `containsWord`：判断单词是否存在；
- `getOutgoing`：获取出边集合；
- `getWeight`：查询边权；
- `size`：结点数量；
- `edgeCount`：边数量。

使用 `LinkedHashMap` 的好处是既能保持较高的访问效率，又能尽量稳定遍历顺序，有利于测试结果可重复。

### 3.10 时间复杂度分析
设规整化后的单词数为 `n`，图中结点数为 `V`，边数为 `E`。

1. **文本规整化与建图**：`O(n)`  
2. **桥接词查询**：约 `O(V)`  
3. **新文本生成**：若输入新文本有 `k` 个单词，则约为 `O(kV)`  
4. **最短路径（Dijkstra）**：`O((V + E) log V)`  
5. **PageRank**：单轮 `O(V + E)`，总计 `O(T(V + E))`  
6. **随机游走**：最坏情况下近似 `O(E)`  
7. **图渲染**：`O(V + E)`

### 3.11 大模型辅助情况
在本实验中，大模型主要被当作编程辅助工具使用，适合承担以下任务：
- 辅助梳理模块划分；
- 给出常见算法实现框架；
- 协助补充测试思路；
- 帮助整理实验文档。

例如可以使用如下提示词：
- “请用 Java 设计一个带权有向图结构，边权表示相邻单词出现次数。”
- “请给出文本规整化和建图的实现思路。”
- “请为 Dijkstra、PageRank 和随机游走设计对应的数据结构与接口。”

但大模型输出并不能直接照搬，还需要人工结合实验要求逐项校验，并通过真实运行与测试结果验证正确性。

---

## 4 算法代码的生成
从当前仓库结构看，代码组织较规范：
- `src/main/java` 中按功能拆分为 app、graph、io、service、util 等包；
- `src/test/java/lab1/tests` 中保留了完整的测试类；
- `src/test/resources` 中保留了多个测试文本；
- `out` 目录中保留了程序输出结果和 Git 证据材料。

这说明项目已经不只是“写出能跑的代码”，而是具有一定工程化组织能力。

在代码生成过程中，大模型可以帮助：
1. 先快速搭出类结构；
2. 为标准算法生成初稿；
3. 辅助补充测试样例；
4. 协助撰写说明文档。

但最终落地时，仍然需要人工完成以下工作：
- 调整类之间的职责边界；
- 校验输入输出格式是否符合题意；
- 修正异常情况和边界行为；
- 整理测试和实验报告。

从最终仓库来看，这些工作已经在实际项目中落实，因此本实验较好地体现了“大模型辅助 + 人工审查 + 测试验证”的开发方式。

---

## 5 实验与测试

### 5.1 测试资源
当前仓库中包含以下测试文本资源：
- `src/test/resources/sample.txt`
- `src/test/resources/Easy Test.txt`
- `src/test/resources/Cursed Be The Treasure.txt`

其中，`sample.txt` 是实验主流程验证使用的核心样例，内容为：

```text
To @ explore strange new worlds,
To seek out new life and new civilizations
```

规整化后得到：

```text
to explore strange new worlds to seek out new life and new civilizations
```

由此可以手工得到：
- 结点数：10
- 边数：12
- 典型边：
  - to → explore
  - explore → strange
  - strange → new
  - new → worlds
  - worlds → to
  - to → seek
  - seek → out
  - out → new
  - new → life
  - life → and
  - and → new
  - new → civilizations

### 5.2 测试体系说明
当前项目没有使用 JUnit，而是采用了自定义轻量测试框架：
- `TestCase`：测试接口；
- `Assert`：断言工具；
- `TestMain`：测试入口。

仓库当前共包含 **13 个测试类**。我基于最新仓库重新编译并运行，结果为：

```text
[PASS] TextNormalizerTest
[PASS] GraphBuilderTest
[PASS] GraphRendererTest
[PASS] Stage1IntegrationTest
[PASS] BridgeWordServiceTest
[PASS] NewTextGeneratorTest
[PASS] BridgeIntegrationTest
[PASS] ShortestPathServiceTest
[PASS] ShortestPathIntegrationTest
[PASS] PageRankServiceTest
[PASS] PageRankIntegrationTest
[PASS] RandomWalkServiceTest
[PASS] RandomWalkIntegrationTest
Passed 13 tests.
```

可见当前仓库版本已经通过全部已有测试。

### 5.3 读取文本文件并展示有向图
#### 输入文本
```text
To @ explore strange new worlds,
To seek out new life and new civilizations
```

#### 期望结果
- 程序成功读取文本并建图；
- 结点数为 10；
- 边数为 12；
- 输出图像文件 `out/directed-graph.png`；
- 保留跨行相邻关系 `worlds -> to`。

#### 实际结果
我重新运行当前仓库主程序后，得到：

```text
Graph loaded successfully.
Words: 10
Edges: 12
Directed graph image saved to: out/directed-graph.png
```

#### 结论
运行结果与预期一致，说明文本读取、规整化、建图和渲染功能均正确。

### 5.4 查询桥接词
| 序号 | 输入 | 期望输出 | 实际输出 | 是否正确 |
|---|---|---|---|---|
| 1 | explore, new | The bridge words from "explore" to "new" is: "strange". | The bridge words from "explore" to "new" is: "strange". | 正确 |
| 2 | to, explore | No bridge words from "to" to "explore"! | No bridge words from "to" to "explore"! | 正确 |
| 3 | exciting, new | No "exciting" in the graph! | No "exciting" in the graph! | 正确 |
| 4 | new, and | The bridge words from "new" to "and" is: "life". | The bridge words from "new" to "and" is: "life". | 正确 |

说明：第 1—3 条来自 `BridgeWordServiceTest`，第 4 条来自 `BridgeIntegrationTest`。

### 5.5 根据桥接词生成新文本
| 序号 | 输入 | 期望输出 | 实际输出 | 是否正确 |
|---|---|---|---|---|
| 1 | Seek to explore new and exciting synergies | seek to explore strange new life and exciting synergies | seek to explore strange new life and exciting synergies | 正确 |
| 2 | Hello world | hello world | hello world | 正确 |

说明：
- 第 1 条来自 `NewTextGeneratorTest` 和当前主程序真实运行；
- 第 2 条体现了无桥接词时仅做单词提取和小写规整化。

### 5.6 计算最短路径
| 序号 | 输入 | 期望输出 | 实际输出 | 是否正确 |
|---|---|---|---|---|
| 1 | to, and | 两条长度为 5 的最短路径之一 | 测试允许任一合法最短路径 | 正确 |
| 2 | civilizations, to | No path from "civilizations" to "to". | No path from "civilizations" to "to". | 正确 |
| 3 | to, civilizations | Shortest path from "to" to "civilizations": ... (length: 4) | Shortest path from "to" to "civilizations": to -> explore -> strange -> new -> civilizations (length: 4) | 正确 |

说明：
- 第 1 条来自 `ShortestPathServiceTest` 与 `ShortestPathIntegrationTest`；
- 第 2 条来自 `ShortestPathServiceTest`；
- 第 3 条来自我对当前程序的真实运行。

### 5.7 计算 PageRank
| 序号 | 输入 | 期望输出 | 实际输出 | 是否正确 |
|---|---|---|---|---|
| 1 | new | 接近 0.24 | 0.2405888935099065 | 正确 |
| 2 | missing | 不存在提示或返回空值 | service 层返回 `null`，应用层输出 `Word not found in graph.` | 正确 |
| 3 | sink node 情况 | 总分保持归一化 | 测试验证通过 | 正确 |

说明：PageRank 部分不仅验证了普通结点分值，也验证了无出边结点场景下算法的正确性。

### 5.8 随机游走
随机游走具有随机性，因此测试重点是“停止条件是否正确”和“是否写入文件”。

| 序号 | 实际输出 | 是否正确 |
|---|---|---|
| 1 | worlds to seek out new civilizations | 正确 |
| 2 | alpha beta gamma beta | 正确 |
| 3 | a b c a | 正确 |

说明：
- 第 1 条来自我对当前程序的真实运行；
- 第 2、3 条来自 `RandomWalkServiceTest`；
- `RandomWalkIntegrationTest` 还验证了结果会同步写入文件。

---

## 6 编程语言与开发环境
### 6.1 编程语言
- Java

### 6.2 JDK 版本
根据仓库 `README.md`，项目建议使用：
- **JDK 23**

### 6.3 IDE 与工程配置
项目采用 IntelliJ IDEA 工程结构，仓库中包含：
- `.idea/` 配置目录；
- `Lab1-2023112455.iml` 工程文件；
- `Run Main` 与 `Run Tests` 运行配置。

### 6.4 项目目录
- 源码目录：`src/main/java`
- 测试目录：`src/test/java`
- 测试资源：`src/test/resources`
- 输出目录：`out`
- 脚本目录：`scripts`

### 6.5 大模型使用情况
本实验中，大模型主要用于：
- 方案设计与模块划分；
- 常见算法代码框架生成；
- 测试思路整理；
- 文档润色与总结。

但最终结果仍需人工检查，并通过程序运行与测试结果验证。

---

## 7 结对编程
根据实验手册说明，本实验以个人独立完成为主，因此本节按个人完成情况填写。

### 7.1 分组依据
本实验未进行正式双人分组，采用个人独立完成方式。

### 7.2 角色切换与任务分工
虽然没有与同学结对，但从开发流程上可以划分为：
- 需求分析；
- 代码实现；
- 测试验证；
- 文档整理。

### 7.3 工作照片
该部分需结合本人实际实验过程补充。

### 7.4 工作日志
可按以下方式描述：
- 阅读实验手册并明确需求；
- 完成文本规整化与图结构设计；
- 实现桥接词、生成新文本、最短路径、PageRank 和随机游走功能；
- 补充图渲染与测试；
- 完善 Git 操作证据和实验报告。

---

## 8 Git 操作过程
本次实验第二部分的 Git 操作，在仓库中保留了较完整的证据材料，主要位于：
- `out/git-evidence/Git操作记录.md`
- `out/git-evidence/run_git_lab.ps1`
- `out/git-evidence/continue_git_lab.ps1`
- `out/git-evidence/logs/`
- `out/git-evidence/screenshots/`

从这些材料可以看出，Git 实验部分不是口头描述，而是实际进行了脚本化记录与截图留证。

### 8.1 场景（1）：仓库创建与提交
根据 `Git操作记录.md`，已完成：
- 本地仓库初始化；
- 首次提交；
- 多次修改后的追加提交；
- 撤销最后一次提交；
- 查看完整提交历史。

对应日志包括：
- `01_r0_before_init.txt`
- `02_r1_init_and_status.txt`
- `03_r2_first_commit.txt`
- `04_r3_diff_after_first_edit.txt`
- `05_r4_second_commit.txt`
- `06_r5_third_commit.txt`
- `07_r6_undo_last_commit.txt`
- `08_r7_full_history_after_reset.txt`

这部分说明实验已经完整覆盖了“初始化—提交—修改—回退—查看历史”的基础 Git 流程。

### 8.2 场景（1）：推送到 GitHub
根据仓库证据，已完成远程仓库关联与推送：
- 远程仓库：`https://github.com/shizhenneko/Lab1-2023112455`
- 已保存远程配置与推送日志；
- 已保存对应终端截图。

相关证据包括：
- `10_r8_remote_setup.txt`
- `11_r9_push_master.txt`
- `screenshots/10_r8_remote_setup.png`
- `screenshots/11_r9_push_master.png`

### 8.3 场景（2）：分支管理
根据 `Git操作记录.md`，实验中完成了：
- 查看分支并切换到 `master`；
- 创建 `B1`、`B2` 分支；
- 从 `B2` 创建 `C4` 分支；
- 分别在分支上修改并提交；
- 将 `C4` 合并到 `B1`，并处理冲突；
- 将未合并修改继续合并到学号分支 `2023112455`；
- 推送学号分支到 GitHub。

对应证据包括：
- 日志：`12_r1_branch_list_and_checkout_master.txt` 至 `22_r11_version_tree.txt`
- 截图：`17_r6_merge_c4_into_b1.png`、`19_r8_branch_merge_status.png`、`21_r10_push_student_branch.png`、`22_r11_version_tree.png`
- GitHub 页面截图：`github-2023112455.png`

这说明实验第二部分不只是做了简单分支创建，而是真正完成了多分支修改、冲突合并和远程推送的完整过程。

### 8.4 场景（3）：在 IDE 中使用 Git
仓库本身包含 IntelliJ IDEA 项目配置，因此可以通过 IDE 的 Git 插件执行：
- 查看修改内容；
- 选择文件提交；
- 查看提交历史；
- 推送到远程仓库。

虽然仓库中的主要证据以命令行日志和截图为主，但与 IDEA 工程结构结合后，可以证明该项目具备在 IDE 中使用 Git 的条件。

---

## 9 对本次实验结果的分析
从当前仓库来看，本实验已经较完整地覆盖了实验手册要求：

1. **程序功能完整**  
   已实现文本规整化、建图、桥接词查询、新文本生成、最短路径、PageRank、随机游走和图像渲染。

2. **工程结构清晰**  
   代码按功能拆分为多个包和服务类，主程序只负责调度和交互，结构较合理。

3. **测试较充分**  
   当前仓库保留了 13 个测试类，并全部通过；同时还加入了多个测试资源文件，说明开发者已经重视验证过程。

4. **Git 实验部分证据充分**  
   当前仓库不只是保留了代码，还额外保存了 Git 操作日志、截图、脚本和说明文档，这一点对实验报告填写很有帮助，也能增强结果的可信度。

5. **大模型使用较合理**  
   从项目组织方式来看，大模型更像是辅助开发和整理的工具，而不是直接替代人工完成实验。最终成果仍然体现了人工整合、调试和验证的过程。

---

## 10 小结
通过本次 Lab 1，我完成了从英文文本建模到有向图分析的一整套实现流程，并在此基础上完成了图算法计算、图像渲染、命令行交互、测试验证和 Git 版本管理等工作。相比单纯完成一个编程题，这次实验更像是一次小型软件工程实践。

在编程部分，我进一步理解了如何将文本抽象为图结构，并在统一的数据模型上实现桥接词、最短路径、PageRank 和随机游走等功能。这样的实现方式不仅逻辑清楚，也有利于程序扩展与测试。

在工程实践部分，我通过 Git 完成了仓库初始化、提交、回退、分支管理、冲突合并和远程推送，并保留了日志与截图证据。这使我对 Git 在软件配置管理中的作用有了更直观的认识。

在开发方式上，我也体会到大模型确实可以提升效率，尤其是在梳理结构、生成算法框架和整理文档方面很有帮助。但与此同时，我也认识到，大模型不能替代人工判断。真正决定项目质量的，仍然是对需求的理解、对边界情况的处理、对结果的验证，以及对整个过程的工程化组织。

总的来说，本次实验让我同时巩固了 Java 面向对象编程、图算法实现、测试方法和 Git 使用，也让我更清楚地认识到：只有将大模型辅助、人工审查、测试验证和版本管理结合起来，才能形成可靠的软件工程实践流程。
