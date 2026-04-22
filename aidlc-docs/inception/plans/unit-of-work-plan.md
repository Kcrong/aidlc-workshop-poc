# Unit of Work Plan — 테이블오더 서비스

## Pre-established Unit Boundaries
Application Design 단계에서 다음이 이미 확정되었습니다:
- **Unit 1: Backend** (Express + Prisma + SQLite, 전체 API)
- **Unit 2: Customer Frontend** (React, 고객 태블릿)
- **Unit 3: Admin Frontend** (React, 관리자 웹)
- **Shared Package** (monorepo packages/shared — 타입/상수/스키마)

본 Plan은 위 단위들을 확정 문서화하고, 22개 스토리를 유닛에 매핑하며, 구현 순서/의존 관계를 정리합니다.

## Execution Checklist

- [x] Application Design 로드 완료
- [x] Unit 경계 초안 식별 (Backend / Customer FE / Admin FE / Shared)
- [x] 질문 생성 (아래)
- [x] 사용자 답변 수령 (1-A ~ 6-A)
- [x] 모호성 follow-up 분석 (없음)
- [x] Plan 최종 승인 수령
- [x] `unit-of-work.md` 생성 — 유닛 정의/책임/코드 조직 전략
- [x] `unit-of-work-dependency.md` 생성 — 의존 매트릭스
- [x] `unit-of-work-story-map.md` 생성 — 22 스토리 ↔ 유닛 매핑

---

## Clarifying Questions

---

### Question 1: Monorepo Directory 구조

npm workspaces monorepo의 디렉토리 레이아웃은?

A) **`apps/backend`, `apps/customer-frontend`, `apps/admin-frontend`, `packages/shared`** (권장 — apps/packages 분리)
B) `backend/`, `customer-frontend/`, `admin-frontend/`, `shared/` — 평면 구조
C) `services/backend/`, `web/customer/`, `web/admin/`, `libs/shared/`
X) Other

[Answer]: A

---

### Question 2: Unit 개수 확정

4개(Backend + 2 FE + Shared)로 유지할까요, 아니면 변경?

A) **4개 유지** (Backend, Customer FE, Admin FE, Shared) — 권장
B) 3개로 통합 (Shared를 Backend에 흡수)
C) Customer/Admin FE를 1개 React 앱으로 통합 (라우팅 분리)
X) Other

[Answer]: A

---

### Question 3: Shared Package 배포 모델

Shared package는 어떤 형태로 사용되나요?

A) **로컬 workspace 링크만** (npm publish 없음, 개발·빌드 시 symlink) — 권장
B) 별도 npm 레지스트리 배포
C) 단순 복사 또는 git submodule
X) Other

[Answer]: A

---

### Question 4: 구현 순서 (의존성 기반)

Construction 단계에서의 Unit 구현 순서는?

A) **Shared → Backend → Customer FE & Admin FE (병렬)** — 타입이 앞선 의존 만족 (권장)
B) Backend → Customer FE → Admin FE — 순차
C) 전 유닛 병렬 (의존 차단 시 mock 사용)
X) Other

[Answer]: A

---

### Question 5: 팀 소유권/개발 모드 (PoC 맥락)

이 PoC는 누가 개발하나요? (단위별 역할 분리 정보)

A) **1인/단일 팀 개발** — 책임 분리 불필요, 문서 중심
B) 2~3인 분할 (Backend / FE 분담)
C) 워크샵 참가자들이 각자 유닛 담당
X) Other

[Answer]: A

---

### Question 6: Deployment 모델 (유닛별)

각 유닛의 로컬 실행 방식은?

A) **개발 시**: Backend는 `npm run dev` (watch mode), Frontend는 Vite dev server; **빌드 후**: Backend가 정적 FE 파일을 서빙하거나 각각 실행 — 유연
B) 모든 FE를 빌드해 Backend가 단일 서버로 서빙 (single port)
C) 완전히 독립 — 3개 포트 동시 실행
X) Other

[Answer]: A

---
