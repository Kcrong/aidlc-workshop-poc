# Story Generation Plan — 테이블오더 서비스

## Assessment Result
- User Stories 실행 결정: **Yes** (see `user-stories-assessment.md`)

## Methodology
- Role: Product Owner
- Focus: 요구사항(requirements.md)을 페르소나 중심의 사용자 스토리로 변환
- Format: **"As a <persona>, I want <capability>, so that <value>"** + Acceptance Criteria (Given/When/Then)
- Compliance: INVEST (Independent / Negotiable / Valuable / Estimable / Small / Testable)

## Plan Execution Checklist

### Planning 단계
- [x] Requirements 및 기술 결정 사항 로드
- [x] User Stories 필요성 평가 (assessment.md)
- [x] 질문 생성 (아래 섹션)
- [x] 사용자 답변 수령
- [x] 모호한 답변 follow-up 분석 (모호성 없음)
- [x] 계획 최종 승인 수령

### Generation 단계 (승인 후 실행)
- [x] personas.md 생성 — Diner Dave, Manager Mia
- [x] stories.md 생성 (9 Epics, 22 stories)
  - [x] Epic 1: Customer — Table Session & Login
  - [x] Epic 2: Customer — Menu Browsing
  - [x] Epic 3: Customer — Cart Management
  - [x] Epic 4: Customer — Order Placement
  - [x] Epic 5: Customer — Order History View (Current Session)
  - [x] Epic 6: Admin — Authentication
  - [x] Epic 7: Admin — Real-time Order Monitoring (SSE)
  - [x] Epic 8: Admin — Table & Session Management
  - [x] Epic 9: Admin — Menu Management
- [x] 각 스토리 INVEST 준수 검증
- [x] 각 스토리 Given/When/Then 수락 기준 포함
- [x] Persona ↔ Story 매핑 포함

---

## Clarifying Questions

각 `[Answer]:` 태그 뒤에 선택한 옵션 문자(A, B, C, ...)를 입력해 주세요.
답변 완료 후 "완료" 또는 "done"이라고 알려주세요.

---

### Question 1: 스토리 Breakdown 접근법

어떤 방식으로 스토리를 구조화할까요?

A) **Persona-Based + Feature-Based 하이브리드** (권장) — 페르소나별 Epic, Epic 하위에 기능 스토리
B) User Journey-Based — 고객/관리자 workflow 시간순 흐름으로 구성
C) Feature-Based만 — 기능(메뉴/주문/테이블 등) 단위로 구성, 페르소나 태그로 구분
D) Epic-Based 계층형 — Epic > Feature > Story 3단계 계층
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

### Question 2: Acceptance Criteria 형식

수락 기준을 어떤 형식으로 작성할까요?

A) **Given/When/Then (Gherkin 스타일)** — 테스트 자동화 친화적
B) 체크리스트 형식 (- [ ] 조건) — 간결
C) 자유 문장 서술 — 최소 형식
X) Other

[Answer]: A

---

### Question 3: 스토리 granularity (크기)

한 스토리의 크기는 어느 수준으로 할까요?

A) **Small (1~2일 작업 단위, MVP용)** — 기능별 세부 스토리
B) Medium (3~5일 단위) — 여러 하위 기능 묶어서
C) Large — Epic 수준, 하위 태스크는 Construction에서 분해
X) Other

[Answer]: A

---

### Question 4: 에러/예외 시나리오 포함 여부

스토리에 에러/예외 시나리오(예: 주문 실패, 세션 만료, 잘못된 비밀번호)를 어떻게 다룰까요?

A) **각 핵심 스토리의 Acceptance Criteria 내에 Negative 케이스 포함** (권장)
B) 별도의 "Error Handling" Epic으로 분리
C) MVP에서는 Happy Path만 다루고 에러는 NFR로만 처리
X) Other

[Answer]: A

---

### Question 5: 우선순위 (MVP 범위 표시)

요구사항 문서상 모든 기능이 MVP 범위지만, 스토리에 우선순위를 표기할까요?

A) **MoSCoW 표기 (Must/Should/Could/Won't)** — 모든 기능 Must로 시작, Construction에서 조정
B) 단순 우선순위 (P0/P1/P2)
C) 표기하지 않음 (전부 MVP)
X) Other

[Answer]: A

---

### Question 6: 페르소나 상세 수준

페르소나는 어느 수준으로 작성할까요?

A) **기본 수준** — 이름/역할/목표/Pain Points/주요 행동 (권장)
B) 상세 수준 — 인구통계/기술 숙련도/시나리오/감정까지
C) 최소 수준 — 역할과 책임만
X) Other

[Answer]: A

---
