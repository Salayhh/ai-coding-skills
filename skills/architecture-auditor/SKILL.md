---
name: architecture-auditor
description: >-
  对已有或 AI 生成的复杂项目做只读架构审计，恢复人类 owner 对入口、主流程、核心数据结构、模块边界、文件分级、测试评估缺口和风险区域的掌控。Use when taking over a codebase, auditing a project before refactor or feature work, generating docs/ARCHITECTURE.md, docs/DATAFLOW.md, docs/MODULE_CONTRACTS.md, docs/EVAL_AND_TESTING.md, docs/PROJECT_STATE.md, or determining whether a project is understandable and maintainable.
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
- 需要判断项目是简洁系统还是屎山雏形。

## Required Inputs

调用时请提供：

```text
1. 项目路径
2. 项目目标或你认为它应该做什么
3. 运行方式，如果已知
4. 技术栈，如果已知
5. 你最困惑的问题
```

如果缺失，仍然继续做只读审计，但标记不确定项。

## Hard Rules

- 禁止修改代码。
- 禁止重构。
- 禁止删除文件。
- 禁止安装依赖，除非用户明确允许。
- 禁止编造不存在的测试结果。
- 优先还原事实，再提出建议。
- 如果推断不确定，标记 `Uncertain`。
- 不要只输出文件列表，必须还原主流程、数据流、模块边界。

## Output Files

生成或更新：

```text
docs/ARCHITECTURE.md
docs/DATAFLOW.md
docs/MODULE_CONTRACTS.md
docs/EVAL_AND_TESTING.md
docs/PROJECT_STATE.md
docs/AUDIT_REPORT.md
```

可选：

```text
docs/FILE_CLASSIFICATION.md
docs/RISK_REGISTER.md
```

## Procedure

### Step 1: Discover Project Structure

扫描：

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

输出：

```text
项目类型：
可能入口：
主要包/模块：
测试位置：
配置位置：
外部依赖：
```

### Step 2: Find Entry Points

必须找出：

```text
CLI 入口
Web/API 入口
main 函数
pipeline 入口
测试入口
配置加载入口
```

输出表：

```md
| Entry Point | File | How It Starts | Downstream Calls | Confidence |
|---|---|---|---|---|
| | | | | High/Medium/Low |
```

### Step 3: Reconstruct Main Flows

至少重建两类流程：

```text
主业务流程
辅助流程，如测试、评估、批处理、入库、迁移
```

例如 RAG 项目：

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

### Step 4: Identify Core Data Structures

找出：

```text
dataclass
pydantic model
TypedDict
ORM model
plain dict schema
主要函数间传递的数据
```

输出：

```md
| Data Structure | File | Purpose | Created By | Modified By | Consumed By | Risk |
|---|---|---|---|---|---|---|
```

如果系统大量使用裸 dict，标记为风险：

```text
Risk: implicit schema / unclear data contract
```

### Step 5: Build Module Responsibility Map

每个核心文件输出：

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

### Step 6: Classify Files

将文件分为：

```text
A 类：核心主流程文件，必须理解
B 类：重要服务模块，需要知道职责
C 类：adapter / 工具文件，知道输入输出即可
D 类：测试、脚本、样例，可稍后看
E 类：疑似废弃、重复或风险文件
```

输出：

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

### Step 7: Assess Test and Evaluation Coverage

找出：

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

输出：

```md
| Area | Covered? | Evidence | Gap |
|---|---:|---|---|
```

### Step 8: Identify Risk Areas

重点风险：

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

输出：

```md
| Risk | Location | Why It Matters | Severity | Suggested Next Step |
|---|---|---|---|---|
```

### Step 9: Generate Control Docs

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

## Current Gaps
```

#### PROJECT_STATE.md

必须包含：

```md
# Project State

## One-Sentence Summary

## Current Capability

## Current Non-Capability

## Current Main Entry

## Current Main Flow

## Core Data Structures

## Module Map

## Current Test Commands

## Current Logging / Observability

## High-Risk Areas

## Files Safe to Modify

## Files Requiring Caution

## Next Recommended Step
```

#### AUDIT_REPORT.md

必须包含：

```md
# Architecture Audit Report

## Executive Summary

## What Is Clear

## What Is Unclear

## Main Flow Reconstruction

## Core Risks

## File Classification Summary

## Immediate Stabilization Plan

## Recommended Refactor Plan

## Questions for Human Owner
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

建议下一步：
...
```

## Quality Bar

只有满足以下条件，才算完成：

- 明确主入口。
- 明确至少一条主流程。
- 明确核心数据结构。
- 明确模块职责。
- 给出文件分级。
- 给出测试/评估缺口。
- 给出高风险区域。
- 没有修改代码。
