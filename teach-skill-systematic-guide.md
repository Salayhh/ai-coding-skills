# `teach` Skill 系统性讲解

> 对应目录：[personal/teach/](personal/teach/)
> 核心文件：[personal/teach/SKILL.md](personal/teach/SKILL.md)

`teach` 是一个 Claude Code Skill，用于**在同一个 workspace 内把用户教会某个主题**。它不是一次性问答，而是一个持续多轮、有状态的教学过程。

---

## 1. 核心定位

- **名称**：`teach`
- **禁用模型调用**（`disable-model-invocation: true`）：意味着它本身不直接调用 LLM，而是提供一套教学协议/框架
- **参数提示**：`"What would you like to learn about?"` —— 触发时让用户说明想学的内容
- **本质**：把当前目录当作一个"教学 workspace"，所有学习状态都沉淀在目录里的文件中

---

## 2. 教学哲学：Knowledge / Skills / Wisdom

深层学习需要三样东西：

| 维度 | 含义 | 如何获得 | 教学策略 |
|------|------|---------|---------|
| **Knowledge（知识）** | 从高质量、高信任来源获取的事实与概念 | 阅读权威资料 | 先讲清楚，难度越低越好，别占用工作记忆 |
| **Skills（技能）** | 能灵活运用知识的 durable / flexible 能力 | 刻意练习、反馈循环 | 用难度做工具，强调提取练习、间隔、交错 |
| **Wisdom（智慧）** | 在真实世界与他人互动中沉淀的判断力 | 社区、实践、真实反馈 | 最终要引导用户加入高质量社区 |

### 2.1 关键区分：Fluency vs Storage Strength

- **Fluency（流畅性）**：当下能回忆起来，容易产生"我已经会了"的错觉
- **Storage（存储强度）**：长期真正记得住、用得上

教学目标是**存储强度**，所以要制造"desirable difficulty"：

- **Retrieval practice（提取练习）**：让用户从记忆中主动回忆，而不是重复阅读
- **Spacing（间隔）**：把练习分散到不同时间
- **Interleaving（交错）**：把相关但不同的主题混在一起练习（仅适用于技能练习）

---

## 3. Workspace 文件结构

`teach` skill 把教学过程物化成一组文件：

```
personal/teach/
├── SKILL.md                      # 教学协议本身（你正在看的）
├── MISSION-FORMAT.md             # MISSION.md 的格式规范
├── RESOURCES-FORMAT.md           # RESOURCES.md 的格式规范
├── LEARNING-RECORD-FORMAT.md     # 学习记录格式规范
├── GLOSSARY-FORMAT.md            # 术语表格式规范
│
├── MISSION.md                    # 用户为什么学这个（由师生共建）
├── RESOURCES.md                  # 高质量资源清单
├── NOTES.md                      # 用户偏好、临时笔记
│
├── lessons/                      # 单次课程（核心产出）
│   └── 0001-<dash-case>.html
├── reference/                    # 快速参考材料
│   └── *.html
├── learning-records/             # 学习记录（类似 ADR）
│   └── 0001-<dash-case>.md
└── assets/                       # 可复用组件（样式、quiz、模拟器等）
```

---

## 4. 各文件详细说明

### 4.1 `MISSION.md` —— 教学的指南针

**作用**：回答"用户为什么想学这个"。所有教学决策都要能追溯回这个文件。

**格式要求**（见 [MISSION-FORMAT.md](personal/teach/MISSION-FORMAT.md)）：

```md
# Mission: {Topic}

## Why
1-3 句话，描述具体的真实世界目标。
避免"理解 X"这种抽象说法，要写出"学完能做什么、生活/工作会有什么变化"。

## Success looks like
- 能观察到、可验证的具体能力 1
- 具体能力 2
- …

## Constraints
- 时间、预算、已有承诺、学习偏好等限制

## Out of scope
- 现在明确不追的相邻主题（保护最近发展区）
```

**关键规则**：

- 一个 workspace 只有一个 mission
- 具体 > 抽象
- 如果用户说不清，先访谈再写；烂 mission 比没有更糟
- mission 会变化，变了要及时更新并写学习记录

---

### 4.2 `RESOURCES.md` —— 高质量资源库

**作用**：收集可信来源。知识型讲解必须从这里取材，不能靠模型参数知识乱编。

**结构**（见 [RESOURCES-FORMAT.md](personal/teach/RESOURCES-FORMAT.md)）：

```md
# {Topic} Resources

## Knowledge
- [书名/文章名 — 作者](链接)
  一句话说明：覆盖什么内容、什么时候查它

## Wisdom (Communities)
- [社区名/论坛](链接)
  一句话说明：为什么靠谱、适合问什么问题
```

**规则**：

- 只放高信任资源：一手来源、公认专家、同行评审、管理严格的社区
- 每个条目必须带注解（ bare link 三个月后就没用了）
- 按 Knowledge / Wisdom 分组
- 如果发现某个领域没好资源，显式写一个 `## Gaps` 章节
- 定期修剪：错的、浅的、跑题的都要删掉

---

### 4.3 `lessons/*.html` —— 单次课程

**核心产出**。每节课是一个自包含的 HTML 文件，命名格式 `0001-<dash-case>.html`。

**设计原则**：

- **小**：能在很短时间内完成
- **一个明确、可验证的小胜利**
- **美**：排版干净、易读，参考 Tufte 风格（因为用户会回来复习）
- **紧扣 mission**
- **位于最近发展区内**

**每节课必须包含**：

- 知识讲解（只讲获得该技能所需的最少知识）
- 技能练习（通过交互反馈 loop）
- 链接到其他课程和 reference 文档（HTML anchor）
- 推荐一个高质量 primary source
- 提醒用户可以追问

**建议**：如果可能，用 CLI 命令帮用户打开课程文件。

---

### 4.4 `reference/*.html` —— 快速参考

**作用**：把课程中的 raw units of knowledge 压缩成便于速查的文档。

**特点**：

- 课程通常不会反复看，reference 会
- 要设计为"打印出来也很好看"
- 常见形式：语法速查、算法流程图、瑜伽体式表、健身动作库、术语表

**术语表（Glossary）尤其重要**：一旦建立，所有课程都要严格遵循其中的术语。

---

### 4.5 `learning-records/*.md` —— 学习记录

**作用**：教学版的 ADR（Architecture Decision Record）。记录"用户已经真正学会什么"、"纠正了什么误解"、"mission 为什么变化"等决策级信息。

**命名**：`0001-<dash-case>.md`，顺序递增。

**格式**（见 [LEARNING-RECORD-FORMAT.md](personal/teach/LEARNING-RECORD-FORMAT.md)）：

```md
# {简短标题}

1-3 句话：学到了什么（或建立了什么先验知识），以及对后续教学有什么影响。
```

**什么时候写**：

1. 用户真正理解了某个非 trivial 的东西（有证据，不只是听过）
2. 用户披露了先验知识（"我已经会 X"）
3. 纠正了一个误解
4. mission 因为学习而发生变化（同时更新 MISSION.md）

**什么时候不写**：

- 只是"覆盖"了某个材料
- 已在 GLOSSARY.md 中简洁定义过的术语
- 逐节课的活动日志

**可选字段**：

- `Status: active | superseded by LR-NNNN`：理解被更新时标记旧记录
- **Evidence**：用户如何展示出理解
- **Implications**：对后续课程解锁/排除什么

---

### 4.6 `GLOSSARY.md` —— 术语表

**作用**：这个 workspace 内的"官方语言"。所有讲解、练习、学习记录都要遵守。

**结构**（见 [GLOSSARY-FORMAT.md](personal/teach/GLOSSARY-FORMAT.md)）：

```md
# {Topic} Glossary

一两句话描述本术语表覆盖的主题。

## Terms

**Term**:
定义（一两句话，说明它是什么，而不是做什么或怎么做）。
_Avoid_: 同义词/避免说法 1, 避免说法 2
```

**规则**：

- 只有用户真正理解了一个概念后才加入术语表
- 对有多个叫法的概念，选一个最好的，其他列为 `Avoid`
- 定义要 tight，1-2 句
- 术语表里的术语可以互相引用，降低复杂概念的理解成本
- 必要时按主题分群（如 Anatomy、Programming）
- 对领域内有歧义的词，显式声明"在这个 workspace 中，X 指的是…"
- 理解加深后要及时更新，不保留陈旧定义

---

### 4.7 `assets/` —— 可复用组件

**作用**：存放跨课程复用的组件：样式表、quiz 小部件、模拟器、图表 helper 等。

**原则**：

- **Reuse is the default**：写第二节课前先看 assets 里有什么
- 第一节课就要创建一个共享 stylesheet，保证所有课程视觉一致
- 如果某节课需要新的、可复用的东西，优先写成 asset，不要 inline

---

### 4.8 `NOTES.md` —— 临时笔记

记录用户的教学偏好、工作笔记。设计课程时回头查看。

---

## 5. 教学流程

基于 SKILL.md，一次典型的 `teach` 会话大致如下：

```
1. 用户触发 /teach <topic>
   ↓
2. 检查 MISSION.md 是否已填充
   - 没有 → 访谈用户：Why？Success looks like？Constraints？Out of scope？
   ↓
3. 检查 RESOURCES.md
   - 如果资源不足 → 先搜索、筛选高质量资源，写入 RESOURCES.md
   ↓
4. 检查 learning-records/ 和 GLOSSARY.md
   - 判断用户当前水平（最近发展区）
   ↓
5. 设计一节课
   - 紧扣 mission
   - 小范围、可完成
   - 先讲知识，再练技能
   - 包含 quiz / 交互 / 真实操作
   ↓
6. 输出到 lessons/000X-<name>.html
   - 链接相关 reference / 其他课程
   - 推荐 primary source
   - 提醒用户追问
   ↓
7. 课后（或发现重要洞察时）
   - 写 learning-record
   - 必要时更新 MISSION.md / GLOSSARY.md / RESOURCES.md
```

---

## 6. 关键教学原则总结

| 原则 | 含义 | 实践方式 |
|------|------|---------|
| **Mission-driven** | 每节课都要 tied into mission | 开课、选题、资源筛选都回头查 MISSION.md |
| **Zone of Proximal Development** | 让学生感到"刚刚好够得着" | 读 learning-records，判断已会什么，再教最近一步 |
| **Knowledge first, then skill** | 先获取知识，再通过练习巩固 | 讲解要简单，练习要有 desirable difficulty |
| **Fluency ≠ Storage** | 当下的流畅会骗人 | 用提取练习、间隔、交错来强化长期记忆 |
| **Tight feedback loop** | 反馈越快越好 | quiz、自动检查、用户操作后立即反馈 |
| **Reference over lessons** | 复习时看 reference，不看课程 | 每节课都要产出一个可复用的 reference 片段 |
| **Community for wisdom** | 智慧来自真实世界互动 | 推荐高质量社区，用户不愿加入则尊重 |
| **Cite your sources** | 增加可信度 | 课程中到处放引用链接 |

---

## 7. 小陷阱与注意点

1. **不要迷信参数知识**：在 RESOURCES.md 充实之前，先去搜高质量资源。
2. **一课不要贪大**：工作记忆很小，一课只教一个 tightly-scoped 的东西。
3. **Quiz 答案要做平**：每个选项字数/字符数尽量一致，避免通过格式泄露答案。
4. **不要只 cover 不讲 evidence**：Material that was merely covered 不是 learning。
5. **mission 变了要确认**：更新前跟用户确认，并写 learning record。
6. **一个 workspace 一个 mission**：想学两个不相关的东西，开两个 workspace。

---

## 8. 这个 skill 的独特价值

`teach` 把"教学"从一个单次回答变成了一个**可追踪、可复盘、可沉淀的学习工程**：

- 状态存在文件里，下次继续
- 学习成果可回顾（lessons + reference + learning-records）
- 术语和假设被显式记录，避免前后不一致
- 教学过程被 mission 锚定，不容易跑题
- 资源被筛选和注解，防止用低质量来源

如果你是这个 workspace 的使用者，触发 `/teach` 后，Claude 会按照这个协议跟你共建一个结构化的学习旅程。
