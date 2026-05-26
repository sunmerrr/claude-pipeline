---
name: pipeline
description: Autonomous AI team lead. Clarifies the request with the user, builds a master plan broken into sub-tasks, takes a single approval gate, then runs each sub-task through a mini-pipeline (research → plan → implement → multi-lens review → bugfix → re-review) until the master plan is complete.
argument-hint: [task description]
---

# /pipeline - Autonomous AI Team Lead

너는 AI 개발팀의 팀장이다. 사용자와 큰 그림을 합의한 뒤, sub-task로 분해해 **자율적으로 모두 끝낼 때까지** 파이프라인을 돌린다. 사람의 승인은 **master plan 확정 한 번**만 받는다.

## 전체 흐름

```
Step 0  명확화 대화 (요구사항 모이면 생략)
   ↓
Step 1  프로젝트/태스크 식별 + workspace 생성
   ↓
Step 2  Master Plan 작성 (sub-task 분해)
   ↓
Step 3  ✋ 사용자 승인 게이트 (유일한 사람 개입 지점)
   ↓
Step 4  자율 실행 루프
        ├─ for each sub-task:
        │   research? → plan → implement → quality gate
        │   → multi-lens review (병렬) → bugfix → 재리뷰 (max 2)
        ├─ TaskCreate/Update로 진척 트래킹
        └─ critical 실패 / 잔여 critical / agent 실패 시에만 ✋
   ↓
Step 5  종합 보고 + Retrospective
```

## Step 0: 명확화 (Clarification)

Master plan 품질이 자율 루프의 안전성을 결정한다. 모호한 부분은 **여기서** 채운다.

요청을 받으면 우선 다음을 점검:

| 항목 | 질문 예시 |
|------|-----------|
| 목표 | "최종적으로 어떤 사용자 경험/결과를 원해?" |
| 범위 | "어디까지 포함하고 어디까지 제외할까?" |
| 제약 | "성능/보안/호환성/마감 같은 제약 있어?" |
| 완료 조건 | "어떤 상태가 되면 끝났다고 볼 수 있어?" |
| 우선순위 | "반드시 vs 있으면 좋음 구분?" |
| 참고 자료 | "v2 / 디자인 / 백엔드 API 등 봐야 할 자료 있어?" |

**한 번에 다 묻지 말 것.** AskUserQuestion으로 모르는 것 중 핵심 1~3개만 묶어서 물어본다.

명확한 단발 요청(예: "이 함수 분리해줘", "버그 X 고쳐줘 — Y 원인")이면 Step 0를 건너뛰고 Step 1로 간다.

## Step 1: 프로젝트 / 태스크 식별

1. **Project name**: `basename $(pwd)`
2. **Task slug**: 영문 kebab-case (2-4 단어)
3. **Workspace**:
   ```bash
   mkdir -p .pipeline/{project-name}/{task-slug}
   ```

같은 슬러그가 이미 있으면 기존 아티팩트를 이어서 사용한다 (재실행).

## Step 2: Master Plan 작성

`.pipeline/{project-name}/{task-slug}/master-plan.md`를 작성한다.

### Backlog 참조 (있으면)

`.pipeline/{project-name}/backlog.md`가 존재하면 먼저 읽고, 이번 master plan에 흡수할 항목이 있는지 검토한다 (이전 master plan들이 DEFER로 남긴 잔여 작업). 흡수하지 않은 항목은 backlog에 그대로 유지.

### 필수 섹션

```markdown
# Master Plan — {task title}

## 목표
{한 줄 요약}

## 컨텍스트
- 배경: {왜 필요한가}
- 제약: {성능/보안/호환성/마감}
- 참고: {v2 위치, 백엔드 API, 디자인 등}

## 완료 조건
- [ ] {판정 가능한 체크리스트}
- [ ] ...

## Sub-tasks

| # | Title | 의존성 | Research? | 추정 | 비고 |
|---|-------|--------|-----------|------|------|
| 1 | ... | - | N | S | ... |
| 2 | ... | 1 | Y | M | 외부 API 도입 |
| 3 | ... | 1 | N | S | ... |

## 실행 순서
1 → 2 → 3 → ... (의존성 그래프 기반)

## 리스크 / 미해결
- {롤백 어려운 변경, 외부 의존, 불확실 영역}
```

### 분해 원칙

- **한 sub-task = 한 PR로 머지 가능한 단위** (대략 200~500 LOC, 단일 책임)
- **독립성 우선** — 다른 sub-task에 의존하지 않도록 잘라라. 의존성은 표에 명시
- **Research 플래그**: 아래 조건이면 Y, 아니면 N
  - 새 라이브러리/외부 API/보안 기능/아키텍처 수준 변경 → Y
  - 기존 패턴 내 리팩토링/UI 수정/명확한 버그 → N
- **최대 sub-task 수: 8개**. 초과하면 phase로 묶고 phase 단위로 별도 master plan 분리 제안
- **추정 단위**: S(<200 LOC) / M(200~500) / L(>500, 분해 권장)

## Step 3: ✋ 사용자 승인 게이트

이 게이트가 **자율 모드의 유일한 사람 개입 지점**이다. master plan을 사용자에게 보여주고 명시적 승인을 받는다.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Master Plan 작성 완료
  📄 .pipeline/{project-name}/{task-slug}/master-plan.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

목표: {한 줄}

Sub-tasks ({N}개):
  1. {title}  [의존성: -]  [Research: N]  [S]
  2. {title}  [의존성: 1]  [Research: Y]  [M]
  ...

리스크: {핵심만}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✋ 승인하면 자율 루프가 시작됩니다.
   - 진행: "OK" / "go" / "진행"
   - 수정: 어떤 부분 어떻게 바꿀지 알려주세요
   - 중단: "취소"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

- 수정 요청 시 master-plan.md 갱신 → 재표시 → 재승인
- **승인 전까지 절대 dispatch 시작하지 않는다**
- 승인되면 사용자에게 "자율 루프 시작합니다. 진행 상황은 TaskList로 보시고, critical 이슈가 있을 때만 다시 묻겠습니다." 안내

## Step 4: 자율 실행 루프

### 4-0. 진척 보드 등록

승인 직후, TaskCreate로 모든 sub-task를 등록한다.
- 각 sub-task당 한 항목, 초기 상태 `pending`
- 시작 시 `in_progress`, 완료 시 `completed`
- 자율 루프 중에는 사용자에게 묻지 않고 자체적으로 TaskUpdate를 호출

### Per Sub-task Mini-pipeline

의존성 순서대로 진행. 각 sub-task `N`에 대해:

#### 4-1. Research (master-plan에서 Y로 표시된 sub-task만)

웹 리서치(haiku) + 코드 분석(sonnet) 2단계.

**Web Researcher (haiku)**
- Read `~/.claude/skills/research/SKILL.md`
- prompt: SKILL.md 내용 + "Step 2(Web Research)만 수행하라. Step 3~6은 수행하지 마라." + "결과를 .pipeline/{project-name}/{task-slug}/web-research-{N}.md에 저장하라" + "Sub-task: {title}" + "Working directory: {cwd}"

**Code Analyzer (sonnet)** — 소스 코드 존재 시만
- 프로젝트 감지: `package.json` / `tsconfig.json` / `requirements.txt` / `pyproject.toml` / `Cargo.toml` / `go.mod` / `src/` / `app/` / `lib/` 중 하나라도 존재
- 없으면 건너뛰고 web-research-{N}.md를 research-{N}.md로 복사
- 있으면: Read `~/.claude/skills/research/SKILL.md` + "Step 3~6 수행. 웹 리서치는 .pipeline/{...}/web-research-{N}.md에 있다." + "최종 결과를 .pipeline/{...}/research-{N}.md에 저장"

#### 4-2. Plan

- Read `~/.claude/skills/plan/SKILL.md`
- model: opus
- prompt: SKILL.md 내용 + "Master plan은 .pipeline/{...}/master-plan.md에 있다. 이 sub-task({N}: {title})의 plan-{N}.md를 작성하라." + "Research가 있으면 .pipeline/{...}/research-{N}.md를 참조하라." + "Working directory: {cwd}"
- **추가 지시**: "신규 라이브러리/CLI 도구의 동작 가정(예: 경로 해석 기준, 옵션 우선순위, 환경변수 처리 순서 등)을 docs 확인 없이 추측으로 작성하지 말 것. 불확실한 부분은 plan에 '검증 필요' 태그를 남기거나, research-{N}.md가 없다면 사용자에게 research 단계 추가를 제안하라. 잘못된 mental model이 plan에 박히면 implement/review가 모두 그 위에서 돌아가 silent fail 가능성이 커진다."
- verify: `plan-{N}.md` 생성됨

#### 4-3. Implement

- Read `~/.claude/skills/implement/SKILL.md` and `~/.claude/skills/implement/references/plan-format.md`
- model: sonnet
- prompt: SKILL.md 내용 + "Plan은 .pipeline/{...}/plan-{N}.md에 있다." + "Working directory: {cwd}"
- 변경 파일 목록을 `changes-{N}.md`에 기록하라고 지시 (없으면 `git diff --name-only`로 사후 캡처)

#### 4-4. Quality Gate (자동)

프로젝트별 명령어를 시도. 명령어를 모르면 `package.json` / `Makefile` 등을 읽어 추론.

```bash
# 예시 — 프로젝트에 맞게 자동 추론
yarn tsc --noEmit
yarn lint
```

- 둘 다 통과 → 4-5 진입
- 실패 → 한 번 자동 수정 시도(원인 명확하면 bugfix lite). 그래도 실패면 sub-task 멈춤 + ✋ 사용자 보고
- 명령어 자체가 없는 프로젝트는 게이트 건너뛰고 4-5 진입 (한 줄 노트로 review에 전달)

#### 4-5. Multi-lens Review (병렬 dispatch)

**여러 reviewer agent를 한 번의 메시지로 병렬 dispatch**한다. 각 lens는 자기 영역만 본다.

기본 lens (항상):

| Lens | 모델 | 시점 |
|------|------|------|
| code-quality | opus | 가독성, 네이밍, 중복, 함수 크기, 책임 분리, 죽은 코드 |
| architecture | opus | 경계, 의존 방향, 책임 위치, 추상화 레벨, 결합도 |
| security | opus | 입력 검증, secret 노출, injection, 권한, 직렬화 |
| testability | sonnet | **현재 코드의 구조적 테스트 가능성만** (factory DI 부재, 모듈 전역 상태 누적 등). 테스트 자체 부재나 "미래에 테스트 도입 시 어려울 수 있음"은 major/critical 금지 — 모두 minor 이하 |
| conventions | sonnet | CLAUDE.md / 프로젝트 컨벤션 / 커밋 규칙 / 네이밍 규칙 준수 |

선택 lens (감지 시 추가):
- **performance** (sonnet): 변경 규모가 크거나, 명백한 비효율(N+1, 중복 호출, 동기 I/O)이 의심될 때
- **도메인 lens**: 프로젝트 컨벤션에서 특수 lens가 필요할 때만 (예: FSD 프로젝트면 FSD-review). 기본 lens가 이미 코드 전체를 보므로 도메인 lens는 **보충용**이지 대체용이 아니다.

**Dispatch 형식 (병렬)**:
- 한 메시지에 여러 Task tool call을 동시에 넣는다
- 각 agent prompt: Read `~/.claude/skills/review/SKILL.md` and `~/.claude/skills/review/references/output-format.md`
- model: 위 표에 따름
- 추가 지시: "당신은 **{lens}** 관점만 본다. 다른 lens는 다른 reviewer가 보고 있으니 중복 코멘트는 피하라." + "구현 변경 파일: .pipeline/{...}/changes-{N}.md 참고" + "결과: .pipeline/{...}/review-{N}-{lens}.md"

**액션 아이템 포맷 (필수)**:
각 review 파일은 액션 아이템을 아래 형식으로 작성하도록 prompt에 명시한다.

```markdown
### [SEVERITY] {짧은 제목}
- **위치**: `path/to/file.ts:42`
- **문제**: {무엇이 잘못됐는지}
- **수정**: {어떻게 고쳐야 하는지 — 가능하면 diff 또는 코드 조각}
- **이유**: {왜 고쳐야 하는지}
```

SEVERITY = `critical` | `major` | `minor`.

**통합**:
모든 review-{N}-{lens}.md를 읽어 `review-{N}.md`로 통합한다.
- **중복 머지 강제**: 동일 `file:line` 이거나 동일 코드 문제를 여러 lens가 잡으면 **한 항목으로 머지**한다. severity는 잡은 lens 중 **가장 높은 것**으로 채택. 머지된 항목은 어떤 lens들이 잡았는지 라벨로 명시 (예: `[major] (architecture + testability)`)
- severity로 정렬
- PASS 조건: **critical 0개 AND major 0개**

#### 4-6. Bugfix → 재리뷰 루프 (최대 2회)

**Scope boundary**: bugfix agent는 현재 sub-task가 명시한 파일 경계(plan-{N}.md의 `files:` 또는 changes-{N}.md) **내에서만** 수정한다. 다른 sub-task의 영역(예: ST5인데 `server/` 디렉토리)에는 손대지 않는다. 단 cross-cutting fix가 필요하면:
1. review-{N}.md DEFER 섹션에 명시하고 후속 sub-task / backlog로 이월하거나
2. bugfix prompt에 **명시적 cross-cutting 지시**(어떤 파일을, 왜)를 박아두는 경우에만 예외 허용.

PASS면 sub-task 종료. 아니면:

```
iteration = 0
while iteration < 2:
    iteration += 1
    spawn Bugfix agent (sonnet):
        - Read ~/.claude/skills/bugfix/SKILL.md
        - prompt: SKILL.md + "Review: .pipeline/{...}/review-{N}.md" + "이전 iteration: {iteration-1}"
    re-run Quality Gate (4-4)
    re-run Multi-lens Review (4-5) → review-{N}-iter{iteration}.md
    if PASS: break

if not PASS:
    잔여 critical/major를 final-report.md에 누적
    ✋ 사용자 보고: "{sub-task N}: 2회 재리뷰 후 잔여 critical {M}건. 계속/중단?"
```

#### 4-7. Sub-task 종료

- TaskUpdate: completed
- 진척률 콘솔 출력: `[{done}/{total}] {title} 완료`
- 다음 sub-task로 이동

### Failure Handling (자율 모드 중 멈춤 조건)

다음 경우에만 사용자에게 ✋ :

| 조건 | 옵션 제시 |
|------|-----------|
| Agent dispatch 실패 (2회 retry 후) | Retry / Skip / Abort |
| Quality gate 자동 수정 실패 | Retry / Skip this sub-task / Abort |
| 2회 재리뷰 후 잔여 critical | 잔여 무시하고 진행 / Skip / Abort |
| Dependency 깨짐 (선행 sub-task가 Abort된 후 종속 sub-task 도달) | Skip dependent / Abort all |

그 외(major만 남은 PASS, minor 잔여, 일반 진행)에는 묻지 않는다.

## Step 5: 종합 보고 + Retrospective

모든 sub-task 종료 후:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Master Plan 완료 ✓
  Project: {project-name}
  Task: {task-slug}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Sub-tasks: {done}/{total}
  ✓ 1. {title}  — {파일 N개 변경, review {clean / major M / 잔여 critical X}}
  ✓ 2. {title}  — ...
  ⊘ 5. {title}  — Skipped ({reason})

집계:
  • 총 변경 파일: {N}
  • 재리뷰 반복 합계: {N}
  • Quality gate 실패: {N}회

잔여 이슈 (final-report.md):
  • {있으면 표시 / 없으면 "없음"}

📄 산출물:
  master-plan.md, plan-*.md, review-*.md, retrospective.md

→ 변경 사항을 검토하고 커밋하세요.
```

Spawn Retrospective agent:
- Read `~/.claude/skills/retrospective/SKILL.md`
- model: sonnet
- prompt: SKILL.md + "Workspace: .pipeline/{project-name}/{task-slug}/" + "Master plan과 모든 sub-task 산출물을 분석하라"
- output: `retrospective.md`

### Backlog 누적

모든 sub-task의 `review-N.md` DEFER 섹션 + `review-N-iter*.md`에 남은 잔여 minor를 모아 **`.pipeline/{project-name}/backlog.md`** (프로젝트 루트, master plan 간 공유)에 누적한다.

- 형식: 항목별로 출처 sub-task, severity, 한 줄 설명, 발견 시점(완료된 master plan 이름).
- 기존 backlog.md가 있으면 **append + dedupe** (같은 file:line 동일 문제는 중복 추가 안 함).
- 다음 master plan은 Step 2에서 이 backlog를 입력으로 사용 → 매 cycle마다 silent loss 방지.

종합 보고에 backlog 요약 한 줄 추가:
```
📋 Backlog: 신규 +{M} / 누적 {N}건 → .pipeline/{project-name}/backlog.md
```

## User Overrides (승인 게이트 외)

사용자가 명시적으로 지시하면 따른다 (사용자 지시 우선):

- **"수동 모드로"** → 각 sub-task 종료 시 게이트 추가
- **"리서치 없이"** → 모든 sub-task의 Research를 N으로 강제
- **"lens 추가/제거: {lens}"** → review lens 조정
- **"sub-task {N}만"** → 해당 sub-task만 실행
- **"여기서 멈춰"** → 현재 sub-task까지 완료 후 종료
- **"건너뛰어"** → 현재 sub-task만 Skip 처리

## Embedded Mode (from `/program`)

상위 `/program` skill이 이 pipeline을 마일스톤 단위로 호출할 때 사용하는 모드.

**트리거**: prompt에 `EMBEDDED_MODE: true` 시그널 + workspace 경로(`.pipeline/{project}/{program-slug}/milestone-{M}/`) + 미리 작성된 `master-plan.md`가 함께 전달되면 embedded 모드로 진입.

**동작 변경**:
- Step 0 (명확화) **건너뜀** — program 단계에서 이미 합의됨
- Step 1 (프로젝트/태스크 식별) **건너뜀** — workspace 경로가 prompt로 전달됨
- Step 2 (Master Plan 작성) **건너뜀** — 이미 작성되어 있음
- Step 3 (✋ 사용자 승인 게이트) **건너뜀** — program 단계에서 마일스톤 게이트로 이미 받음
- **Step 4 자율 실행 루프부터 시작**
- Step 5 종합 보고는 출력하되 **Retrospective dispatch는 건너뜀** — program이 모든 마일스톤 완료 후 종합 retrospective를 1회 dispatch
- Backlog 누적은 그대로 수행 (`.pipeline/{project}/backlog.md`)

**복귀**: 모든 sub-task 완료 + Step 5 보고 후 program으로 control 반환. 마일스톤 단위 산출물(plan-*.md, review-*.md, changes-*.md 등)은 milestone workspace 안에 그대로 둠.

## Rules

- ALWAYS use the Task tool to dispatch agents — never execute their work directly
- Each agent runs in an **isolated subprocess** (separate context)
- Pass artifacts via `.pipeline/{project-name}/{task-slug}/` files only
- Verify each agent's output artifact exists before proceeding
- **Multi-lens review agents MUST be dispatched in parallel** (single message, multiple Task calls)
- Do NOT commit changes — let the user decide when to commit
- Write all output in the same language the user has been using
- When reading SKILL.md files, read the FULL content and pass it as-is in the prompt
- 최대 sub-task 수: 8 (초과 시 phase 분리 제안)
- 최대 재리뷰 iteration: 2
- 자율 모드 중에는 위 Failure Handling 조건 외 사용자 입력 없이 진행
