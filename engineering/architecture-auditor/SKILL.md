---
name: architecture-auditor
description: >-
  对已有或 AI 生成的复杂项目做只读架构审计，恢复人类 owner 对入口、主流程、核心数据结构、模块边界、文件分级、测试评估缺口和风险区域的掌控。Use when taking over a codebase, auditing a project before refactor or feature work, generating docs/ARCHITECTURE.md, docs/DATAFLOW.md, docs/MODULE_CONTRACTS.md, docs/EVAL_AND_TESTING.md, or determining whether a project is understandable and maintainable.
---

# Skill: architecture-auditor

## Purpose

接手一个已有项目时，做只读架构审计，快速恢复人类 owner 对项目的掌控感。

这个 skill 的目标不是重构，而是回答：

1. 主流程从哪里开始，到哪里结束。
2. 核心数据结构是什么，在哪些模块之间流动。
3. 每个模块的职责边界是什么。
4. 哪些文件是核心文件，哪些文件可能可删或废弃。
5. 加一个功能可能影响哪里。
6. 如何用测试、日志、评估集判断项目有没有坏。

## When to Use

在以下场景调用：

- 接手 AI 生成的复杂项目。
- 项目文件数增长后，人类 owner 失去掌控感。
- README 很长但仍然不知道系统怎么跑。
- 准备重构前，需要先恢复可理解性。

调用方式：

```text
/architecture-auditor
```

默认审计当前工作目录。不要要求用户先提供项目目标、运行方式、技术栈或困惑点；这些信息应从代码、README、配置、脚本和测试中恢复。

如果用户显式提供项目路径、运行方式或重点问题，可以作为辅助线索使用。

只有在无法确认审计目标路径时，才向用户询问。

如果某些事实无法从代码中确认，继续做只读审计，并标记 `Uncertain`。

## Hard Rules

- 禁止修改代码。
- 禁止重构。
- 禁止删除文件。
- 如果推断不确定，标记 `Uncertain`。
- 不要只输出文件列表，必须还原主流程、数据流、模块边界。

## Output Files

生成或更新：

```text
docs/ARCHITECTURE.md
docs/DATAFLOW.md
docs/MODULE_CONTRACTS.md
docs/EVAL_AND_TESTING.md
```

文件分级、风险、稳定建议和新增功能影响范围并入以上四份文档，无需生成额外审计报告。

## Procedure

### Step 1: Establish Project Orientation

目标：先判断这是一个什么系统，以及代码、配置、测试、脚本和文档分别在哪里。

需要回答：

```text
项目类型是什么？
主要源码在哪里？
配置、依赖、测试、脚本、文档分别在哪里？
哪些文件或目录看起来是主流程相关？
哪些信息无法从代码中确认，需要标记 Uncertain？
```

检查对象：

```text
目录结构
入口文件
配置文件
依赖文件
测试目录
脚本目录
文档目录
主要源代码目录
```

产物：

```text
项目类型：
主要包/模块：
可能入口：
测试位置：
配置位置：
外部依赖：
Uncertain：
```

### Step 2: Reconstruct Execution Paths

目标：恢复系统从入口到结果的主要执行路径，而不是只列入口文件。

需要回答：

```text
用户、CLI、API、pipeline 或测试分别从哪里进入？
主业务流程经过哪些模块？
辅助流程，如测试、评估、批处理、入库、迁移，如何运行？
编排逻辑是集中在少数文件，还是散落在多个文件？
```

必须识别的入口类型：

```text
CLI 入口
Web/API 入口
main 函数
pipeline 入口
测试入口
配置加载入口
```

入口产物：

```md
| Entry Point | File | How It Starts | Downstream Calls | Confidence |
|---|---|---|---|---|
| | | | | High/Medium/Low |
```

流程产物：

```text
主业务流程
辅助流程，如测试、评估、批处理、入库、迁移
```

示例：

```text
Ingestion Flow:
raw document -> loader -> cleaner -> chunker -> embedder -> vector store

Query Flow:
user query -> resolver -> decomposer -> retriever -> reranker -> generator -> evaluator -> final answer
```

如果代码中没有清晰流程，明确指出：

```text
主流程不集中，编排逻辑散落在多个文件。
```

### Step 3: Recover Data Contracts

目标：找出系统中真正流动的核心数据对象，以及它们是否有清晰契约。

需要回答：

```text
核心数据结构是什么？
谁创建、修改、消费这些数据？
数据在主流程中如何流动？
是否大量使用裸 dict 或隐式 schema？
哪些数据契约变化会影响主流程？
```

检查对象：

```text
dataclass
pydantic model
TypedDict
ORM model
plain dict schema
主要函数间传递的数据
```

产物：

```md
| Data Structure | File | Purpose | Created By | Modified By | Consumed By | Risk |
|---|---|---|---|---|---|---|
```

如果系统大量使用裸 dict，标记为风险：

```text
Risk: implicit schema / unclear data contract
```

### Step 4: Define Module Boundaries and File Priority

目标：明确模块职责边界，并告诉接手者哪些文件必须先理解，哪些可以稍后看。

需要回答：

```text
每个核心模块负责什么？
它不应该负责什么？
上游和下游是谁？
哪些文件是 A/B/C/D/E 类？
是否存在职责重叠、循环依赖、工具膨胀或业务逻辑混杂？
```

模块职责产物：

```md
| File | Responsibility | Should Not Do | Key Functions/Classes | Upstream | Downstream | Risk |
|---|---|---|---|---|---|---|
```

重点识别：

```text
职责过宽
职责重叠
循环依赖
工具函数膨胀
业务逻辑和 adapter 混杂
prompt 和控制流混杂
测试和生产逻辑混杂
```

文件分级标准：

```text
A 类：核心主流程文件，必须理解
B 类：重要服务模块，需要知道职责
C 类：adapter / 工具文件，知道输入输出即可
D 类：测试、脚本、样例，可稍后看
E 类：疑似废弃、重复或风险文件
```

文件分级产物：

```md
| File | Class | Reason | Action |
|---|---|---|---|
```

Action 可以是：

```text
Understand first
Keep
Review later
Candidate for merge
Candidate for deletion
Do not touch without tests
```

### Step 5: Assess Change Safety and Produce Control Docs

目标：判断项目是否可安全修改，并把审计结果沉淀成长期可用文档。

需要回答：

```text
当前测试覆盖了哪些主流程？
缺少哪些 smoke test、golden cases、evaluation dataset、日志或 trace？
最高风险区域在哪里？
新增功能前最可能需要查看哪些文件？
哪些文件不应轻易修改？
```

测试和评估检查对象：

```text
测试命令
测试文件
覆盖的模块
未覆盖的主流程
是否有 golden cases
是否有 smoke test
是否有 evaluation dataset
是否有日志/trace
```

测试和评估产物：

```md
| Area | Covered? | Evidence | Gap |
|---|---:|---|---|
```

重点风险类型：

```text
主流程不清
数据结构隐式
模块职责重叠
副作用隐藏
异常处理薄弱
外部 API 调用未隔离
缺少测试
缺少日志
功能散落
重复代码
无回滚策略
```

风险产物：

```md
| Risk | Location | Why It Matters | Severity | Suggested Next Step |
|---|---|---|---|---|
```

最终必须生成或更新以下控制文档。

#### ARCHITECTURE.md

必须包含：

```md
# Architecture

## System Summary

## Main Components

## Main Entry Points

## Main Flows

## Layering

## External Dependencies

## File Classification

| File | Class | Reason | Action |
|---|---|---|---|

## Change Impact Map

| Change Type | Likely Impacted Files | Why | Tests / Checks |
|---|---|---|---|

## Architecture Risks

## Recommended Architecture Direction
```

#### DATAFLOW.md

必须包含：

```md
# Data Flow

## Main Data Objects

## Flow 1: ...

## Flow 2: ...

## State and Storage

## Failure Points

## Data Contract Risks
```

#### MODULE_CONTRACTS.md

必须包含：

```md
# Module Contracts

| Module | Owns | Does Not Own | Input | Output | Side Effects | Tests |
|---|---|---|---|---|---|---|

## Boundary Risks

| Module | Boundary Risk | Evidence | Suggested Constraint |
|---|---|---|---|

## Feature Change Guide

| Feature Area | First Files to Inspect | Files Usually Affected | Files to Avoid Touching Without Reason |
|---|---|---|---|
```

#### EVAL_AND_TESTING.md

必须包含：

```md
# Evaluation and Testing

## Current Test Commands

## Existing Tests

## Missing Tests

## Smoke Tests

## Golden Cases

## Evaluation Strategy

## Manual Verification

## Logging / Observability

## Current Gaps
```

## Final Response Format

调用完成后输出：

```text
只读架构审计完成。

我确认没有修改代码。

生成/更新的文档：
- ...

当前我识别出的主流程：
...

A 类核心文件：
- ...

最高风险：
1. ...
2. ...
3. ...

新增功能前最可能需要查看：
- ...

建议下一步验证：
...
```

## Quality Bar

只有满足以下条件，才算完成：

- 明确项目类型、主要源码位置、配置位置、测试位置和外部依赖。
- 明确至少一个可信主入口，并说明它如何启动和调用下游模块。
- 明确至少一条主业务流程；如果主流程分散，必须指出编排逻辑散落的位置。
- 明确核心数据结构或主要隐式数据契约，并说明创建者、修改者和消费者。
- 明确核心模块职责、上下游关系和边界风险。
- 完成文件分级，至少识别 A 类核心文件和 E 类疑似废弃、重复或高风险文件。
- 明确当前测试、评估、日志或 trace 的覆盖情况，并列出主要缺口。
- 明确最高风险区域，并给出可执行的下一步建议。
- 生成或更新 `docs/ARCHITECTURE.md`、`docs/DATAFLOW.md`、`docs/MODULE_CONTRACTS.md`、`docs/EVAL_AND_TESTING.md`，语言为简体中文。
- 所有关键判断必须来自代码、配置、测试、脚本或现有文档；无法确认的内容必须标记 `Uncertain`。
- 没有修改、重构或删除任何项目代码。