# Project Onboard

Set up or update Claude Code project settings for the current repository.

## Mode Detection

Check if `CLAUDE.md` exists in the project root.
- **If NOT found** → Run "Initial Setup" (Phase 1 → Phase 2 → Phase 3)
- **If found** → Run "Update" (Phase 2 → Phase 3)

---

## Phase 1: Gather User Input (Initial Setup Only)

Ask the user the following questions using AskUserQuestion. Do NOT skip this phase. Do NOT guess answers.

### Process Guidelines
- Even when only one option exists, present it and confirm with the user before applying.
- Show all auto-detected information transparently. Do not hide or filter scan results.
- Present risks alongside each option at decision time, not in a separate section after decisions.
- For pure technical decisions: present pros/cons with a recommendation, then let the user decide.

### Required Questions

1. **Project overview**: "What does this project do? (1-2 sentences)"
2. **Terminology**: "Are there terms in this project that are easily confused or have specific meanings? List them with correct vs incorrect usage. (Skip if none)"
3. **Abbreviations**: "What abbreviations does this project use? e.g., PRD, QA, API (Skip if none)"
4. **Custom prohibitions**: "Any project-specific rules Claude must follow beyond the defaults? (Skip if none)"

Store all answers for use in Phase 3.

---

## Phase 2: Auto-Detect from Codebase

Scan the project and collect the following. If the repo is empty (no source files), mark each as `EMPTY` and move on.

### 2.1 Programming Language(s)
- Check file extensions (`.ts`, `.py`, `.go`, `.rs`, `.java`, etc.)
- Check `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`
- Result: list of programming languages, primary programming language first

### 2.2 Framework(s)
- Check dependencies in package.json, requirements.txt, pyproject.toml, etc.
- Look for: React, Next.js, Vue, Svelte, Django, FastAPI, Flask, Express, NestJS, Spring, etc.
- Result: list of frameworks

### 2.3 Package Manager
- `package-lock.json` → npm
- `yarn.lock` → yarn
- `pnpm-lock.yaml` → pnpm
- `bun.lockb` → bun
- `requirements.txt` / `pyproject.toml` → pip / poetry
- `go.sum` → go modules
- Result: package manager name

### 2.4 Verification Commands
- From `package.json` scripts: look for `test`, `lint`, `typecheck`, `check`, `build`
- From `Makefile`: look for `test`, `lint`, `check` targets
- From `pyproject.toml`: tool configs (pytest, ruff, mypy)
- Result: exact commands to run for verify loop (e.g., `npm run lint && npm run test`)
- If empty repo: use placeholder `{{add verify commands}}`

### 2.5 Directory Structure
- Run a top-level directory listing
- Identify: source dirs (`src/`, `app/`, `lib/`), test dirs (`tests/`, `__tests__/`), config files
- Result: brief structure summary

### 2.6 File Naming Convention
- Analyze existing filenames in source directories
- Detect: `kebab-case`, `camelCase`, `PascalCase`, `snake_case`
- Result: detected convention for files and directories

### 2.7 Default Branch
- Run `git branch --show-current` or check `git remote show origin`
- Result: branch name (default: `main`)

### 2.8 Code Style Summary
- For the detected primary programming language, derive specific conventions:
  - **TypeScript**: type vs interface, enum policy, assertion policy, function style
  - **Python**: type hints policy, docstring style, import style
  - **Go**: error handling pattern, naming conventions
  - **Other**: derive equivalent conventions from the generic rules below

---

## Phase 3: Generate Files

### Section Marker Convention

Use HTML comment markers to separate auto-detected content from user-provided content:
```
<!-- auto:section-name -->
(auto-generated content here)
<!-- /auto:section-name -->
```

On **Update** mode: only overwrite content between `<!-- auto:* -->` markers. Preserve everything else.

---

### File 1: `CLAUDE.md` (project root)

```markdown
# CLAUDE.md — {project name from user or directory name}

## Project Overview

{user's answer to project overview question}

## User Communication

사용자는 제품/도메인 전문가이며 개발 비전문가. 빠른 결정, 철저한 검증. 간결함 선호.

### 소통 규칙
- 기술 용어 유지 + 설명 첨부. 쉬운 말로 대체 금지.
- 학술적/현학적 용어 지양. 쉬운 말로 같은 정확성 달성.
- 비유/은유 금지. 직접 설명.
- 리스크는 선택 시점에 제시. 결정 후 추가 리스크 금지.
- 선택지가 하나뿐이어도 사용자에게 확인. 자동 적용 금지.
- 순수 기술 결정: Builder 소관이지만, 장단점 + 추천 안내.
- 시스템이 아는 모든 정보 제공. 정보와 결정을 구분.
- 이름은 역할 기준. 실제로 일어나는 일을 반영.
- 의미의 정확성 > 편의성. 기존 용어 의미 확장 금지, 정확한 새 개념 추가.
- 복잡도 = 통제 가능성. 크기가 아님.
- 문서를 위한 문서 금지. 기존 문서에 간결 포함.
- 전역 규칙은 model-independent 위치(README.md 등)에 배치.
- 행위 주체는 설계 원칙에서 도출. 원칙에서 논리적으로 추론.

### 작업 방식
- 주요 결정마다 병렬 에이전트 리뷰 (6인).
- 2라운드 구조: Round 1 독립 리뷰 → 종합 → Round 2 재응답.
- 리뷰 기준: 목적·철학·persona 정합성 우선. 단순 오류 검출보다 상위.
- 전문가 만장일치도 근거 약하면 재검토. 합의 자체가 아니라 합의의 논리를 평가.
- 결정은 빠르게, 검증은 철저하게.
- 커밋은 의미 있는 작업 단위로.
- 상위설계: 레거시 무관. 세부구현: 레거시 참고 (실패 반복 방지).
- 좋은 이름이 떠오르지 않으면 기존 이름 유지.

### Anti-Patterns (X → O)
| 하지 말 것 | 해야 할 것 |
|-----------|-----------|
| 기술 용어를 쉬운 말로 대체 | 용어 유지 + 설명 첨부 |
| 학술적 용어로 정확성 추구 | 쉬운 말로 같은 정확성 달성 |
| 별도 문서로 원칙을 장황하게 서술 | 기존 문서에 간결 포함 |
| model-dependent 위치에 전역 규칙 배치 | model-independent 위치에 배치 |
| 리스크를 결정 후 별도 섹션에 제시 | 선택 시점에 각 선택지 리스크 제시 |
| 전문가 만장일치를 무비판적 수용 | 합의 근거를 별도 검증 |
| "파급범위가 크다"로 차선책 선택 | "통제 가능한 복잡도인가" 기준 |
| 편의를 위해 기존 용어 의미 확장 | 의미가 정확한 새 개념 추가 |

## Tech Stack

<!-- auto:tech-stack -->
- Programming Language: {detected}
- Framework: {detected}
- Package Manager: {detected}
<!-- /auto:tech-stack -->

## Verification Loop

<!-- auto:verify -->
After every change: {detected verify commands}.
Before PR: {detected full verify command}.
<!-- /auto:verify -->

## Code Style

Follow `@.claude/rules/coding-conventions.md` for all code.

## Project Patterns

Follow `@.claude/rules/project-patterns.md` for file naming, conventions, and terminology.

## Plan Mode — Design Protocol

Every non-trivial task starts in plan mode. Complete all 4 steps before switching to auto-accept.

**Step 1 — Scope Lock**
- In scope: concrete outcomes this change delivers
- Out of scope: anything else (tag Phase 2 if worth revisiting)
- Affected surface: which existing files/modules are touched
- Never expand scope mid-design

**Step 2 — Contracts First**
Define before writing any implementation:
- Input/output types: exact signatures for every public function/endpoint
- State transitions: all states, triggers, and illegal transitions
- Error cases: every failure mode with its type
- Invariants: conditions that must always hold

**Step 3 — Pre-mortem**
Answer explicitly before finalizing:
1. "If this fails in production, what breaks first?"
2. "What system state does this design not handle?"
3. "What assumption about existing code might be wrong?"
→ Any gap found → revise Step 2 contracts first

**Step 4 — Simplicity Gate**
- Remove any abstraction layer that isn't required for correctness
- Every new file/type/function must trace to a Step 1 requirement
- Plan must be understandable in <5 minutes by a new reader

**Transition**: If Claude can't 1-shot the implementation from this plan, the plan isn't done. Return to plan mode.

## Parallel Work

Subagents for clean context windows. One agent per file. For parallel streams: `git worktree add .claude/worktrees/<n> origin/main`

## Available Commands & Agents

### Individual Expert (`/ask-{agent}`)
단일 전문가 관점 상담. 각 에이전트가 자신의 전문 영역에서 답변.

| Command | Role |
|---------|------|
| `/ask-philosopher` | 시스템 목적·철학 정합성 |
| `/ask-ux_expert` | 사용자 경험 |
| `/ask-systems_architect` | 시스템 아키텍처 |
| `/ask-product_engineer` | 제품 엔지니어링 |
| `/ask-reliability_engineer` | 신뢰성 엔지니어링 |
| `/ask-integration_engineer` | 통합·연동 |

### Team Review (`/ask-review`)
6인 에이전트 패널 리뷰. 5인 독립 리뷰 → Philosopher 종합 → 5인 재응답 → 최종 합의.

### Multi-Agent Analysis (Prism)
| Command | Purpose |
|---------|---------|
| `/prism:plan` | 다각도 분석 → Devil's Advocate 검증 → 3인 위원회 토론 → 실행 계획 |
| `/prism:prd` | PRD를 정책 문서 대비 충돌·모호성 분석 |
| `/prism:incident` | 인시던트 포스트모템. 동적 관점 → DA 검증 → Tribunal |

## Prohibitions

- No skipped error handling
- No commits without tests
- No breaking API changes without discussion
- No scope expansion beyond Step 1 lock
- No implementation with unresolved pre-mortem gaps
- No abstractions without concrete in-scope justification
{append user's custom prohibitions here, if any}

## Self-Improvement

After every correction → update this file with a prevention rule.
```

---

### File 2: `.claude/rules/coding-conventions.md`

```markdown
# Coding Conventions

Self-contained, explicit, linearly readable code.

## File Structure
- One concern = one file. Co-locate types, constants, logic.
- Acceptable duplication over fragile shared abstraction.
- Export only what's consumed externally.
- Soft limit: 300 lines. Split by concern, not category.

## Naming
Names eliminate the need for comments:
- Booleans: `is/has/should/can`
- Transforms: `to/from/parse`
- Validation: `validate/assert`
- Events: `handle/on`
- Factories: `create/build/make`
- Fetching: `fetch/load/get`
- Constants: `UPPER_SNAKE_CASE`
- Avoid: single-letter vars, `data`/`info`/`temp`/`obj`, abbreviations

## Functions
- Pure by default: inputs → outputs, no side effects.
- Separate computation (pure) from coordination (I/O).
- Max 40 lines, max 3 params (options object beyond that).
- Early returns over nested if/else.
- No nested ternaries.

## Errors
- Fail fast: validate at boundary, not deep inside.
- Never empty catch. Include context: what failed, which inputs, why.

## Control Flow
- Linear top-to-bottom. No implicit event chain ordering.
- Explicit parallel vs sequential async.

## Config as Data
- Business rules as data structures, not procedural branches.
- New option = data change, not logic change.

## Module Boundaries
```
[External] → [Boundary: parse/validate] → [Pure Logic] → [Boundary: persist/send]
```
Business logic never imports I/O directly. Pass deps as params.

## Comments & Tests
- Comments: WHY only. Workarounds: link to issue. Delete commented-out code.
- Tests: co-locate. Behavior names. AAA pattern. Mock I/O only.

<!-- auto:programming-language-rules -->
{Generate programming-language-specific rules here based on the detected primary programming language.

For TypeScript:
- Explicit types on all function params and returns.
- States → discriminated unions, not boolean flags.
- Options → string literal unions, not enums.
- `unknown` + type guards, not `any`. No `as` assertions.
- Max 2 levels of generic nesting. Extract intermediate types.
- `type` over `interface`. Never `enum`.
- `function` keyword for top-level. Arrow for inline only.
- Result/Either for business logic. try/catch only at I/O boundaries.
- No shared barrel type files. No default exports.

For Python:
- Type hints on all function params and returns.
- Use dataclasses or Pydantic models for structured data.
- Use `Enum` for fixed option sets.
- Prefer `pathlib` over `os.path`.
- Use `raise` with specific exception types, not bare `raise`.
- Docstrings: Google style.
- Imports: standard lib → third-party → local, separated by blank lines.

For Go:
- Always check returned errors. No `_` for error values.
- Use named return values only when they improve readability.
- Interfaces: accept interfaces, return structs.
- Prefer table-driven tests.
- Use `context.Context` as first param for functions with I/O.

For other programming languages:
Derive equivalent conventions from the generic rules above, adapted to the programming language's idioms and best practices.}
<!-- /auto:programming-language-rules -->

## Do NOT
- Nested ternaries
- Empty catch blocks
- Mixed I/O and business logic
- Commented-out code
- Magic numbers
- Mutate function params
```

---

### File 3: `.claude/rules/project-patterns.md`

```markdown
# Project Patterns — {project name}

## File & Directory Naming

<!-- auto:naming -->
- Directories: {detected convention, e.g., `kebab-case/`}
- Files: {detected convention}
<!-- /auto:naming -->

## Language Protocol
- All files on `{default branch}`: English.
- Config keys, enum values, file paths: always English.
- User-facing string values: follow locale setting.
- Comments in code: English.

## Terminology Boundaries

{user's terminology answer, or "No project-specific terminology defined yet." if skipped}

| Term | Usage | NOT |
|------|-------|-----|
{rows from user input}

## Abbreviation Registry

{user's abbreviation answer, or "`N/A`" if skipped}
No other abbreviations without adding to this registry.

## Branch Rule

<!-- auto:branch -->
`{detected default branch}` = always deployable. All work on feature branches.
<!-- /auto:branch -->
```

---

## Phase 4: Report

After generating all files, output a summary:

```
## Onboard Complete

| File | Status |
|------|--------|
| CLAUDE.md | Created / Updated |
| .claude/rules/coding-conventions.md | Created / Updated |
| .claude/rules/project-patterns.md | Created / Updated |

### Auto-Detected
- Programming Language: {value}
- Framework: {value}
- Package Manager: {value}
- Verify: {value}
- Naming: {value}
- Branch: {value}

### From User Input
- Overview: {provided / skipped}
- Terminology: {provided / skipped}
- Abbreviations: {provided / skipped}
- Custom rules: {provided / skipped}
```

If in Update mode, also list which `<!-- auto:* -->` sections were changed.
