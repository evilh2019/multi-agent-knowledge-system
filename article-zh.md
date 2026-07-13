# 5 个 AI Agent 共享一套知识体系：我为什么设计了四层检索架构

> 198 个 skill，430+ 篇笔记，446 条知识库条目，5 个 Agent，3 个平台，20 个 cron job。当你的 AI 助手多到需要一套知识管理系统时，事情就变得有趣了。

---

## 问题：5 个 Agent 怎么共享知识？

我有 5 个 AI Agent，分别跑在 3 个不同的平台上：

- **马马1**（Hermes）— 思考层，负责架构和方向判断
- **虾虾1**（OpenClaw）— 执行层，负责落地
- **Oscar马1**（Hermes）— 工作赛道的思考层
- **Oscar虾1**（OpenClaw）— 工作赛道的执行层
- **CC1**（StepCode/Claude Code）— 独立编程

每个 Agent 有自己的记忆系统、工具链、运行时环境。但问题来了：**它们需要共享同一套知识。**

比如，我和马马1讨论过"Skill 治理框架"的架构设计，第二天虾虾1在处理一个 skill 审计任务时，它应该知道这个框架已经存在，而不是重新发明。

传统方案有两种：

1. **各自维护一份知识副本** — 导致信息漂移和重复劳动
2. **强制共用一份数据库** — 但 3 个平台没法共用同一个后端

我需要第三种方案。

---

## 方案：四层降级检索

核心思路很简单：**把所有知识分成四层，每个 Agent 按固定顺序检索，命中即停。**

```
L3  Hindsight 长期记忆    ← 最快，语义搜索近期对话
L2  Obsidian Vault 知识库  ← 最权威，正式编译的笔记
L1  IMA 知识库（飞书）     ← 最全，446 条原始资料
L0  外部搜索               ← 最新，仅 Editor 可触发
```

**为什么是这个顺序？** "速度 → 权威 → 覆盖 → 新鲜度"的权衡。

大多数问题在 L3（近期对话）就能命中。如果 L3 没找到，L2 的正式知识库是第二道防线。L1 是原始资料库，L0 是兜底。

**未命中怎么办？** 写 demand 文件排队。由 Editor Agent（马马1）统一处理——要么从 IMA 编译，要么外部搜索后入库。

---

## 关键设计：单 Editor 模式

5 个 Agent 都能读，但写权限是分层的：

| Agent | 写权限 |
|-------|--------|
| 马马1 | **全库写**（唯一 Editor） |
| Oscar马1 | 限定域写（体验+AI） |
| 虾虾1 | 限定域写（个人提案） |
| Oscar虾1 | 限定域写（工作提案） |
| CC1 | 只读 |

为什么不让所有 Agent 都能写？

**为防止知识漂移。** 同一个概念，不同 Agent 可能以不同方式编译，产生矛盾。单 Editor 模式牺牲了写入吞吐，换来了知识一致性。

这个决策的代价是 Editor 成为瓶颈。解决方式是 demand 队列 + 定时 merge，批量处理。

---

## 治理层：三条硬约束

### 1. Activity Log 只能追加，不能覆盖

所有 Agent 的每一次操作，都必须追加一行到 `activity-log.md`：

```
| 2026-07-14 | mama1 | reflex | L2 | OK | 查询xxx → vault/path/to/file.md |
```

**append-only 是硬约束。** 有一次，某个 Agent 用了 `>`（覆盖）而不是 `>>`（追加），导致 18 行历史记录丢失。此后，append-only 从"建议"升级为"架构级约束"，由 lint 脚本自动检测违规。

### 2. 问就查，不凭记忆答

Agent 不能靠"我记得"来回答问题。每次检索必须走四层降级，答前标注命中了哪一层、哪个文件。未命中就写 demand，不臆测。

### 3. 被纠正必记录

如果用户纠正了 Agent 的某个回答，这个纠正必须写回对应的 source 文件，标注 `trust=0.95` 和 `corrections` 记录。这样下次其他 Agent 查询同一主题时，不会犯同样的错误。

---

## 4 个关键架构决策

完整 ADR 见 [GitHub Repo](https://github.com/evilh2019/multi-agent-knowledge-system)，摘几个核心的：

**决策 1：为什么不用单一向量数据库？**

3 个平台、5 个 Agent，各自有独立的工具链。单一数据库意味着所有 Agent 必须能访问同一 API，这在跨平台场景下不可行。四层方案让每层独立演进。

**决策 2：为什么保留 IMA（飞书知识库）？**

446 条存量资料不能一夜丢弃。但 IMA 检索慢，且没有 rename/delete 能力。v3 计划逐步将 IMA 降级为纯归档。

**决策 3：为什么用 Markdown 日志而不是 SQLite？**

Markdown 文件所有 Agent 都能用 `grep`/`Read` 读取。append-only 保证历史不可篡改。SQLite 查询能力强，但增加了跨平台依赖。

**决策 4：为什么 L3 优先于 L2？**

大多数问题是关于近期讨论过的内容。L3（Hindsight 语义记忆）检索速度最快，命中率最高。L2（正式知识库）作为第二优先级，保证权威性。

---

## 怎么用？

**最小版本（1 天搭建）：**
1. 创建共享 vault 目录
2. 每个 Agent 的 SOUL 加 5 行反射纪律
3. 创建 activity-log（append-only）
4. 创建 demand 目录
5. 指定一个 Editor Agent

**完整版本（1 周）：**
加上 taxonomy、merge_demands cron、IMA 同步、kb-notify 通知层、编译管线、跨平台 skill 实现。

完整架构文档和 ADR：[github.com/evilh2019/multi-agent-knowledge-system](https://github.com/evilh2019/multi-agent-knowledge-system)

---

## 下一步

这套系统已经跑了 2 周，处理了 200+ 次检索，产出了 50+ 篇编译笔记。v3 规划中：IMA 降级为纯归档，Vault 成为唯一知识源，引入 trust decay 自动降级。

如果你也在搭建多 Agent 系统，或者对知识管理架构有兴趣，欢迎讨论。

---

*作者：Oscar（袁蔚），5 个 AI Agent 的"饲养员"，前产品经理，现全职探索 AI Agent 个人基础设施。*