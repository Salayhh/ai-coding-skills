---
name: project-state-updater
description: >-
  在每次 AI coding、重构、bug fix 或 checklist item 完成后，同步 docs/PROJECT_STATE.md、对应 plans/*.md 进度和必要控制文档。Use when updating project state after code changes, marking checklist progress, recording tests and known issues, deciding whether ARCHITECTURE/DATAFLOW/MODULE_CONTRACTS/EVAL_AND_TESTING/DECISIONS need updates, or keeping the human owner aligned with the real codebase.
---

# Skill: project-state-updater

## Purpose

在每次 AI coding 完成后，同步项目状态、开发清单进度和必要控制文档。

这个 skill 的目标是防止：

```text
代码已经变化，但人类 owner 脑子里的项目模型没有同步更新。
```

它应该帮助人类 owner 清楚知道：

1. 本次实际改了什么。
2. 哪些开发清单已完成。
3. 主流程是否变化。
4. 核心数据结构是否变化。
5. 模块边界是否变化。
6. 测试和评估方式是否变化。
7. 下一步应该做什么。

## When to Use

在以下场景调用：

- 每次 AI 完成一个 checklist item 后。
- 每次实现一个功能后。
- 每次重构后。
- 每次修 bug 后，如果影响了主流程、数据结构、模块契约或测试。
- 每次合并较大变更前。

## Required Inputs

调用时请提供：

```text
1. 本次变更目标
2. 对应的 plans/*.md 文件
3. 本次实际修改的文件列表
4. 测试结果
5. 是否有失败项
6. 是否有架构或数据流变化
```

如果调用者没有提供这些信息，需要通过代码 diff、git status、测试输出来推断，但必须标记不确定项。

## Hard Rules

- 不要实现新功能。
- 不要顺手重构。
- 不要伪造测试结果。
- 不要把未完成任务勾选为完成。
- 不要更新无关文档。
- 文档必须反映当前真实代码，而不是理想设计。
- 如果代码和文档不一致，明确标记 `Out of Sync`。

## Output Files

通常更新：

```text
docs/PROJECT_STATE.md
plans/对应开发清单.md
```

必要时更新：

```text
docs/ARCHITECTURE.md
docs/DATAFLOW.md
docs/MODULE_CONTRACTS.md
docs/EVAL_AND_TESTING.md
docs/DECISIONS.md
```

## Procedure

### Step 1: Inspect the Change

先总结本次变更：

```text
变更目标：
实际完成：
未完成：
修改文件：
新增文件：
删除文件：
测试结果：
已知问题：
```

### Step 2: Decide Which Docs Need Updates

输出一个影响判断表：

```md
| Document | Update Needed? | Reason |
|---|---:|---|
| docs/PROJECT_STATE.md | Yes | 每次 coding 后必须更新 |
| plans/xx.md | Yes | 对应 checklist 进度变化 |
| docs/ARCHITECTURE.md | Yes/No | 是否改变模块结构 |
| docs/DATAFLOW.md | Yes/No | 是否改变数据流 |
| docs/MODULE_CONTRACTS.md | Yes/No | 是否改变模块职责或接口 |
| docs/EVAL_AND_TESTING.md | Yes/No | 是否改变测试/评估方式 |
| docs/DECISIONS.md | Yes/No | 是否做出重要技术决策 |
```

只有 `Yes` 的文档才更新。

### Step 3: Update Checklist Progress

在对应的 `plans/*.md` 中：

- 勾选已完成任务。
- 不勾选部分完成任务。
- 对部分完成项添加说明。
- 对偏离计划的实现添加 `Deviation Note`。
- 对新增任务添加 `Follow-up Task`。

格式：

```md
- [x] Task name
  - Completed in:
  - Verification:
  - Notes:

- [ ] Task name
  - Status: partial / blocked / not started
  - Reason:
  - Next action:
```

如果实现内容不在原计划中，添加：

```md
## Implementation Deviations

| Deviation | Reason | Risk | Follow-up |
|---|---|---|---|
| | | | |
```

### Step 4: Update PROJECT_STATE.md

`docs/PROJECT_STATE.md` 至少维护以下结构：

```md
# Project State

## One-Sentence Summary

## Current Capability

列出当前已经真实可用的能力。

## Current Non-Capability

列出尚未完成、容易被误以为已完成的能力。

## Current Main Entry

程序入口、CLI 命令、API 入口或主函数。

## Current Main Flow

用简洁链路表示：

```text
input -> step A -> step B -> output
```

## Core Data Structures

| Name | Purpose | Created By | Consumed By | Notes |
|---|---|---|---|---|

## Module Map

| Module/File | Responsibility | Important Exports | Notes |
|---|---|---|---|

## External Dependencies

| Dependency | Used By | Purpose | Failure Mode |
|---|---|---|---|

## Current Test Commands

```bash
...
```

## Current Evaluation Method

说明如何判断系统效果，而不仅仅是代码能运行。

## Current Logging / Observability

说明有哪些日志、trace、debug 输出。

## High-Risk Areas

| Area | Risk | Why It Matters |
|---|---|---|

## Files Safe to Modify

## Files Requiring Caution

## Current Development Stage

## Last Completed Checklist Item

## Next Recommended Step

## Last Update

- Date:
- Change summary:
- Tests:
- Known issues:

## Human Control Checklist

- [ ] I know where the main flow starts and ends.
- [ ] I know the core data structures.
- [ ] I know the module boundaries.
- [ ] I know how to test the current system.
- [ ] I know where to look when something breaks.
```

### Step 5: Update Architecture Docs If Needed

#### Update ARCHITECTURE.md only if:

- 新增/删除模块。
- 改变模块层次。
- 改变主 pipeline。
- 新增重要外部依赖。
- 改变部署或运行方式。

#### Update DATAFLOW.md only if:

- 核心数据对象变化。
- 数据流经阶段变化。
- 输入输出结构变化。
- 新增缓存、队列、数据库、向量库等状态系统。

#### Update MODULE_CONTRACTS.md only if:

- 模块职责变化。
- 函数/类接口变化。
- 某模块新增副作用。
- 某模块开始依赖新的下游模块。

#### Update EVAL_AND_TESTING.md only if:

- 新增测试命令。
- 新增 golden cases。
- 修改评估指标。
- 改变测试运行方式。
- 已知缺口变化。

#### Update DECISIONS.md only if:

- 做出重要技术选型。
- 引入不可轻易替换的依赖。
- 改变架构方向。
- 接受某个重要 trade-off。

### Step 6: Produce a Human-Friendly Summary

最后输出：

```text
本次状态更新完成。

实际完成：
- ...

未完成：
- ...

更新的文档：
- ...

没有更新的文档及原因：
- ...

测试结果：
- ...

当前风险：
- ...

建议下一步：
- ...
```

## Completion Criteria

只有满足以下条件，才算完成：

- 对应 checklist 状态准确。
- PROJECT_STATE.md 反映真实代码状态。
- 没有把理想设计写成当前能力。
- 明确说明测试是否真实运行。
- 明确说明当前风险。
- 明确下一步。
