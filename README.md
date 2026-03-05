# claude-pipeline

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Claude Code](https://img.shields.io/badge/Claude_Code-Skills-blueviolet)

Claude Code 커스텀 스킬 기반 AI 개발 파이프라인.

`/pipeline`을 실행하면 리드 에이전트가 전문 에이전트들을 순서대로 디스패치해서 리서치부터 구현, 리뷰, 버그 수정까지 자동으로 처리합니다.

## 구조

```
/pipeline 결제 시스템 연동해줘

→ 리드 에이전트(Pipeline)가 상황 판단
  → Web Researcher (haiku)  — 웹 검색 + 정보 수집
  → Code Analyzer (sonnet)  — 코드베이스 분석 + 종합
  → Planner (opus)          — 태스크 분해 + 설계
  → Implementer (sonnet)    — 코드 구현
  → Reviewer (opus)         — 빌드/테스트 + 코드 리뷰
  → Bugfixer (sonnet)       — 이슈 수정
  → Retrospective           — 회고 + 스킬 자동 개선
```

각 에이전트는 격리된 서브프로세스로 실행되고, `.pipeline/` 디렉토리의 마크다운 파일로 소통합니다.

## 에이전트 팀
| Agent | Role | Model | Specialty |
|-------|------|-------|-----------|
| Web Researcher | 웹 검색 + 정보 수집 | haiku | 빠르고 저렴한 정보 수집 |
| Code Analyzer | 코드베이스 분석 + 종합 | sonnet | 패턴 파악, 구현 방향 제안 |
| Planner | 태스크 분해 + 설계 | opus | 아키텍처, 깊은 사고 |
| Implementer | 코드 구현 | sonnet | 코드 작성, 속도/비용 밸런스 |
| Reviewer | 코드 리뷰 + 빌드/테스트 | opus | 꼼꼼한 리뷰, 품질 중요 |
| Bugfixer | 이슈 수정 | sonnet | 지시된 수정, 속도 중요 |

## 주요 기능
### 리드 에이전트의 자율 판단

Decision Matrix는 기본 가이드라인일 뿐, 리드 에이전트가 상황에 따라 단계를 건너뛰거나 조정합니다.

```
우선순위: User Override > 자율 판단 > Decision Matrix (Default)
```

- 리팩토링, 내부 로직 변경 등은 리서치 생략 가능
- 새 기술 도입, 외부 API 연동 등은 리서치 필수
- 사용자가 "research 건너뛰어줘" 하면 항상 따름

### 태스크별 워크스페이스 분리

같은 프로젝트에서 여러 태스크를 돌려도 아티팩트가 충돌하지 않습니다.

```
.pipeline/
  my-app/
    login-feature/
      research.md
      plan.md
      progress.md
      review.md
      retrospective.md
    payment-integration/
      research.md
      plan.md
      ...
```

### 빌드 & 테스트 검증

Reviewer가 코드 diff만 보는 게 아니라 실제로 빌드와 테스트를 실행합니다.

### 구현 진행 상황 추적

Implementer가 태스크를 완료할 때마다 `progress.md`를 갱신합니다.

## 스킬 목록

| 스킬 | 설명 | 단독 사용 |
|------|------|-----------|
| `/pipeline` | 풀 파이프라인 (리드 에이전트가 판단해서 디스패치) | O |
| `/research` | 웹 리서치 + 코드베이스 분석 | O |
| `/plan` | 구현 플랜 작성 | O |
| `/implement` | 플랜 기반 구현 | O |
| `/review` | 코드 리뷰 + 빌드/테스트 | O |
| `/bugfix` | 리뷰 기반 버그 수정 | O |
| `/retrospective` | 파이프라인 회고 + 스킬 자동 개선 | O |

각 스킬은 `/pipeline` 없이도 독립적으로 사용할 수 있습니다.

## 설치

Claude Code에서 직접 설치:

```bash
/plugin marketplace add sunmerrr/claude-pipeline
/plugin install claude-pipeline@sunmerrr-claude-pipeline
```

### 권장 Permission 설정

파이프라인 실행 시 기본 명령어 승인을 줄이려면 `~/.claude/settings.local.json`에 추가:

```json
{
  "permissions": {
    "allow": [
      "Bash(ls *)", "Bash(pwd)", "Bash(basename *)",
      "Bash(mkdir *)", "Bash(cat *)", "Bash(head *)", "Bash(tail *)",

      "Bash(git status*)", "Bash(git diff*)", "Bash(git log*)",
      "Bash(git branch*)", "Bash(git show*)", "Bash(git rev-parse*)",

      "Bash(npm run *)", "Bash(npm test*)", "Bash(npx *)",
      "Bash(bun run *)", "Bash(bun test*)", "Bash(bunx *)",

      "Bash(tsc *)", "Bash(eslint *)", "Bash(prettier *)",
      "Bash(python -m pytest*)", "Bash(python -c *)",

      "WebSearch"
    ]
  }
}
```

## 사용 예시

```bash
# 풀 파이프라인
/pipeline OAuth2 소셜 로그인 추가해줘

# 리서치 건너뛰기
/pipeline 사이드바에 알림 아이콘 추가해줘
# → 리드 에이전트가 자율 판단으로 research 생략 가능

# 개별 스킬 사용
/research WebSocket 실시간 알림
/plan
/implement
/review
/bugfix
```

## 아티팩트 흐름

```
Web Researcher → web-research.md
                        ↓
Code Analyzer  → research.md (web-research.md 통합)
                        ↓
Planner        → plan.md (research.md 참조)
                        ↓
Implementer    → 코드 구현 + progress.md
                        ↓
Reviewer       → review.md (빌드/테스트 결과 포함)
                        ↓
Bugfixer       → 코드 수정 (review.md 참조)
                        ↓
Retrospective  → retrospective.md
```

---

## English Quick Start

**claude-pipeline** is a set of Claude Code custom skills that orchestrate an AI development team — from research and planning through implementation, review, and bugfix — all driven by a pipeline leader agent.

**Install in Claude Code:**

```bash
/plugin marketplace add sunmerrr/claude-pipeline
/plugin install claude-pipeline@sunmerrr-claude-pipeline
```

**Basic usage:**

```
/pipeline add user authentication with OAuth2
```

Individual skills can also be used standalone:

```
/research  →  /plan  →  /implement  →  /review  →  /bugfix  →  /retrospective
```

See [CONTRIBUTING.md](CONTRIBUTING.md) to contribute new skills or improvements.

---

## Contributing

Contributions are welcome! Whether you want to fix a bug, improve an existing skill, or add a new one, please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## License

MIT License - see [LICENSE](LICENSE) for details.
