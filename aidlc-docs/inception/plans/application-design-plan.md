# Application Design Plan — 테이블오더 서비스

## Purpose
- Requirements + User Stories 기반의 **고수준** 컴포넌트/서비스 식별
- 업무 규칙 상세는 Functional Design (per-unit Construction) 단계에서 다룸

## Context 요약
- **Stack**: Node.js + TypeScript (Backend), React + TypeScript (Frontend × 2), SQLite
- **Personas**: Diner Dave, Manager Mia
- **Key Domains**: Store / AdminUser / Table / TableSession / Menu(+Category) / Order(+Item) / OrderHistory
- **Key Flows**: 자동 로그인, 주문 생성, SSE 실시간 브로드캐스트, 세션 라이프사이클, 주문 상태 머신

## Execution Checklist

- [x] Requirements 및 Stories 로드
- [x] 핵심 도메인/컴포넌트 후보 식별
- [x] 질문 생성 (아래)
- [x] 사용자 답변 수령 (권장대로 1-A ~ 8-A)
- [x] 모호성 follow-up 분석 (모호성 없음)
- [x] 최종 Plan 승인 수령 (사용자: 권장대로 진행)
- [x] `components.md` 생성 — 컴포넌트 정의/책임/인터페이스
- [x] `component-methods.md` 생성 — 메서드 시그니처
- [x] `services.md` 생성 — 서비스 orchestration 패턴
- [x] `component-dependency.md` 생성 — 의존 관계 + 통신 패턴 + 데이터 흐름
- [x] `application-design.md` 생성 — 통합 문서

---

## Clarifying Questions

각 `[Answer]:` 태그 뒤에 옵션을 입력해 주세요.

---

### Question 1: Backend 아키텍처 스타일

어떤 Backend 아키텍처 스타일을 채택할까요?

A) **Layered (Controller → Service → Repository)** — 단순, 표준적 (권장)
B) Hexagonal / Ports & Adapters — 테스트/확장성 강조, 러닝 커브 존재
C) Feature-Sliced (도메인별 모듈) — 도메인 중심, monolith 내 모듈화
X) Other

[Answer]: A

---

### Question 2: Backend Framework 세부 선택

Question 2의 답변(Node.js + TS) 하위 프레임워크를 확정해 주세요.

A) **Express** — 가볍고 빠른 시작, 수동 구조 (권장 for PoC)
B) NestJS — 의견 강한 구조, DI/Decorator, 러닝 커브
X) Other

[Answer]: A

---

### Question 3: ORM / DB 접근 방식

SQLite 접근 방법은?

A) **Prisma** — schema 선언형, 타입 자동생성, 마이그레이션 편리 (권장)
B) TypeORM — 데코레이터 기반 Entity
C) better-sqlite3 (Raw SQL) — ORM 없이 SQL 작성
X) Other

[Answer]: A

---

### Question 4: SSE 브로드캐스트 구조

실시간 주문 알림 (SSE) 구현 방식은?

A) **In-memory EventEmitter 기반** (단일 프로세스) — PoC 단순 구현 (권장)
B) Redis Pub/Sub — 다중 인스턴스 확장 가능, 추가 의존성
C) WebSocket (SSE 대신) — 양방향
X) Other (요구사항 문서에 SSE 명시되어 A/B 중 선택 권장)

[Answer]: A

---

### Question 5: 인증 방식 세부

관리자 JWT 처리 방식은?

A) **JWT in Authorization header + refresh 없음** (16h 단일 토큰, PoC 적합)
B) JWT Access (짧은 만료) + Refresh Token 방식 — 프로덕션 보안
C) Session Cookie (stateful) — 간단한 UI 친화적
X) Other

[Answer]: A

---

### Question 6: Frontend 상태관리

Frontend (Customer + Admin)의 상태관리 방식은?

A) **React Query (TanStack Query) + Context** — 서버 상태/클라이언트 상태 분리, 권장
B) Redux Toolkit — 중앙 집중
C) Zustand — 가벼운 스토어
D) useState/useReducer만 — 최소 의존성
X) Other

[Answer]: A

---

### Question 7: Monorepo 여부

프로젝트 구조는?

A) **Monorepo (npm workspaces)** — shared types, 빌드 일관성, 권장
B) 별도 저장소 (backend/, customer-frontend/, admin-frontend/ 독립)
C) Single package (Frontend 2개를 한 React 앱으로 라우팅 분리)
X) Other

[Answer]: A

---

### Question 8: API 통신 스타일

Frontend ↔ Backend API 포맷은?

A) **REST + JSON** (+ SSE for 실시간) — 단순, 권장
B) GraphQL
C) tRPC — Type-safe Node<->Node
X) Other

[Answer]: A

---
