# AI Coding Personal Workflow Skills

这是一套用于 AI Coding 的个人开发控制流 Codex skill 包。

## Background

Coding agent 已经让个人开发效率大幅提升：从简单脚本、单功能工具，到稍复杂的 pipeline，AI 都能快速生成可运行代码。瓶颈通常不是 AI 不会写，而是当项目进入更复杂的系统形态后，代码增长速度会超过人的心智建模速度。

即使有 plan mode 和 code review，开发者对项目的工程控制力仍会随复杂度提高而衰减：你很难快速判断当前系统是一个简洁、优雅、强鲁棒、有工程冗余的设计，还是一个临时拼凑、边界 case 一碰就垮的庞然大物。"把许多局部看似合理的代码，拼成无法验证整体质量的系统"，这种情况并不罕见。

更具体地说，开发者常常会失去掌控感：

- 不知道主流程从哪里开始，到哪里结束。
- 不知道核心数据结构是什么，在哪些模块之间流动。
- 不知道每个模块到底负责什么、不负责什么。
- 不知道新增一个功能会影响哪些文件、接口、测试和文档。
- 不知道系统出问题该看测试、日志还是评估集，还是只能凭猜测排查。
- 不知道当前代码是简洁可维护的系统，还是勉强跑起来的复杂堆叠。

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
  AGENTS_SNIPPET.en.md
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
请使用 architecture-auditor，对当前代码库做只读架构审计。禁止修改代码。
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

把 `AGENTS_SNIPPET.md` 的内容放进目标项目的 `AGENTS.md`，让 coding agent 默认遵守这套工作流。

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

## Validation

当前 5 个 skill 已通过 Codex `skill-creator` 的基础校验：

```text
Skill is valid!
```

校验覆盖 YAML frontmatter、必需字段和命名规则。
