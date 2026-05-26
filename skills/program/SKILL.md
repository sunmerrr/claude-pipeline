---
name: program
description: Multi-master-plan autonomous orchestrator. Takes a vision-level request (e.g., "도자기 쇼핑 사이트 만들고 싶어"), decomposes it into milestones, then runs each milestone as a master plan via the pipeline skill in embedded mode. After all milestones complete, proposes the next program cycle automatically.
argument-hint: [vision description]
---

# /program — Multi-Master-Plan Autonomous Orchestrator

비전 수준 요청을 받아 **마일스톤으로 분해**하고, 각 마일스톤을 `pipeline` skill(embedded 모드)로 자동 실행한다. 모든 마일스톤이 끝나면 종합 retrospective + **다음 program 후보 3개**를 자동 제안한다.

## 전체 흐름

```
사용자 비전 (예: "도자기 쇼핑 사이트")
   ↓
Step 0  비전 명확화 (목표/스택/우선순위)
   ↓
Step 1  프로젝트/program 식별 + workspace 생성
   ↓
Step 2  program-plan.md 작성 (마일스톤 분해, 최대 5개)
   ↓
Step 3  ✋ Program 승인 게이트 (사용자 1회 승인)
   ↓
Step 4  마일스톤 루프
        ├─ 4-1. master-plan-M.md 작성 (program 컨텍스트 + 마일스톤 정보)
        ├─ 4-2. ✋ 마일스톤 게이트 (go/조정/stop)
        ├─ 4-3. pipeline skill을 embedded 모드로 dispatch
        └─ 4-4. 마일스톤 완료 보고 → 다음으로
   ↓
Step 5  Program 완료 보고 + 종합 retrospective + backlog 정리
   ↓
Step 6  🎯 다음 program 후보 3개 자동 제안 → 사용자 선택
```

## Step 0: 비전 명확화

요청이 program 수준(여러 마일스톤이 필요한 비전)인지 먼저 확인. 단일 작업(예: "버그 X 고쳐줘", "로그인 추가")이면 사용자에게 `/pipeline` 사용을 안내하고 멈춘다.

program 수준이라면 다음을 합의:

| 항목 | 질문 예시 |
|------|-----------|
| **비전** | "이 program이 끝났을 때 사용자가 어떤 가치를 얻길 원해?" |
| **기술 스택** | "프론트/백/DB 스택 정해진 게 있어?" |
| **마일스톤 우선순위** | "MVP는 어디까지? 메인/로그인/상품/결제/관리자 같은 마일스톤 중 1차 cycle은 어디서 끊을까?" |
| **제약** | "마감/예산/사용 라이브러리 제약 있어?" |
| **참고 자료** | "디자인 시안 / 기획서 / 참고 사이트?" |
| **완료 정의** | "1차 cycle 끝났다고 판단할 기준?" |

AskUserQuestion으로 핵심 2~3개만 묶어서 묻는다. 한 번에 다 묻지 말 것.

## Step 1: 프로젝트 / Program 식별

1. **Project name**: `basename $(pwd)`
2. **Program slug**: 영문 kebab-case (예: `pottery-shop-mvp`, `admin-dashboard`)
3. **Workspace**:
   ```bash
   mkdir -p .pipeline/{project-name}/{program-slug}
   ```

**Backlog 참조** (있으면): `.pipeline/{project-name}/backlog.md`를 먼저 읽고, 이번 program에 흡수할 항목이 있는지 검토. (이전 program이 남긴 잔여)

## Step 2: program-plan.md 작성

`.pipeline/{project-name}/{program-slug}/program-plan.md` 작성.

### 필수 섹션

```markdown
# Program Plan — {비전 title}

## 비전
{한 문단. 최종 사용자 경험과 가치.}

## 컨텍스트
- 배경: ...
- 기술 스택: ...
- 제약: ...
- 참고: ...

## 1차 cycle 완료 정의
- [ ] {판정 가능한 기준}
- [ ] ...

## 마일스톤

| # | Title | 의존성 | 추정 sub-task 수 | 비고 |
|---|-------|--------|------------------|------|
| 1 | ... | - | 4~6 | ... |
| 2 | ... | 1 | 3~5 | ... |
| 3 | ... | 1, 2 | 3~5 | ... |

## 실행 순서
1 → 2 → 3 → ...

## 리스크 / 미해결
- {기술 검증 필요 영역, 외부 의존 등}
```

### 분해 원칙

- **한 마일스톤 = 한 master plan** (사용자가 클릭/확인 가능한 단위)
- **최대 마일스톤 수: 5개**. 초과하면 program을 더 작게 쪼개도록 제안 (예: "메인 + 로그인"만 cycle 1, "상품 + 결제"는 cycle 2)
- 각 마일스톤이 끝나면 **그 자체로 사용자 가치**가 있어야 함 (예: "메인 페이지 보임" → 디자인 검증 가능)
- 의존성 명시. 병렬 가능 마일스톤도 일단 순차로 실행 (자율 안전성)

## Step 3: ✋ Program 승인 게이트

이 게이트가 program 모드의 **첫 번째이자 가장 무거운 사람 개입 지점**.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Program Plan 작성 완료
  📄 .pipeline/{project-name}/{program-slug}/program-plan.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

비전: {한 줄}

마일스톤 ({N}개):
  1. {title}  [의존성: -]  [≈{K} sub-task]
  2. {title}  [의존성: 1]  [≈{K} sub-task]
  ...

총 예상 규모: 약 {Σ sub-task}개 sub-task, {N}개 마일스톤 게이트
리스크: {핵심만}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✋ 승인하면 마일스톤 루프가 시작됩니다.
   - 진행: "OK" / "go" / "진행"
   - 마일스톤 조정: 어떤 마일스톤을 어떻게 바꿀지 알려주세요
   - 중단: "취소"

  자율 모드 안내: 이후 각 마일스톤 시작 시 짧은 게이트가 한 번씩
  뜹니다. 마일스톤 내부의 sub-task 루프는 자동 진행됩니다.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

- 수정 요청 시 program-plan.md 갱신 → 재표시 → 재승인
- **승인 전까지 절대 마일스톤 루프 시작하지 않는다**

## Step 4: 마일스톤 루프

### 4-0. 진척 보드 등록

TaskCreate로 모든 마일스톤을 진척 보드에 등록 (sub-task가 아닌 **마일스톤** 단위). 자율 루프 중 TaskUpdate.

### Per Milestone

#### 4-1. master-plan-M.md 작성

마일스톤 `M`을 위한 master plan을 작성한다. 위치: `.pipeline/{project-name}/{program-slug}/milestone-{M}/master-plan.md`

```bash
mkdir -p .pipeline/{project-name}/{program-slug}/milestone-{M}
```

master-plan.md 내용은 pipeline skill의 Step 2 master plan 포맷을 따른다 (목표/컨텍스트/완료 조건/Sub-tasks 표/실행 순서/리스크). 단:
- **컨텍스트 섹션**에 program-plan.md와 이전 마일스톤 산출물 reference 명시
- **Sub-tasks** 분해는 PR-sized (200~500 LOC, max 8개)
- 이전 마일스톤이 만든 코드/구조를 활용하도록 명시

**작성 시 주의** (pipeline의 Plan agent 지시 그대로 적용):
> 신규 라이브러리/CLI 도구의 동작 가정을 docs 확인 없이 추측으로 작성하지 말 것. 불확실하면 master plan의 한 sub-task에 "research Y"로 표시.

#### 4-2. ✋ 마일스톤 게이트

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  마일스톤 {M}/{N}: {제목}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

목표: {한 줄}

Sub-tasks ({K}개):
  1. {title}  [Research: N]  [S]
  2. {title}  [Research: Y]  [M]
  ...

영향 영역: {파일/디렉토리 추정}
의존성: {앞 마일스톤 산출물 어떤 부분?}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✋ - 진행: "OK"
   - 조정: 어떤 sub-task 어떻게 바꿀지
   - 중단: "stop" (program 전체 중단)
   - 건너뛰기: "skip this milestone"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

- 조정 시 master-plan-M.md 갱신 → 재게이트
- stop 시 program 전체 중단 (이미 끝난 마일스톤은 그대로 보존)
- skip 시 마일스톤을 건너뛰고 다음으로. 종속 마일스톤이 있다면 dependency 깨짐 처리

#### 4-3. pipeline skill을 embedded 모드로 dispatch

`pipeline` skill의 Step 4(자율 실행 루프)부터 시작하도록 dispatch.

**Dispatch 방식**: Task tool로 general-purpose agent를 띄우되 prompt에 다음을 포함:
- Read `~/.claude/skills/pipeline/SKILL.md` FULL content
- `EMBEDDED_MODE: true` 시그널
- workspace 경로: `.pipeline/{project-name}/{program-slug}/milestone-{M}/`
- master-plan.md는 이미 작성됨 → Step 0~3 건너뛰고 Step 4부터 시작
- Retrospective dispatch는 건너뛰고 program이 종합 retrospective 1회만 dispatch한다고 명시

verify: 마일스톤 종료 시 `milestone-{M}/` 안에 `plan-*.md`, `review-*.md`, `changes-*.md` 산출물이 생성됨.

#### 4-4. 마일스톤 완료 보고

```
━━━ 마일스톤 {M}/{N} 완료 ━━━
- Sub-task: {done}/{total}
- 변경 파일: {N}
- 잔여 critical: {N}
- DEFER → backlog 누적: +{M}건
- 다음 마일스톤: {M+1} {제목}

(자동으로 다음 마일스톤 게이트로)
```

TaskUpdate: 해당 마일스톤 → completed.

### Failure Handling (program 모드)

| 조건 | 동작 |
|------|------|
| pipeline embedded dispatch 실패 | program 중단 → ✋ Retry / Skip this milestone / Abort program |
| pipeline이 critical 잔여로 멈춤 | program 중단 → ✋ 사용자 결정 (이미 끝난 마일스톤은 보존) |
| 사용자가 마일스톤 게이트에서 stop | program 전체 중단 (graceful) |
| Dependency 깨짐 (앞 마일스톤 skip 또는 abort 후) | ✋ Skip dependent / Abort all |

## Step 5: Program 완료 보고

모든 마일스톤 종료 후:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Program 완료 ✓
  비전: {program title}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

마일스톤: {done}/{total}
  ✓ 1. {title}  — {파일 N개, 잔여 X건}
  ✓ 2. {title}  — ...
  ⊘ {M}. {title}  — Skipped ({reason})

집계:
  • 총 sub-task: {N}
  • 총 변경 파일: {N}
  • 재리뷰 반복 합계: {N}
  • Quality gate 실패: {N}회
  • Backlog 신규 +{M} / 누적 {total}건

📄 산출물:
  program-plan.md
  milestone-1/, milestone-2/, ...
  backlog.md (프로젝트 루트)
  retrospective.md (아래 단계에서 생성)
```

**종합 retrospective dispatch** (sonnet):
- Read `~/.claude/skills/retrospective/SKILL.md`
- prompt: SKILL.md + "Workspace: .pipeline/{project-name}/{program-slug}/" + "이건 multi-milestone program의 종합 retrospective다. 각 milestone-{M}/ 하위의 산출물과 program-plan.md를 모두 분석. 마일스톤 간 패턴(예: 반복되는 lens finding, agent 품질 변화)에 주목."
- output: `.pipeline/{project-name}/{program-slug}/retrospective.md`

**Backlog 정리**: 모든 마일스톤의 DEFER 항목을 `.pipeline/{project-name}/backlog.md`에 append + dedupe (기존 backlog 형식 유지).

## Step 6: 🎯 다음 program 후보 3개 자동 제안

retrospective.md + backlog.md + 이번 program 비전을 입력으로 다음 program 후보를 자동 생성한다.

**Dispatch** (opus, Plan agent 재사용):
- Read `~/.claude/skills/plan/SKILL.md`
- prompt: SKILL.md + "이번 program(`{program-slug}`)이 끝났다. 다음 program 후보 **3개**를 제안하라. 각 후보는 다음 형식으로 작성:
  ```markdown
  ### 후보 {A/B/C}: {제목}
  - **사유**: 한 줄 (예: backlog에 누적된 X건 정리, 또는 사용자 가치 다음 단계)
  - **예상 마일스톤**: 3~5개 (간략 나열)
  - **예상 규모**: S/M/L
  - **근거**: retrospective.md / backlog.md 어디서 도출됐는지
  ```
  후보 다양성: (1) backlog 정리 중심, (2) 비전 확장 중심(다음 사용자 가치), (3) 비기능적 개선(성능/보안/SEO/테스트) 중 다른 축에서 1개씩. 입력: program-plan.md, retrospective.md, backlog.md, 모든 milestone-*/review-*.md."
- output: `.pipeline/{project-name}/{program-slug}/next-program-candidates.md`

**사용자 제시**:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 다음 program 후보
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

A. {제목}
   사유: {한 줄}
   마일스톤: ~{N}개, 규모 {S/M/L}

B. {제목}
   사유: {한 줄}
   마일스톤: ~{N}개, 규모 {S/M/L}

C. {제목}
   사유: {한 줄}
   마일스톤: ~{N}개, 규모 {S/M/L}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✋ - 선택: "A" / "B" / "C" — 새 program cycle 시작
   - 새로 정의: 직접 비전 입력
   - 종료: "끝"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

사용자가 후보 선택 시 → 새 program cycle을 Step 1부터 시작 (선택된 후보의 제목 + 사유를 비전으로 사용). 직접 정의 시 → 사용자 입력을 비전으로 Step 0부터. 종료 시 → 작업 마침.

## User Overrides (게이트 외)

사용자가 명시적으로 지시하면 따른다:

- **"수동 모드"** → 각 마일스톤의 sub-task 사이에도 게이트 추가 (pipeline의 수동 모드 override 전달)
- **"마일스톤 {M}만"** → 해당 마일스톤만 실행
- **"여기서 멈춰"** → 현재 마일스톤까지 완료 후 중단
- **"backlog만 정리"** → master plan 생성 없이 backlog.md 정리 후 종료

## Rules

- ALWAYS use the Task tool to dispatch agents — never execute their work directly
- Each agent runs in an **isolated subprocess**
- 모든 산출물은 `.pipeline/{project-name}/{program-slug}/`(program workspace) 또는 그 안 `milestone-{M}/`(마일스톤 workspace)에만 작성
- backlog.md는 프로젝트 루트(`.pipeline/{project-name}/backlog.md`) — program 간 공유
- pipeline을 호출할 때는 **반드시 EMBEDDED_MODE 시그널** 포함 — 안 그러면 pipeline이 자기 승인 게이트를 다시 띄움
- Do NOT commit changes — 사용자 결정
- Write all output in the same language the user has been using
- When reading SKILL.md files, read the FULL content and pass it as-is
- 최대 마일스톤 수: 5 (초과 시 program 분리 제안)
- 사용자 개입 지점: program 승인 1회 + 마일스톤 게이트 N회 + 다음 program 선택 1회. 그 외는 자율
- 자율 모드 중에는 Failure Handling 조건 외 사용자 입력 없이 진행
