---
name: plan-decomposer
description: >-
  把 Codex、Claude、Cursor 等 coding agent 给出的抽象 plan 拆成可执行、可测试、可验收的 plans/*.md 开发清单，并生成或更新 docs/PROJECT_STATE.md。Use when starting a new project before implementation, decomposing a high-level plan into staged checklist files, defining main flow/core data structures/module boundaries, or preventing an AI coding agent from generating an entire project at once.
---

# Skill: plan-decomposer

## Purpose

把 AI 在 plan mode 中给出的抽象计划，拆解成多个可执行、可验收、可追踪的开发清单。

这个 skill 的目标不是让 AI 继续写更多代码，而是让人类 owner 能够清楚知道：

1. 主流程从哪里开始，到哪里结束。
2. 核心数据结构是什么。
3. 每个模块负责什么，不负责什么。
4. 每个开发阶段的输入、输出、测试和验收标准是什么。
5. 当前阶段完成后，项目状态应该如何变化。

## When to Use

在以下场景调用：

- 从 0 创建项目后，AI 已经给出初始 plan。
- Codex / Claude / Cursor 给出的计划太抽象，需要拆成真实开发步骤。
- 项目开始编码前，需要生成 `plans/*.md`。
- 想避免 AI 一口气生成完整项目。

## Required Inputs

调用时请提供：

```text
1. 项目目标
2. 初始 plan
3. 技术栈
4. 当前约束
5. MVP 范围
6. 非目标范围
7. 是否已有目录结构
```

如果信息缺失，不要立刻编造。先根据现有信息生成“待确认项”。

## Hard Rules

- 不要写代码。
- 不要修改代码。
- 不要创建复杂实现细节。
- 不要把 plan 写成宽泛建议。
- 每个开发清单必须可执行、可测试、可验收。
- 每个阶段必须有明确的“不做什么”。
- 每个阶段必须说明是否影响架构、数据流、模块契约、测试策略。

## Output Files

需要生成或更新：

```text
plans/00_MASTER_PLAN.md
plans/01_PROJECT_SCAFFOLD.md
plans/02_CORE_DATA_MODELS.md
plans/03_MINIMAL_VERTICAL_SLICE.md
plans/04_*.md
plans/05_*.md
...
docs/PROJECT_STATE.md
```

如果项目尚未有 `docs/PROJECT_BRIEF.md`，建议生成，但不要替代用户做产品决策。

## Procedure

### Step 1: Normalize the Initial Plan

先把原始 plan 归一化为以下结构：

```text
项目目标：
MVP 范围：
非目标范围：
主用户流程：
主技术流程：
核心模块：
核心数据结构：
外部依赖：
主要风险：
```

如果原始 plan 没有这些内容，标记为 `Missing / 待确认`。

### Step 2: Identify the Main Flow

必须明确主流程。

例如 RAG 项目：

```text
文档输入
  -> 文档解析
  -> 文档清洗
  -> 文档切分
  -> embedding
  -> 入库
  -> 用户问题
  -> 查询预处理
  -> 检索
  -> 重排
  -> 上下文组装
  -> 答案生成
  -> 评估
  -> 返回最终答案
```

要求输出：

```text
主流程入口：
主流程出口：
每个阶段的输入：
每个阶段的输出：
失败时应该在哪里暴露错误：
```

### Step 3: Identify Core Data Structures

列出核心数据对象。

每个对象必须说明：

```text
对象名称：
用途：
创建者：
修改者：
消费者：
关键字段：
不应该包含什么：
```

示例：

```text
Document
Chunk
Query
ResolvedQuery
SubQuery
RetrievedChunk
RerankedChunk
AnswerDraft
EvaluationResult
CorrectionDecision
FinalAnswer
```

### Step 4: Identify Module Boundaries

每个模块必须说明：

```text
模块名称：
职责：
不负责：
输入：
输出：
允许调用：
禁止调用：
测试方式：
```

特别注意检查：

```text
是否有模块职责重叠？
是否有模块既编排流程又处理业务逻辑？
是否有模块直接依赖外部 API 而没有 adapter？
是否有 prompt 和业务逻辑混在一起？
```

### Step 5: Split into Development Checklists

按照“先骨架，后主链路，再高级功能”的顺序拆分。

推荐阶段：

```text
00_MASTER_PLAN.md
01_PROJECT_SCAFFOLD.md
02_CORE_DATA_MODELS.md
03_MINIMAL_VERTICAL_SLICE.md
04_INGESTION_PIPELINE.md
05_QUERY_PIPELINE.md
06_EVALUATION.md
07_SELF_CORRECTION.md
08_OBSERVABILITY.md
09_HARDENING.md
```

可以根据项目类型调整，但必须遵守：

```text
先定义数据结构，再写复杂逻辑。
先跑通主链路，再加高级功能。
每个功能必须有验收标准。
```

## Checklist Template

每个 `plans/xx_*.md` 必须使用以下模板：

```md
# Plan XX: 阶段名称

## 1. Goal

本阶段要达成什么。

## 2. Non-Goals

本阶段明确不做什么。

## 3. Why This Stage Exists

为什么这个阶段必须独立存在。

## 4. Entry Conditions

开始本阶段前必须满足什么。

## 5. Affected Modules

| Module/File | Change Type | Reason |
|---|---|---|
| | create/update/delete | |

## 6. Core Data Structures

| Data Structure | Created By | Modified By | Consumed By |
|---|---|---|---|
| | | | |

## 7. Main Flow Position

本阶段插入主流程的位置：

```text
before -> current stage -> after
```

## 8. Tasks

- [ ] Task 1
  - Expected change:
  - Files:
  - Test:
  - Acceptance:
- [ ] Task 2
  - Expected change:
  - Files:
  - Test:
  - Acceptance:

## 9. Tests and Evaluation

必须说明：

- unit tests
- integration tests
- smoke tests
- golden cases
- manual verification command

## 10. Logging and Observability

本阶段应该产生哪些日志，帮助定位问题。

## 11. Acceptance Criteria

- [ ] 功能验收
- [ ] 测试验收
- [ ] 文档验收
- [ ] 人类可理解性验收

## 12. Risks

| Risk | Symptom | Mitigation |
|---|---|---|
| | | |

## 13. Rollback Plan

如果本阶段失败，如何回滚。

## 14. Documentation Updates

- [ ] docs/PROJECT_STATE.md
- [ ] docs/ARCHITECTURE.md
- [ ] docs/DATAFLOW.md
- [ ] docs/MODULE_CONTRACTS.md
- [ ] docs/EVAL_AND_TESTING.md
- [ ] docs/DECISIONS.md
```

## MASTER_PLAN Template

`plans/00_MASTER_PLAN.md` 必须包含：

```md
# Master Plan

## Project Goal

## MVP Scope

## Non-Goals

## Main Flow

## Core Data Structures

## Module Map

## Development Stages

| Stage | File | Goal | Exit Criteria |
|---|---|---|---|

## Global Risks

## Global Testing Strategy

## Documentation Strategy

## Human Control Checklist

在进入下一阶段前，人类 owner 必须能回答：

- [ ] 主流程从哪里开始，到哪里结束？
- [ ] 当前阶段改变了主流程的哪一部分？
- [ ] 当前阶段引入或修改了哪些数据结构？
- [ ] 当前阶段影响哪些模块？
- [ ] 当前阶段如何测试？
- [ ] 如果出错，应该看哪些日志或文件？
```

## PROJECT_STATE Initial Template

如果需要创建 `docs/PROJECT_STATE.md`，使用：

```md
# Project State

## One-Sentence Summary

## Current Capability

## Current Non-Capability

## Current Main Entry

## Current Main Flow

## Core Data Structures

## Module Map

## External Dependencies

## Current Test Commands

## Current Evaluation Method

## Current Logging / Observability

## High-Risk Areas

## Files Safe to Modify

## Files Requiring Caution

## Current Development Stage

## Last Completed Checklist Item

## Next Recommended Step

## Human Control Checklist

- [ ] I know where the main flow starts and ends.
- [ ] I know the core data structures.
- [ ] I know the module boundaries.
- [ ] I know how to test the current system.
- [ ] I know where to look when something breaks.
```

## Final Response Format

调用完成后，必须输出：

```text
已生成/更新的文件：
- ...

当前拆分出的开发阶段：
1. ...
2. ...

建议下一步：
...

需要人类确认的问题：
- ...
```

## Quality Bar

只有满足以下条件，才算完成：

- 每个阶段都可独立执行。
- 每个阶段都有明确输入输出。
- 每个阶段都有验收标准。
- 每个阶段都说明了不做什么。
- 能看出主流程如何从 0 到 MVP。
- 人类 owner 可以根据这些清单逐步建立项目认知。
