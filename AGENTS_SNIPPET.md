# AGENTS.md 可复制规则片段

你可以把下面的短规则复制到目标项目根目录的 `AGENTS.md` 中。

```md
# AI Coding Workflow Rules

## 基本原则

- 优先保持系统简单、可理解、可测试、可维护。
- 不要一次实现过大范围；每次只处理一个明确的 checklist 或 feature。
- 修改前先说明会改哪些文件、不会改哪些文件、如何验证。
- 不确定、需求有歧义或改动影响过大时，先停下来询问。

## Skill 使用约定

- 新项目拆 plan：使用 `plan-decomposer`，生成 `plans/*.md` 和 `docs/PROJECT_STATE.md`，不要直接写代码。
- 接手已有项目：使用 `architecture-auditor` 做只读审计，先恢复主流程、数据流和模块边界。
- 新增功能前：使用 `feature-impact-analyzer` 分析影响范围、最小方案、测试方式和回滚方案。
- 实现完成后：使用 `implementation-reviewer` 审查是否偏离计划、破坏边界或缺少测试。
- 审查通过后：使用 `project-state-updater` 同步 `docs/PROJECT_STATE.md` 和对应 `plans/*.md`。

## 编码边界

- 不要顺手重构无关代码。
- 不要新增不必要文件或提前抽象。
- 不要把外部 API 细节泄漏到 domain 层。
- 不要把 prompt、pipeline 编排和业务逻辑混在一起。
- 优先使用明确的数据结构，避免到处传裸 `dict`。
- 工具函数不要隐藏文件写入、网络请求、全局状态修改等副作用。

## 测试与验证

每个非平凡改动必须提供至少一种验证方式：

- unit test
- integration test
- smoke test
- golden case
- 手动验证命令

如果没有新增测试，必须说明原因和风险。

## 文档同步

只更新受影响的文档：

- 每次接受实现后，更新 `docs/PROJECT_STATE.md`。
- checklist 进度变化时，更新对应 `plans/*.md`。
- 架构、数据流、模块契约、测试策略或重大决策变化时，再更新对应文档。

## 停止条件

遇到以下情况，先停止并请求确认：

- 单次改动预计超过 5 个文件
- 核心数据结构或公开接口要改变
- 需要引入新依赖或新抽象
- 测试失败且原因不明确
- 当前代码与控制文档明显不一致
```
