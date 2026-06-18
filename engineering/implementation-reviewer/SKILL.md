---
name: implementation-reviewer
description: >-
  在 AI 完成代码实现、feature、重构、bug fix 或 checklist item 后，审查实现是否偏离计划、破坏模块边界、引入重复逻辑、产生隐藏副作用、增加不必要文件、缺少测试日志评估或需要更新文档。Use for post-implementation review before accepting changes, merging, moving to the next checklist item, or calling project-state-updater.
---

# Skill: implementation-reviewer

## Purpose

在 AI 完成代码实现后，审查本次实现是否保持了项目的可理解性、可测试性和模块边界。

这个 skill 的目标不是挑语法小错，而是防止：

```text
局部代码看似合理，但整体系统质量不可验证。
```

重点检查：

1. 是否偏离开发清单。
2. 是否破坏模块边界。
3. 是否引入重复逻辑。
4. 是否产生隐藏副作用。
5. 是否新增不必要文件。
6. 是否缺少测试、日志或评估。
7. 是否需要更新项目状态文档。

## When to Use

在以下场景调用：

- AI 完成一个 checklist item 后。
- AI 完成一个 feature 后。
- 合并代码前。
- 重构后。
- 你感觉“代码好像能跑，但我说不清为什么能跑”时。
- 项目文件数或复杂度突然增加时。

## Required Inputs

调用时请提供：

```text
1. 本次实现目标
2. 对应 plans/*.md 或 feature plan
3. 本次修改的 diff
4. 测试结果
5. 当前 PROJECT_STATE.md
6. 当前 MODULE_CONTRACTS.md
```

如果没有 diff，至少检查当前代码与计划的差异。

## Hard Rules

- 不要直接修改代码，除非用户明确要求修复。
- 审查必须基于当前代码事实。
- 不要只说“看起来不错”。
- 必须给出是否通过的结论。
- 必须区分 blocker、warning、suggestion。
- 不要要求完美主义式重构。
- 优先保护主流程、数据结构和模块边界。

## Output Files

默认不修改文件。

如果用户要求记录审查结果，可以生成：

```text
docs/REVIEW_REPORT_YYYYMMDD.md
```

如果发现文档需要更新，只提出建议，不自动更新，除非用户要求调用 `project-state-updater`。

## Procedure

### Step 1: Restate Implementation Scope

输出：

```text
本次目标：
对应计划：
声明的修改范围：
实际修改范围：
测试结果：
```

### Step 2: Check Plan Alignment

对照开发清单：

```md
| Checklist Item | Expected | Actual | Status |
|---|---|---|---|
```

Status：

```text
pass
partial
missing
deviated
extra
```

重点识别：

```text
是否做了计划外功能？
是否跳过了计划内任务？
是否改动了不该改的文件？
```

### Step 3: Check Module Boundaries

检查：

```text
模块是否只做自己的事？
pipeline 是否只负责编排？
service 是否包含外部 API 细节？
adapter 是否泄漏业务逻辑？
prompt 是否混入 Python 控制流？
evaluator 是否修改了 pipeline state？
retriever 是否生成了最终答案？
工具函数是否出现隐藏副作用？
```

输出：

```md
| Module | Expected Responsibility | Observed Behavior | Issue |
|---|---|---|---|
```

### Step 4: Check Data Flow and Data Contracts

检查：

```text
核心数据结构是否稳定？
是否到处传裸 dict？
是否出现同一概念多个结构？
字段命名是否一致？
谁创建对象、谁修改对象、谁消费对象是否清楚？
是否存在隐式状态？
```

输出：

```md
| Data Object | Expected Flow | Observed Flow | Risk |
|---|---|---|---|
```

### Step 5: Check Complexity and File Growth

检查：

```text
是否新增太多文件？
是否文件职责过小或过碎？
是否一个文件职责过宽？
是否出现重复代码？
是否有可以合并的模块？
是否有过度抽象？
是否有未来扩展名义下的提前复杂化？
```

输出：

```md
| Complexity Issue | Location | Severity | Recommendation |
|---|---|---|---|
```

### Step 6: Check Side Effects and Failure Modes

检查：

```text
是否有隐藏文件写入？
是否有隐藏网络调用？
是否有全局状态修改？
是否有异常被吞掉？
是否失败时无法定位？
是否外部依赖失败时没有 fallback 或明确错误？
```

输出：

```md
| Failure Mode | Location | Current Handling | Risk |
|---|---|---|---|
```

### Step 7: Check Tests and Evaluation

检查：

```text
是否新增或更新测试？
测试是否覆盖本次变更？
是否有 smoke test？
是否有集成测试？
是否有 golden case？
是否只是测试 mock，没有测试主流程？
是否有手动验证命令？
```

输出：

```md
| Test Area | Present? | Evidence | Gap |
|---|---:|---|---|
```

### Step 8: Check Documentation Sync

判断哪些文档需要更新：

```md
| Document | Needs Update? | Reason |
|---|---:|---|
| docs/PROJECT_STATE.md | Yes/No | |
| plans/xx.md | Yes/No | |
| docs/ARCHITECTURE.md | Yes/No | |
| docs/DATAFLOW.md | Yes/No | |
| docs/MODULE_CONTRACTS.md | Yes/No | |
| docs/EVAL_AND_TESTING.md | Yes/No | |
| docs/DECISIONS.md | Yes/No | |
```

### Step 9: Produce Review Verdict

必须给出明确结论：

```text
Verdict: PASS / PASS_WITH_WARNINGS / NEEDS_CHANGES / BLOCKED
```

定义：

```text
PASS:
实现符合计划，测试足够，文档只需常规更新。

PASS_WITH_WARNINGS:
可以继续，但存在小风险，需要记录。

NEEDS_CHANGES:
不建议进入下一阶段，需要先修复部分问题。

BLOCKED:
主流程、数据结构、模块边界或测试存在严重问题，不能继续。
```

### Step 10: Give Minimal Fix Plan

如果不是 PASS，给出最小修复计划：

```md
## Minimal Fix Plan

- [ ] Fix 1
  - Why:
  - File:
  - Acceptance:
- [ ] Fix 2
  - Why:
  - File:
  - Acceptance:
```

不要提出大规模重构，除非确实必要。

## Review Report Template

最终输出使用：

```md
# Implementation Review

## Verdict

PASS / PASS_WITH_WARNINGS / NEEDS_CHANGES / BLOCKED

## Summary

## Plan Alignment

| Item | Status | Notes |
|---|---|---|

## Module Boundary Issues

| Issue | Location | Severity | Recommendation |
|---|---|---|---|

## Data Flow / Data Contract Issues

| Issue | Location | Severity | Recommendation |
|---|---|---|---|

## Complexity Issues

| Issue | Location | Severity | Recommendation |
|---|---|---|---|

## Testing Gaps

| Gap | Severity | Recommendation |
|---|---|---|

## Documentation Updates Needed

| Document | Needed? | Reason |
|---|---:|---|

## Minimal Fix Plan

## Next Step

是否可以调用 `project-state-updater`：

Yes / No

原因：
```

## Quality Bar

只有满足以下条件，才算完成：

- 有明确 verdict。
- 对照了开发清单。
- 检查了模块边界。
- 检查了数据流。
- 检查了复杂度。
- 检查了测试。
- 检查了文档同步。
- 给出最小修复方案。
