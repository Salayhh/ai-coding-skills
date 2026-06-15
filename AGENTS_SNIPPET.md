# AGENTS.md 可复制规则片段

你可以把下面内容复制到项目根目录的 `AGENTS.md` 中。

```md
# AI Coding Project Rules

## Core Principle

Do not optimize for producing more code. Optimize for a simple, understandable, testable, and maintainable system.

The human owner must always be able to answer:

1. Where does the main flow start and end?
2. What are the core data structures?
3. Which modules own which responsibilities?
4. What is the impact scope of a new feature?
5. How can we test, log, and evaluate whether the system still works?

## Working Modes

### Plan Mode

When asked to plan, do not implement code.

A valid plan must include:

- project goal
- non-goals
- main flow
- core data structures
- module boundaries
- development stages
- testing and evaluation strategy
- risks and rollback strategy

After producing a plan, invoke or follow `skills/plan-decomposer/SKILL.md` to convert it into executable checklist files.

### Implementation Mode

Before coding, state:

1. Which checklist item is being implemented.
2. Which files will be changed.
3. Which files will not be touched.
4. Which data structures are affected.
5. Which tests will be added or updated.

Only implement one small checklist section at a time.

### Review Mode

After coding, invoke or follow `skills/implementation-reviewer/SKILL.md`.

Review for:

- deviation from plan
- unclear module ownership
- hidden side effects
- duplicated logic
- excessive file creation
- missing tests
- missing logs
- outdated documentation

### State Update Mode

After each accepted implementation, invoke or follow `skills/project-state-updater/SKILL.md`.

Update:

- `docs/PROJECT_STATE.md`
- the corresponding `plans/*.md` checklist

Update other docs only if affected:

- `docs/ARCHITECTURE.md`
- `docs/DATAFLOW.md`
- `docs/MODULE_CONTRACTS.md`
- `docs/EVAL_AND_TESTING.md`
- `docs/DECISIONS.md`

## Architecture Rules

- Keep pipeline orchestration separate from business logic.
- Keep adapters separate from domain logic.
- Keep prompts separate from Python control flow.
- Prefer typed data models over unstructured dictionaries.
- Do not let retrievers generate final answers.
- Do not let evaluators mutate pipeline state unexpectedly.
- Do not hide side effects inside utility functions.
- Do not create new files unless the responsibility cannot fit clearly into an existing module.
- Do not duplicate logic across modules.

## Testing Rules

Every non-trivial change must include at least one of:

- unit test
- integration test
- smoke test
- golden query regression test
- manual verification command

If no test is added, explain why and document the risk.

## Documentation Rules

Documentation is part of the codebase.

When architecture changes, update `docs/ARCHITECTURE.md`.

When data flow changes, update `docs/DATAFLOW.md`.

When module ownership changes, update `docs/MODULE_CONTRACTS.md`.

When testing or evaluation changes, update `docs/EVAL_AND_TESTING.md`.

When a major irreversible technical decision is made, update `docs/DECISIONS.md`.

## Stop Conditions

Stop and ask for review if:

- a change affects more than 5 files
- a new abstraction is introduced
- a core data model changes
- a public interface changes
- tests fail
- the current architecture no longer matches the documentation
```
