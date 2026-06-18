---
name: feature-impact-analyzer
description: >-
  在新增功能、修改核心流程、引入外部依赖或增加 agent/self-correction/rerank/cache/memory 等复杂机制前，分析功能对主流程、数据结构、模块、测试、文档和风险的影响。Use before implementation when the user wants the smallest viable design, impact scope, files that should not change, feature checklist, test plan, evaluation method, rollback plan, or human approval gate.
---

# Skill: feature-impact-analyzer

## Purpose

在新增功能前，分析该功能会影响哪些主流程、数据结构、模块、测试、文档和风险点。

这个 skill 的目标是防止 AI coding 中最常见的失控模式：

```text
用户要求加一个小功能，AI 顺手修改一堆文件，破坏原有架构边界。
```

## When to Use

在以下场景调用：

- 新增功能前。
- 修改核心流程前。
- 引入新外部依赖前。
- 增加 agent / self-correction / rerank / cache / memory 等复杂机制前。
- 不确定某个需求是否会牵动很多模块时。
- 想找“更小实现方案”时。

## Required Inputs

调用时请提供：

```text
1. 目标功能
2. 当前项目状态文档：docs/PROJECT_STATE.md
3. 当前架构文档：docs/ARCHITECTURE.md
4. 当前数据流文档：docs/DATAFLOW.md
5. 当前模块契约：docs/MODULE_CONTRACTS.md
6. 相关开发清单，如果有
```

如果控制文档不存在，先做轻量审计，不要直接写代码。

## Hard Rules

- 不要写代码。
- 不要修改代码。
- 不要直接创建文件。
- 必须先判断该功能是否真的需要新增模块。
- 必须给出最小实现方案。
- 必须给出影响范围。
- 必须给出测试方式。
- 必须给出回滚方案。
- 如果需求过大，必须拆成多个更小阶段。

## Output Files

通常生成或更新：

```text
plans/FEATURE_*.md
```

可选更新：

```text
docs/DECISIONS.md
```

但只有在人类确认方案后，才应更新架构类文档。

## Procedure

### Step 1: Restate the Feature

先用工程语言重述需求：

```text
原始需求：
工程化解释：
预期用户价值：
成功标准：
非目标范围：
```

### Step 2: Locate Feature in Main Flow

必须说明功能插入主流程的哪个位置。

格式：

```text
当前主流程：
A -> B -> C -> D

功能插入位置：
A -> B -> [New Feature] -> C -> D
```

如果功能横跨多条流程，分别说明。

### Step 3: Identify Affected Data Structures

输出：

```md
| Data Structure | Change Needed? | Change Type | Reason | Risk |
|---|---:|---|---|---|
| Document | No | - | - | - |
| Query | Yes | add field | need xxx | medium |
```

Change Type 可为：

```text
none
add field
remove field
rename field
new structure
schema migration
behavior change
```

必须特别说明：

```text
是否需要改变核心数据结构？
是否可以通过局部结构避免污染核心模型？
是否会破坏已有调用方？
```

### Step 4: Identify Affected Modules

输出：

```md
| Module/File | Change Needed? | Expected Change | Why | Risk |
|---|---:|---|---|---|
```

必须标记：

```text
必须修改
可能修改
不应修改
```

尤其要列出：

```text
不应该被这个功能影响的模块。
```

### Step 5: Check Module Boundary Fit

回答：

```text
这个功能应该属于哪个模块？
是否需要新模块？
是否可以放进现有模块？
会不会导致某个模块职责膨胀？
会不会引入反向依赖？
会不会让 adapter 泄漏到 domain 层？
会不会让 prompt 和业务逻辑混在一起？
```

### Step 6: Propose Implementation Options

至少给出 2 个方案：

```md
## Option A: Minimal Implementation

优点：
缺点：
涉及文件：
适合情况：

## Option B: More Extensible Implementation

优点：
缺点：
涉及文件：
适合情况：
```

必要时给出：

```md
## Option C: Do Not Implement Yet

原因：
替代方案：
前置条件：
```

### Step 7: Recommend One Option

必须给出明确推荐：

```text
推荐方案：
推荐理由：
不推荐其他方案的原因：
```

原则：

```text
优先选择最小可验证方案。
除非已有明确扩展需求，否则不要过度工程化。
```

### Step 8: Generate Feature Checklist

生成 `plans/FEATURE_xxx.md`：

```md
# Feature Plan: 功能名称

## 1. Feature Goal

## 2. Non-Goals

## 3. Main Flow Insertion Point

```text
before -> feature -> after
```

## 4. Affected Data Structures

| Data Structure | Change | Reason |
|---|---|---|

## 5. Affected Modules

| Module/File | Change | Risk |
|---|---|---|

## 6. Files That Should Not Be Changed

| File/Module | Reason |
|---|---|

## 7. Implementation Steps

- [ ] Step 1
  - Files:
  - Expected behavior:
  - Test:
  - Acceptance:
- [ ] Step 2
  - Files:
  - Expected behavior:
  - Test:
  - Acceptance:

## 8. Tests

## 9. Logging / Observability

## 10. Evaluation

如何证明这个功能让系统变好，而不只是变复杂。

## 11. Rollback Plan

## 12. Risks

## 13. Documentation Updates

- [ ] docs/PROJECT_STATE.md
- [ ] docs/ARCHITECTURE.md
- [ ] docs/DATAFLOW.md
- [ ] docs/MODULE_CONTRACTS.md
- [ ] docs/EVAL_AND_TESTING.md
- [ ] docs/DECISIONS.md

## 14. Human Approval Gate

在开始实现前，人类 owner 必须确认：

- [ ] 我知道这个功能插入主流程的哪里。
- [ ] 我知道会改哪些数据结构。
- [ ] 我知道会改哪些模块。
- [ ] 我知道哪些模块不应该被改。
- [ ] 我知道如何测试。
- [ ] 我知道如何回滚。
```

### Step 9: Produce Final Summary

输出：

```text
功能影响分析完成。

推荐方案：
...

影响范围：
- 数据结构：
- 模块：
- 测试：
- 文档：

不应修改：
- ...

建议拆分步骤：
1. ...
2. ...

是否建议现在实现：
是/否，原因：
```

## Quality Bar

只有满足以下条件，才算完成：

- 明确功能插入主流程位置。
- 明确影响数据结构。
- 明确影响模块。
- 明确不应修改的模块。
- 给出至少两个方案。
- 推荐最小可验证方案。
- 生成可执行 feature checklist。
- 给出测试和回滚方式。
