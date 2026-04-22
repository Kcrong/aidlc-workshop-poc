# Unit of Work — 테이블오더 서비스

## Confirmed Decisions (Plan Answers)
- **Directory layout**: `apps/` + `packages/` 분리
- **Unit 개수**: 4개 (Backend + Customer FE + Admin FE + Shared)
- **Shared 배포**: 로컬 workspace 링크 (npm publish 없음)
- **구현 순서**: Shared → Backend → Customer/Admin FE 병렬
- **팀 소유권**: 1인/단일 팀 (문서 중심)
- **Deployment**: 개발 시 독립 dev server, 빌드 후 유연한 실행

---

## Monorepo Directory Structure (Greenfield Code Organization)

```
aidlc-workshop-poc/                  # Workspace root
├── package.json                     # npm workspaces 루트
├── tsconfig.base.json               # 공유 TypeScript config
├── .env.example                     # 공유 환경 변수 템플릿
│
├── apps/
│   ├── backend/                     # Unit 1 — Backend API
│   │   ├── src/
│   │   │   ├── controllers/
│   │   │   ├── services/
│   │   │   ├── repositories/
│   │   │   ├── middleware/
│   │   │   ├── events/              # OrderEventBus
│   │   │   ├── utils/
│   │   │   ├── seed.ts              # SeedRunner
│   │   │   └── index.ts             # Express app entry
│   │   ├── prisma/
│   │   │   └── schema.prisma
│   │   ├── tests/                   # 단위 테스트
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── customer-frontend/           # Unit 2 — Customer Tablet UI
│   │   ├── src/
│   │   │   ├── pages/
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   ├── api/                 # React Query client
│   │   │   └── main.tsx
│   │   ├── public/
│   │   ├── index.html
│   │   ├── package.json             # vite + react + rq
│   │   ├── tsconfig.json
│   │   └── vite.config.ts
│   │
│   └── admin-frontend/              # Unit 3 — Admin Web UI
│       ├── src/
│       │   ├── pages/
│       │   ├── components/
│       │   ├── hooks/
│       │   ├── api/                 # SSE EventSource wrapper
│       │   └── main.tsx
│       ├── public/
│       ├── index.html
│       ├── package.json
│       ├── tsconfig.json
│       └── vite.config.ts
│
└── packages/
    └── shared/                       # Unit 4 — Shared package
        ├── src/
        │   ├── types/                # DTO / Entity 타입
        │   ├── constants/            # OrderStatus enum, error codes
        │   ├── schemas/              # zod schemas (선택)
        │   └── index.ts
        ├── package.json              # "name": "@tableorder/shared"
        └── tsconfig.json
```

---

## Unit Definitions

### Unit 1 — Backend API (`apps/backend`)

**Goal**: REST API + SSE 스트림 제공, Prisma/SQLite 데이터 영속화, 시드 데이터 생성

**Responsibilities**:
- 관리자/테이블 인증 (JWT, bcrypt)
- 메뉴/주문/테이블/세션 CRUD 및 비즈니스 로직
- 주문 상태 머신 (엄격한 순서) 강제
- 세션 라이프사이클 (시작/종료 → OrderHistory 이동)
- OrderEventBus → SSE 스트림
- Seed script

**Tech**: Node.js 20+, TypeScript, Express, Prisma, SQLite, jsonwebtoken, bcrypt, zod, node:events

**External Interface**:
- HTTP REST 엔드포인트 (상세: `component-methods.md`)
- SSE 엔드포인트: `GET /api/admin/orders/stream`

**Dependencies (other units)**: `@tableorder/shared` (types, constants)

---

### Unit 2 — Customer Frontend (`apps/customer-frontend`)

**Goal**: 테이블 태블릿용 고객 주문 UI

**Responsibilities**:
- 테이블 자격 저장/자동 로그인
- 메뉴 조회/탐색 (카테고리, 카드 레이아웃)
- 장바구니 관리 (localStorage 지속)
- 주문 생성 플로우 (5초 성공 화면 → 메뉴 리다이렉트)
- 현재 세션 주문 내역 조회

**Tech**: React 18, TypeScript, Vite, @tanstack/react-query, React Router

**External Interface**: Backend REST API

**Dependencies**: `@tableorder/shared`, `apps/backend` (런타임 API)

---

### Unit 3 — Admin Frontend (`apps/admin-frontend`)

**Goal**: 매장 관리자용 웹 UI (데스크톱/태블릿)

**Responsibilities**:
- 관리자 로그인 (JWT 16h)
- 실시간 주문 대시보드 (SSE EventSource)
- 주문 상세/상태 전환/직권 삭제
- 테이블 관리 (초기 설정/재설정/이용 완료)
- 메뉴 CRUD/순서 조정
- 과거 주문 이력 조회

**Tech**: React 18, TypeScript, Vite, @tanstack/react-query, React Router, native EventSource

**External Interface**: Backend REST API + SSE

**Dependencies**: `@tableorder/shared`, `apps/backend`

---

### Unit 4 — Shared Package (`packages/shared`)

**Goal**: 타입/상수/스키마를 모든 apps가 import 하도록 중앙화

**Responsibilities**:
- DTO 타입 정의 (MenuDto, OrderDto, TableDto, TableSessionDto, AdminDto, OrderStatus enum 등)
- Error code 상수
- zod 스키마 (선택 — 입력 검증 공용)

**Tech**: TypeScript 전용 (런타임 의존 없음 또는 최소 — zod만 선택)

**External Interface**: `@tableorder/shared` 패키지로 workspace 링크

**Dependencies**: 없음 (leaf)

**Distribution**: 로컬 workspace link만 (npm publish 없음)

---

## Build & Run Strategy (Deployment Model = A)

### 개발 모드
```bash
# 터미널 1
npm run dev --workspace=apps/backend           # tsx watch, :4000

# 터미널 2
npm run dev --workspace=apps/customer-frontend # vite, :5173

# 터미널 3
npm run dev --workspace=apps/admin-frontend    # vite, :5174
```

Vite dev proxy로 `/api` 및 `/api/admin/orders/stream`을 backend로 forward.

### 프로덕션-유사 (로컬)
```bash
npm run build   # 모든 workspace 빌드
npm run start   # Backend만 실행 (정적 FE 서빙 옵션)
```

### 시드
```bash
npm run seed --workspace=apps/backend
```

---

## Story Coverage Summary
모든 22개 스토리 유닛 매핑 → `unit-of-work-story-map.md` 참고.

---

## Next Stages (per-unit Construction)
각 유닛에 대해 Construction 단계 반복:
1. Unit 4 **Shared** (타입 먼저) → Functional/NFR skip 가능, Code Generation만
2. Unit 1 **Backend** → Functional Design + NFR Req/Design + Code Generation
3. Unit 2 **Customer FE** & Unit 3 **Admin FE** (병렬) → 동일 단계
4. **Build and Test** (전체)
