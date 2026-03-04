# Skill Writing Guide

이 문서는 Claude가 스킬 파일(SKILL.md)을 생성할 때 참조하는 기술 가이드이다.
Anthropic 공식 Agent Skills spec에서 핵심을 추출하고 프로젝트 규칙을 반영했다.

---

## 1. 디렉토리 구조

```
skills/
├── automation/         ← 자동화 체험 스킬
├── connector/          ← 도구 연결 스킬
├── skill-creator/      ← 이 스킬
└── {스킬명}/           ← 사용자가 만든 스킬
    ├── SKILL.md        (필수)
    └── references/     (선택)
```

사용자가 만드는 스킬은 대부분 `SKILL.md` 단일 파일로 충분하다.
고급 사용자(개발자)가 요청하면 references/, scripts/, assets/ 디렉토리를 추가할 수 있다.

---

## 2. SKILL.md Frontmatter 형식

```yaml
---
name: {스킬명}
description: >
  {무엇을 하는지 + 언제 사용하는지 — 자연어로}
triggers:
  - "{스킬명}"
  - "{스킬명의 다른 표현}"
  - "{영어 표현 (선택)}"
user-invocable: true
---
```

### 필수 필드
- **name**: 스킬 이름. **영문 lowercase + hyphens 권장** (예: `weekly-report`, `meeting-notes`). 한글도 허용한다.
  - 디렉토리명 = name 필드 값
  - 예: `name: weekly-report` → `skills/weekly-report/SKILL.md`
- **description**: 스킬이 하는 일과 **언제 사용하는지** 둘 다 포함한다.
  - "slightly pushy" 원칙: description이 충분히 구체적이면 Claude가 적절한 상황에서 자동으로 이 스킬을 제안할 수 있다.

### 선택 필드
- **triggers**: 스킬을 호출하는 키워드 목록. 최소 2개 이상 등록 권장
- **disable-model-invocation**: `true`로 설정하면 수동 실행만 허용 (자동 트리거 방지)
- **user-invocable**: `true`로 설정하면 `/스킬명`으로 직접 실행 가능

### 이름 규칙
- **영문 lowercase + hyphens** 권장: `weekly-report`, `meeting-notes`, `ticket-create`
- 한글도 허용: `주간보고`, `회의록`, `티켓생성`
- 짧고 직관적으로: 2~4단어
- triggers에 한글 키워드를 등록하면 한글로도 호출 가능

### triggers 규칙
- 한글 자연어: `"주간보고"`, `"주간보고 해줘"`
- 영어: `"weekly report"`
- 최소 2개 이상

### description 작성 가이드

**좋은 예시** (무엇 + 언제):
```yaml
description: >
  Jira에서 이번 주 내 티켓을 조회하여 주간 업무 리포트를 위키에 자동 작성한다.
  매주 주간보고를 작성할 때, 또는 "주간보고" 요청 시 사용한다.
```

**나쁜 예시** (무엇만, 비구체적):
```yaml
description: 주간보고를 작성해주는 스킬입니다.
```

---

## 3. SKILL.md 본문 작성 스타일

### Imperative/Infinitive Form (동사 시작 지시문)

스킬 본문은 **imperative form**으로 작성한다. Claude에게 주어지는 지시문이기 때문.

**올바른 예시 (imperative):**
```
Jira에서 내 정보를 확인한다.
이번 주 업데이트된 티켓을 전체 조회한다.
상태별로 분류하여 보고서를 작성한다.
```

**잘못된 예시 (second person):**
```
당신은 Jira에서 내 정보를 확인해야 합니다.
사용자가 원하면 티켓을 조회해주세요.
```

### Third-Person Description (frontmatter)

frontmatter의 description은 third-person 형식으로 작성한다:

**올바른 예시:**
```yaml
description: >
  Jira에서 이번 주 내 티켓을 조회하여 주간 업무 리포트를 위키에 자동 작성한다.
```

**잘못된 예시:**
```yaml
description: 주간보고를 작성해주는 스킬입니다.  # 사용자 관점, 비구체적
```

### "Explain the Why" 원칙

스킬 본문에서 특정 동작을 지시할 때, **왜 그렇게 하는지** 간단히 설명한다.
Claude가 의도를 이해하면 예외 상황에서도 올바르게 판단할 수 있다.

**좋은 예시:**
```
상태별로 분류한다 (완료/진행중/대기 — 관리자가 진행 상황을 한눈에 파악하기 위해).
```

**나쁜 예시:**
```
상태별로 분류한다.
```

### 자연어 사용 (MCP 함수명 금지)

스킬 본문에 `mcp__jira__myself()` 같은 기술적 함수명을 쓰지 않는다.
Claude는 자연어 설명만으로 올바른 도구를 찾아 실행할 수 있다.

**나쁜 예시:**
```markdown
1. `mcp__jira__myself()`로 내 정보 확인
2. JQL `assignee = currentUser() AND updated >= startOfWeek()`로 티켓 조회
```

**좋은 예시:**
```markdown
1. Jira에서 내 정보 확인
2. 이번 주 업데이트된 내 티켓 전체 조회
3. 상태별(완료/진행중/대기)로 분류하여 보고서 작성
```

---

## 4. Progressive Disclosure 원칙

스킬은 세 단계 로딩 시스템으로 컨텍스트를 효율적으로 관리한다:

1. **Metadata (name + description)** — 항상 컨텍스트에 로드 (~100 tokens)
2. **SKILL.md 본문** — 스킬이 트리거되면 로드 (**500줄 이내**)
3. **Bundled resources** — 필요할 때만 로드 (제한 없음)

### 사용자 스킬에서의 적용

대부분의 사용자 스킬은 간단하므로 SKILL.md 하나로 충분하다.
복잡한 스킬의 경우:
- 핵심 워크플로우 → SKILL.md 본문
- 상세 참조 자료 → references/ 디렉토리
- 실행 스크립트 → scripts/ 디렉토리

**SKILL.md 본문은 500줄 이내를 목표**로 한다.
500줄을 초과하면 references/로 분리를 검토한다.

---

## 5. 스킬 본문 구조 (권장)

```markdown
# {스킬 제목}

{스킬이 하는 일 1~2문장 설명}

## 실행 절차

1. {Step 1 — 자연어로 무엇을 하는지}
2. {Step 2}
3. {Step 3}
...

## 출력 형태

{결과물에 대한 설명 — 위키 페이지, PPT, Excel, 화면 표시 등}

## 참고사항 (선택)

{추가 규칙이나 조건 — 필요한 경우만}
```

### 주의사항
- 하드코딩된 값(스페이스명, 프로젝트명 등)은 사용자가 확인한 값만 사용
- 민감한 정보(토큰, 비밀번호)를 스킬에 포함하지 않는다
- 기존 스킬을 삭제하지 않는다 (수정만 가능)

---

## 6. 출력 형태별 예시

### 위키 출력 스킬

```yaml
---
name: weekly-report
description: >
  Jira에서 이번 주 내 티켓을 조회하여 주간 업무 리포트를 위키에 자동 작성한다.
  매주 주간보고를 작성할 때 사용한다.
triggers:
  - "주간보고"
  - "주간 보고"
  - "weekly report"
user-invocable: true
---

# 주간 업무 리포트

Jira에서 이번 주 내 티켓을 조회하여 주간 업무 리포트를 위키에 자동 작성한다.

## 실행 절차

1. Jira에서 내 정보 확인
2. 이번 주 업데이트된 내 티켓 전체 조회
3. 상태별(완료/진행중/대기)로 분류하여 보고서 작성 (관리자가 진행 상황을 한눈에 파악하기 위해)
4. 주요 성과, 다음 주 전망, 리스크 요약
5. {스페이스명} 위키에 "주간 업무 리포트 — {이름} ({날짜})" 페이지 생성
```

### PPT 출력 스킬

```yaml
---
name: status-ppt
description: >
  Jira 프로젝트 현황을 PPT 발표자료로 자동 생성한다.
  프로젝트 현황 발표가 필요할 때 사용한다.
triggers:
  - "현황발표"
  - "현황 PPT"
  - "발표자료"
user-invocable: true
---

# 프로젝트 현황 발표자료

Jira 프로젝트 현황을 PPT 발표자료로 자동 생성한다.

## 실행 절차

1. Jira에서 프로젝트 선택
2. 전체 이슈 현황 조회 (완료/진행중/대기)
3. 핵심 지표 정리 (완료율, 주요 마일스톤, 리스크)
4. PPT 파일 생성 (표지, 현황 요약, 상세, 다음 단계)
5. 파일 저장 및 경로 안내
```

### Excel 출력 스킬

```yaml
---
name: ticket-export
description: >
  Jira 티켓 목록을 엑셀 파일로 정리하여 내보낸다.
  티켓 현황을 엑셀로 관리하거나 공유할 때 사용한다.
triggers:
  - "티켓정리"
  - "티켓 엑셀"
  - "티켓 정리"
user-invocable: true
---

# 티켓 엑셀 정리

Jira 티켓 목록을 엑셀 파일로 정리하여 내보낸다.

## 실행 절차

1. Jira에서 프로젝트 선택
2. 기간/상태 필터 확인
3. 티켓 목록 조회 및 정리
4. Excel 파일 생성 (헤더, 조건부 서식, 필터)
5. 파일 저장 및 경로 안내
```

---

## 7. Validation Checklist

스킬 파일 생성 후 아래 항목을 확인한다:

### 구조
- [ ] SKILL.md 파일이 존재하는가
- [ ] YAML frontmatter에 name과 description이 있는가
- [ ] 마크다운 본문이 있는가

### Name 규칙
- [ ] 영문 lowercase + hyphens 형식인가 (한글도 허용)
- [ ] 디렉토리명이 name 필드와 일치하는가

### Description 품질
- [ ] 스킬이 **무엇을 하는지** 구체적으로 설명하는가
- [ ] **언제 사용하는지** 포함하는가
- [ ] triggers에 2개 이상의 키워드가 있는가

### 본문 품질
- [ ] imperative/infinitive form으로 작성되었는가 (동사 시작)
- [ ] MCP 함수명이 포함되지 않았는가
- [ ] 실행 절차가 명확한가
- [ ] **500줄 이내**인가

### 보안
- [ ] 민감 정보(토큰, 비밀번호)가 포함되지 않았는가
- [ ] 하드코딩된 값이 사용자 확인을 거친 값인가

---

## 8. 스킬 충돌 처리

기존 skills/ 디렉토리에 동일 이름 스킬이 있는지 확인한다.

**시스템 스킬 보호**: `connector`, `automation`, `onboarding`, `skill-creator` 이름은 시스템 스킬이므로 사용자 스킬로 생성/업데이트할 수 없다.

이름이 겹치는 경우 (시스템 스킬 제외):
```
"{스킬명}" 스킬이 이미 있어요!

① 기존 스킬을 업데이트하기 (새 버전으로 수정)
② 다른 이름으로 만들기
③ 기존 스킬을 먼저 확인하기

어떻게 할까요?
```

### 기존 스킬 수정

사용자가 "{스킬명} 스킬 수정해줘"라고 요청하면:
1. skills/ 디렉토리에서 해당 스킬을 찾는다
2. 현재 SKILL.md 내용을 읽는다
3. 수정 사항을 확인한다
4. SKILL.md를 업데이트한다 (Edit 도구로 수정)
