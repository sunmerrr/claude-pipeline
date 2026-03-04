# Review Output Format

Use this markdown template for the review output:

---

## 작업 내용 요약

> 이 브랜치/워크트리에서 수행된 작업의 핵심 요약

- (what was changed and why — 2-5 bullets)
- Each bullet should answer: what changed + why it matters
- Group related changes into single bullets

## 변경 상세

### [Component/Area Name]

| 파일 | 변경 내용 | 비고 |
|------|-----------|------|
| `path/to/file.ts` | Description of change | Notes, warnings, etc. |

- Key implementation details or decisions worth noting
- Any potential issues or things to watch out for

> Repeat ### subsection for each component/area

### 주의사항

- Breaking changes, if any
- Migration steps needed, if any
- Configuration changes required, if any

## 빌드 & 테스트 결과

| 항목 | 결과 | 상세 |
|------|------|------|
| Build | ✅ Passed / ❌ Failed / ⚠️ No script | (에러 시 요약) |
| Test | ✅ Passed (N개) / ❌ Failed (N개) / ⚠️ No script | (실패 시 목록) |

> 실패한 경우 에러 메시지 핵심 부분을 포함한다.

## 테스트 방식 추천

### 자동 테스트

- [ ] Specific test command to run (e.g., `npm test -- --grep "feature"`)
- [ ] Specific test file to check or create
- Suggested test cases with brief descriptions

### 수동 테스트

- [ ] Step-by-step manual verification steps
- [ ] UI/UX checks if applicable
- [ ] API endpoint tests with example curl commands if applicable

### 엣지 케이스

- [ ] Edge cases that should be tested
- [ ] Error scenarios to verify
- [ ] Boundary conditions to check

---

## Notes on Usage

- Section headers can be in Korean or English — match the user's language
- The table format under "변경 상세" is preferred but not mandatory for simple changes
- If a section has no content (e.g., no edge cases), omit it rather than leaving it empty
- Be practical: test recommendations should be things the user can actually do right now
