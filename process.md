# Agent Process — 공통 정의

각 프로세스 파일(`processes/`)이 참조하는 공통 정의.

## 프로세스 맵

| 프로세스 | 파일 | 설명 | 관련 프로세스 |
|---|---|---|---|
| 온보딩 | `processes/onboard.md` | 프로젝트에 onto-review 환경 설정 | → 리뷰, 질문 |
| 개별 질문 | `processes/question.md` | 1인 에이전트에게 질문 | 학습 → 승격 |
| 팀 리뷰 | `processes/review.md` | 8인 패널 리뷰 (Agent Teams) | 학습 → 승격 |
| 코드 기반 구축 | `processes/build.md` | 코드에서 온톨로지 추출 (Agent Teams) | → 변환, 리뷰 |
| 변환 | `processes/transform.md` | Raw Ontology 형식 변환 | 구축 → |
| 학습 승격 | `processes/promote.md` | 프로젝트 학습을 글로벌로 승격 (Agent Teams) | 리뷰/질문 → |

---

## 에이전트 구성

### 검증 에이전트 (7인)
| ID | 역할 | 검증 차원 |
|---|---|---|
| `onto_logic` | 논리적 일관성 검증자 | 모순, 타입 충돌, 제약 상충 |
| `onto_structure` | 구조적 완전성 검증자 | 고립된 요소, 끊어진 경로, 누락 관계 |
| `onto_dependency` | 의존성 무결성 검증자 | 순환, 역방향, 다이아몬드 의존 |
| `onto_semantics` | 의미적 정확성 검증자 | 이름-의미 일치, 동의어/동형이의어 |
| `onto_pragmatics` | 활용 적합성 검증자 | 질의 가능성, 역량 질문 테스트 |
| `onto_evolution` | 확장·진화 적합성 검증자 | 새 데이터/도메인 추가 시 깨짐 |
| `onto_coverage` | 도메인 포괄성 검증자 | 누락 하위 영역, 개념 편중, 표준 대비 빈 영역 |

### 목적 정합성 검증자 (1인)
| ID | 역할 |
|---|---|
| `philosopher` | 시스템 목적 기반 메타 관점 제공, 7인의 판단을 목적 관점에서 종합·재정의, 새로운 관점 제시 |

### 도메인 문서
각 에이전트는 실행 시 해당 도메인의 문서를 읽습니다:
```
~/.claude/agent-memory/domains/{domain}/
  ├── logic_rules.md        ← onto_logic
  ├── structure_spec.md     ← onto_structure
  ├── dependency_rules.md   ← onto_dependency
  ├── concepts.md           ← onto_semantics
  ├── competency_qs.md      ← onto_pragmatics
  ├── extension_cases.md    ← onto_evolution
  └── domain_scope.md       ← onto_coverage
```

### 도메인 판별 규칙

1. **프로젝트 선언 우선**: 프로젝트의 CLAUDE.md에 `domain:` 또는 `agent-domain:` 선언이 있으면 해당 도메인을 사용합니다.
2. **다중 도메인**: `secondary_domains:`가 선언되어 있으면, 주 도메인 문서를 우선 참조하고, 보조 도메인 문서를 추가로 참조합니다. 규칙이 충돌하면 주 도메인이 우선합니다.
3. **선언 없음**: 프로젝트에 도메인 선언이 없으면 사용자에게 질문합니다.
4. **도메인 문서 없음**: 선언된 도메인의 문서(`~/.claude/agent-memory/domains/{domain}/`)가 존재하지 않으면, 도메인 규칙 없이 범용 원칙만으로 검증합니다.

---

## 학습 저장 구조

에이전트의 학습은 **3개 경로**로 분리 저장됩니다:

| 학습 유형 | 저장 경로 | 내용 |
|---|---|---|
| **소통 학습 (공통)** | `~/.claude/agent-memory/communication/common.md` | 모든 에이전트에 적용되는 소통 규칙 |
| **소통 학습 (개별)** | `~/.claude/agent-memory/communication/{agent-id}.md` | 특정 에이전트의 소통 선호 |
| **방법론 학습** | `~/.claude/agent-memory/methodology/{agent-id}.md` | 도메인 무관한 검증 원칙, 일반적 교훈 |
| **도메인 학습** | `~/.claude/agent-memory/domains/{domain}/learnings/{agent-id}.md` | 특정 도메인에서만 유효한 학습 |

---

## Agent Teams 실행 방식

병렬 에이전트 실행이 필요한 프로세스(팀 리뷰, 코드 기반 구축, 학습 승격)는 **Agent Teams**를 우선 사용합니다.

### Fallback 규칙

TeamCreate 실패 시, Agent tool(subagent) 방식으로 fallback합니다. fallback 시에도 프로세스 흐름은 동일하되:
- TeamCreate/SendMessage 대신 Agent tool을 사용합니다.
- 각 Agent tool call에 아래 초기 prompt + 작업 지시를 합쳐서 전달합니다.
- 직접 토론(Round 3 등)은 생략합니다.

### Team lead 행동 규칙

- 수집한 결과를 전달할 때 내용을 수정하거나 요약하지 않는다.
- 자신의 판단을 리뷰에 개입시키지 않는다.
- teammate 간 결과를 교차 공유하지 않는다 (독립성 보장).
- 팀 생명주기를 관리한다 (생성 → 작업 할당 → 종료).

### Teammate 초기 prompt 템플릿

각 teammate 생성 시 (Agent tool의 prompt), 아래 형식으로 에이전트 정체성과 컨텍스트를 설정합니다:

```
당신은 {역할}입니다.
{팀명} 팀에 참여합니다.

[당신의 정의]
{~/.claude/plugins/onto-review/agents/{agent-id}.md 내용}

[과거 학습 — 방법론]
{~/.claude/agent-memory/methodology/{agent-id}.md 내용. 없으면 "아직 없음"}

[과거 학습 — 도메인]
{도메인 학습 (글로벌 + 프로젝트) 내용. 없으면 "아직 없음"}

[도메인 규칙]
{해당 에이전트의 도메인 문서 내용. 없으면 "도메인 문서 없음"}

[소통 학습]
{공통 + 개별 소통 학습 내용. 없으면 "아직 없음"}

[팀 규칙]
- team lead가 SendMessage로 작업을 할당합니다. 작업을 받으면 수행하고 결과를 team lead에게 SendMessage로 보고하세요.
- team lead가 허용하기 전에는 다른 teammate에게 직접 메시지를 보내지 마세요.
- 한국어(존댓말)로 답변하세요.
- 비유/은유를 사용하지 마세요.
```

---

## 학습 저장 규칙

### 3분류 저장

**소통 학습** (`~/.claude/agent-memory/communication/{agent-id}.md`):
- "소통 학습" 항목 중 "없음"이 아닌 것을 저장합니다.
- 사용자의 소통 선호, 작업 방식, 피드백에 대한 발견입니다.

항목 형식:
```markdown
### {날짜} — {프로젝트명} / {질문/리뷰 대상 요약}

- **맥락**: {어떤 질문/리뷰에서, 어떤 상황에서 발견되었는지}
- **발견 내용**: {사용자 소통에 대해 새로 알게 된 것}
- **반영 여부**: 미반영 (사용자 확인 필요)
```

**방법론 학습** (`~/.claude/agent-memory/methodology/{agent-id}.md`):
- "방법론 학습" 항목을 파일 끝에 추가합니다.
- 도메인에 무관하게 적용 가능한 검증 원칙/교훈입니다.
- 기존 항목과 중복되면 추가하지 않습니다.
- 기존 항목과 모순되면 새 학습으로 교체합니다.

항목 형식:
```markdown
- {학습 내용} (출처: {프로젝트명}, {질문/리뷰 대상 요약}, {날짜})
```

**도메인 학습**:
- 저장 경로 판별:
  - 프로젝트에 `.claude/learnings/` 디렉토리가 존재하면: `{project}/.claude/learnings/{agent-id}.md` (프로젝트 수준)
  - 존재하지 않으면: `~/.claude/agent-memory/domains/{domain}/learnings/{agent-id}.md` (글로벌 수준)
- "도메인 학습" 항목을 파일 끝에 추가합니다.
- 해당 도메인/프로젝트에서만 유효한 학습입니다.
- 동일한 중복/모순 규칙을 적용합니다.

항목 형식:
```markdown
- {학습 내용} (출처: {질문/리뷰 대상 요약}, {날짜})
```

### 저장 후 안내

새로 추가된 communication 항목이 있으면 사용자에게 알립니다:
"communication 발견사항이 N건 기록되었습니다. `~/.claude/agent-memory/communication/`에서 확인하시고, 전역 설정(`~/.claude/CLAUDE.md`)에 반영할지 결정해 주세요."

---

## Rules

- 모든 에이전트는 한국어(존댓말)로 답변합니다.
- 에이전트는 비유/은유를 사용하지 않습니다. 직접 설명합니다.
- 기술 용어는 유지하고 설명을 붙입니다.
- 팀 리뷰 모드에서 만장일치에 도달해도, 합의의 **논리적 근거**를 별도로 검증합니다.
- 질문/리뷰 대상이 불명확하면 사용자에게 질문합니다.
- 학습 파일이 200행을 초과하면, 사용자에게 알립니다. 정리 여부와 범위는 사용자가 결정합니다.
- 도메인 문서가 존재하지 않는 프로젝트에서는, 에이전트가 도메인 규칙 없이 범용 원칙만으로 검증합니다.
