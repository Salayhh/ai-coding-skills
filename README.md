# AI Coding Personal Workflow Skills

这是一套用于 AI Coding 的个人开发控制流 Codex skill 包。

## Background

Coding agent 已经让个人开发效率大幅提升：从简单脚本、单功能工具，到稍复杂的 pipeline，AI 都能快速生成可运行代码。瓶颈通常不是 AI 不会写，而是当项目进入更复杂的系统形态后，代码增长速度会超过人对项目的认知构建速度。

更具体地说，开发者常常会失去掌控感：

- 不知道主流程从哪里开始，到哪里结束。
- 不知道核心数据结构是什么，在哪些模块之间流动。
- 不知道每个模块到底负责什么、不负责什么。
- 不知道新增一个功能会影响哪些文件、接口、测试和文档。
- 不知道系统出问题该看测试、日志还是评估集，还是只能凭猜测排查。
- 不知道当前代码是简洁可维护的系统，还是勉强跑起来的复杂堆叠。

即使有 plan mode 和 code review，开发者对项目的工程控制力仍会随复杂度提高而衰减：你很难快速判断当前系统是一个简洁、可维护、强鲁棒、有工程冗余的设计，还是一个临时拼凑、边界 case 一碰就垮的庞然大物。"把许多局部看似合理的代码，拼成无法验证整体质量的系统"，这种情况并不罕见。

我们并不希望 AI 给出 plan 后一次性完成整个项目。即便代码能跑通，事后的审查与掌控感重建也足以让人痛苦；如果跳过审查、继续追加功能，其隐藏的技术债务会以滚雪球的方式，在某天给你一个巨大的惊喜。

这套 skill 的出发点是：**AI coding 需要的不只是更强的代码生成能力，还需要一套工程控制面。** 它把抽象规划、代码实现、审查、状态更新和接手审计拆成明确的步骤，让每次 AI 写代码之后，人对系统的理解不是变少，而是变多。

## Goal

目标是：

> 让 AI 负责生成和修改代码，但让人始终掌握系统主流程、核心数据结构、模块边界、变更影响范围、测试评估方式和当前项目状态。

## Use Cases

它适合两类场景：

1. 从 0 开发项目：把 coding agent 给出的抽象 plan 拆成可执行、可验收、可追踪的开发清单。
2. 接手已有项目：通过只读审计快速恢复对入口、主流程、数据流、模块职责和风险区域的理解。

## Skills

| Skill | 用途 |
|---|---|
| `plan-decomposer` | 把抽象 plan 拆成分阶段开发清单，并初始化 `docs/PROJECT_STATE.md` |
| `project-state-updater` | 每次 coding 后更新项目状态、开发清单进度和必要控制文档 |
| `architecture-auditor` | 接手项目时做只读架构审计，生成架构、数据流、模块契约和测试评估文档 |
| `feature-impact-analyzer` | 新增功能前分析影响范围、最小实现方案、测试方式和回滚方案 |
| `implementation-reviewer` | 实现后审查是否偏离计划、破坏模块边界、缺少测试或文档不同步 |

## Repository Structure

```text
ai_coding_skills/
  README.md
  AGENTS_SNIPPET.md
  skills/
    plan-decomposer/
      SKILL.md
      agents/openai.yaml
    project-state-updater/
      SKILL.md
      agents/openai.yaml
    architecture-auditor/
      SKILL.md
      agents/openai.yaml
    feature-impact-analyzer/
      SKILL.md
      agents/openai.yaml
    implementation-reviewer/
      SKILL.md
      agents/openai.yaml
```

每个 skill 都是标准 Codex Skill 目录，包含：

- `SKILL.md`：必需的 skill 主文件，包含 YAML frontmatter 和执行流程。
- `agents/openai.yaml`：推荐的 UI 元数据，包含显示名称、简短说明和默认调用提示。

## Install

如果希望 Codex 全局发现这些 skill，把 `skills/` 下的 5 个目录放进 Codex skills 目录：

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R skills/* "${CODEX_HOME:-$HOME/.codex}/skills/"
```

安装后可以直接按 skill 名称调用：

```text
请使用 plan-decomposer，基于当前 plan 生成可执行开发清单。不要写代码。
```

```text
/architecture-auditor
```

## Use In A Project

建议目标项目采用以下结构：

```text
project-root/
  AGENTS.md
  docs/
    PROJECT_BRIEF.md
    PROJECT_STATE.md
    ARCHITECTURE.md
    DATAFLOW.md
    MODULE_CONTRACTS.md
    EVAL_AND_TESTING.md
    DECISIONS.md
  plans/
    00_MASTER_PLAN.md
    01_PROJECT_SCAFFOLD.md
    02_CORE_DATA_MODELS.md
    03_MINIMAL_VERTICAL_SLICE.md
```

如果希望 coding agent 在目标项目中默认遵守这套工作流，可以把 `AGENTS_SNIPPET.md` 里的精简规则复制到目标项目的 `AGENTS.md`。这不是必需步骤；也可以在需要时手动调用对应 skill。

如果这些 skill 没有全局安装，只是放在项目根目录，可以显式指定 `SKILL.md`：

```text
请调用 skills/plan-decomposer/SKILL.md，基于当前 plan 生成可执行开发清单。不要写代码。
```

```text
请调用 skills/project-state-updater/SKILL.md，基于本次代码变更更新 PROJECT_STATE.md 和对应 plans 文件。
```

```text
请调用 skills/feature-impact-analyzer/SKILL.md，分析新增 XXX 功能会影响哪些模块、数据结构、测试和文档。不要写代码。
```

```text
请调用 skills/implementation-reviewer/SKILL.md，审查本次实现是否偏离开发清单、破坏模块边界或缺少测试。
```

## Examples

### 1. 从 0 开始拆解一个新项目

适合在 coding agent 已经给出一个抽象方案后使用。目标是先得到可执行清单，而不是直接写代码。

```text
请使用 plan-decomposer。不要写代码。

项目目标：
做一个本地运行的 RAG 问答工具，支持导入 markdown 文档，然后基于文档回答问题。

初始 plan：
1. 创建 Python 项目结构
2. 实现文档读取、切分、embedding、向量入库
3. 实现查询、检索、答案生成
4. 增加简单评估和日志

技术栈：
Python，命令行工具，向量库暂定 Chroma，配置放在 .env。

MVP 范围：
- 支持本地 markdown 文件
- 支持单轮问答
- 支持 smoke test

非目标范围：
- 不做 Web UI
- 不做用户系统
- 不做多轮记忆

请生成 plans/*.md 和 docs/PROJECT_STATE.md，并明确每个阶段的验收标准。
```

### 2. 接手一个已经变复杂的项目

适合项目文件变多、你不确定主流程和模块边界时使用。

```text
/architecture-auditor
```

默认只读审计当前代码库，并生成：

- `docs/ARCHITECTURE.md`
- `docs/DATAFLOW.md`
- `docs/MODULE_CONTRACTS.md`
- `docs/EVAL_AND_TESTING.md`

这四份文档需要共同回答：

- 主流程从哪里开始，到哪里结束。
- 核心数据结构是什么，在哪些模块之间流动。
- 每个模块的职责边界是什么。
- 哪些文件是核心文件，哪些文件可能可删或废弃。
- 加一个功能可能影响哪里。
- 如何用测试、日志、评估集判断项目有没有坏。

### 3. 新增功能前先分析影响范围

适合在加功能前控制改动范围，避免“小功能”牵动大面积文件。

```text
请使用 feature-impact-analyzer。不要写代码、不要修改文件。

目标功能：
给查询流程增加 rerank 步骤，在检索结果返回后、答案生成前，对候选 chunk 重新排序。

请基于以下文档分析影响范围：
- docs/PROJECT_STATE.md
- docs/ARCHITECTURE.md
- docs/DATAFLOW.md
- docs/MODULE_CONTRACTS.md
- plans/05_QUERY_PIPELINE.md

请重点回答：
1. rerank 应该插入主流程哪个位置？
2. 是否需要修改核心数据结构？
3. 哪些模块必须修改，哪些模块不应修改？
4. 最小实现方案是什么？
5. 需要哪些测试和 smoke test？
6. 如果效果不好，如何回滚？

最后请生成一个 plans/FEATURE_RERANK.md 草案，等待我确认后再实现。
```

### 4. 实现后做一次工程审查

适合在一个 checklist item、feature 或重构完成后使用。

```text
请使用 implementation-reviewer，审查本次实现。默认不要修改代码。

本次实现目标：
完成 plans/03_MINIMAL_VERTICAL_SLICE.md 中的最小主链路：
markdown 文件 -> chunk -> mock retrieval -> mock answer -> CLI 输出。

请检查：
1. 实现是否偏离 checklist？
2. 是否引入了计划外文件或复杂抽象？
3. pipeline、domain model、adapter 的边界是否清楚？
4. 是否出现重复逻辑或隐藏副作用？
5. 测试是否足够证明最小主链路可用？
6. 哪些文档需要在通过后更新？

可用信息：
- 对应计划：plans/03_MINIMAL_VERTICAL_SLICE.md
- 当前状态：docs/PROJECT_STATE.md
- 当前模块契约：docs/MODULE_CONTRACTS.md
- 测试结果：pytest 通过，python -m app --help 通过
```

### 5. 代码完成后同步项目状态

适合在审查通过后，把真实代码状态同步回控制文档。

```text
请使用 project-state-updater，同步本次变更后的项目状态。不要实现新功能。

本次变更目标：
完成最小 CLI 主链路。

对应计划：
plans/03_MINIMAL_VERTICAL_SLICE.md

本次实际修改：
- app/cli.py
- app/pipeline.py
- app/models.py
- tests/test_pipeline.py

测试结果：
- pytest 通过
- python -m app --input examples/demo.md --question "这份文档讲了什么？" 可以输出答案

请更新：
1. docs/PROJECT_STATE.md
2. plans/03_MINIMAL_VERTICAL_SLICE.md 的 checklist 进度
3. 如有必要，更新 docs/DATAFLOW.md 和 docs/MODULE_CONTRACTS.md

不要把未完成能力写成已完成。
```

### 6. 一次完整小步迭代

适合把一个功能从分析、实现、审查到状态同步串起来。

```text
第一步：请使用 feature-impact-analyzer，分析“给 CLI 增加 --json 输出参数”的影响范围。不要写代码。

第二步：等我确认方案后，只实现 plans/FEATURE_JSON_OUTPUT.md 中的最小方案。

第三步：实现完成后，请使用 implementation-reviewer 审查本次 diff，确认没有破坏现有文本输出。

第四步：审查通过后，请使用 project-state-updater 更新 PROJECT_STATE.md 和对应 feature plan。
```

## Workflow

从 0 开发：

```text
写 AGENTS.md
-> 进入 plan mode
-> 调用 plan-decomposer
-> 只实现一个 checklist 小节
-> 调用 implementation-reviewer
-> 调用 project-state-updater
-> 人类确认 PROJECT_STATE.md
-> 进入下一个小节
```

接手项目：

```text
冻结代码
-> 调用 architecture-auditor
-> 阅读控制文档和文件分级
-> 跑 smoke test
-> 新功能前调用 feature-impact-analyzer
-> 实现后调用 implementation-reviewer
-> 接受后调用 project-state-updater
```
