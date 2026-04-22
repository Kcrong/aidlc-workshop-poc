# Components — 테이블오더 서비스

## 확정 기술 스택 (Application Design 답변 기반)
- **Backend**: Node.js + TypeScript, **Express** (Layered 아키텍처)
- **ORM**: **Prisma** (SQLite)
- **SSE**: In-memory EventEmitter
- **Auth**: JWT (Authorization header, 16h 단일 토큰, refresh 없음)
- **Frontend**: React + TypeScript, **React Query + Context**
- **Repo 구조**: **npm workspaces monorepo**
- **API**: REST + JSON (+ SSE)

---

## 1. Backend Components

### 1.1 Controllers (HTTP Layer)
| Component | 책임 | 주요 엔드포인트 |
|---|---|---|
| `AuthController` | 관리자 로그인/토큰 검증 endpoint | `POST /api/auth/login`, `GET /api/auth/me` |
| `MenuController` | 메뉴/카테고리 CRUD, 공개 조회 | `GET /api/menus`, `POST/PUT/DELETE /api/admin/menus` |
| `OrderController` | 주문 생성/조회, 상태 변경, 삭제 | `POST /api/orders`, `GET /api/orders/current`, `PATCH /api/admin/orders/:id/status`, `DELETE /api/admin/orders/:id` |
| `TableController` | 테이블 초기 설정/재설정, 현재 테이블 조회 | `POST /api/admin/tables/:id/setup`, `GET /api/admin/tables` |
| `SessionController` | 테이블 세션 종료(이용 완료), 과거 이력 조회 | `POST /api/admin/tables/:id/close`, `GET /api/admin/tables/:id/history` |
| `TableAuthController` | 테이블 자격 인증 (자동 로그인) | `POST /api/table/login` |
| `SseController` | 관리자 실시간 주문 스트림 | `GET /api/admin/orders/stream` (text/event-stream) |

### 1.2 Services (Business Layer)
| Component | 책임 |
|---|---|
| `AuthService` | 관리자 자격 검증, bcrypt 비교, JWT issue/verify, rate limiting |
| `TableAuthService` | 테이블 자격(매장ID+테이블번호+비밀번호) 검증, 테이블 토큰 발급 |
| `MenuService` | 메뉴/카테고리 CRUD, 노출 순서 조정, soft delete 처리 |
| `OrderService` | 주문 생성(세션 연결), 세션별 조회, 상태 전환(엄격 머신), 직권 삭제, 총액 재계산 |
| `TableService` | 테이블 초기 설정/재설정, 테이블 목록 조회 |
| `SessionService` | 세션 start(첫 주문 시 자동), close(이용 완료 → OrderHistory 이동) |
| `OrderEventBus` | SSE 브로드캐스트용 EventEmitter 래퍼 (order.created/updated/deleted, table.closed) |

### 1.3 Repositories (Persistence Layer)
Prisma Client 기반, 각 도메인 엔티티별 데이터 접근:

| Component | 대상 모델 |
|---|---|
| `AdminUserRepository` | AdminUser |
| `StoreRepository` | Store |
| `TableRepository` | Table |
| `TableSessionRepository` | TableSession |
| `MenuRepository` | Menu, Category |
| `OrderRepository` | Order, OrderItem |
| `OrderHistoryRepository` | OrderHistory, OrderHistoryItem |

### 1.4 Cross-cutting Components
| Component | 책임 |
|---|---|
| `AuthMiddleware` | JWT 검증, `req.admin` 주입 |
| `TableAuthMiddleware` | 테이블 토큰 검증, `req.table` 주입 |
| `RateLimiter` | 로그인 시도 rate limiting (메모리 기반, IP+storeId 키) |
| `ErrorHandler` | 표준 에러 응답 포맷 (code / message / details) |
| `Logger` | 요청/에러 로깅 (console/pino) |
| `SeedRunner` | 초기 시드 데이터 생성 (Store/AdminUser/Category/Menu) |

---

## 2. Customer Frontend Components

### 2.1 Pages
- `SetupPage` — 테이블 초기 설정 (관리자 1회)
- `MenuPage` — 메뉴 화면 (기본)
- `CartPage` — 장바구니 (or 슬라이드 패널)
- `OrderConfirmPage` — 주문 확정 및 성공 화면
- `OrderHistoryPage` — 현재 세션 주문 내역

### 2.2 UI Components
- `CategoryTabs`, `MenuCard`, `MenuDetailModal`
- `CartBadge`, `CartItem`, `CartSummary`
- `OrderSuccessBanner` (주문 번호 + 5초 자동 리다이렉트)
- `OrderStatusChip` (대기중/준비중/완료 색상)

### 2.3 Hooks / Data
- `useTableAuth` — 로컬 저장/자동 로그인
- `useMenus` — 메뉴 조회 (React Query)
- `useCart` — localStorage 동기화 장바구니
- `usePlaceOrder` — 주문 생성 mutation
- `useMyOrders` — 현재 세션 주문 조회

---

## 3. Admin Frontend Components

### 3.1 Pages
- `AdminLoginPage` — 로그인
- `DashboardPage` — 테이블 그리드 + 실시간 주문
- `OrderDetailModal` — 주문 상세 + 상태 전환 + 삭제
- `TableHistoryPage` — 테이블별 과거 이력
- `MenuAdminPage` — 메뉴 CRUD + 노출 순서
- `TableAdminPage` — 테이블 목록, 재설정, 이용 완료

### 3.2 UI Components
- `TableCard` (테이블 번호, 총액, 최근 주문 미리보기, 이용 완료 버튼)
- `OrderRow`, `OrderStatusTransitionButtons` (엄격 머신 반영, 역방향 버튼 비활성)
- `MenuForm`, `MenuList`, `OrderReorderList`
- `LiveIndicator` (SSE 연결 상태)

### 3.3 Hooks / Data
- `useAdminAuth` — JWT 저장/갱신 불필요(16h 단일)
- `useOrdersStream` — SSE EventSource 래퍼 (자동 재연결)
- `useTables`, `useMenuAdmin`, `useOrderActions`
- `useTableHistory`

---

## 4. Shared (monorepo packages/shared)

- `types/` — DTO 타입 정의 (Menu, Order, TableSession, AdminUser 등)
- `constants/` — OrderStatus enum, error codes
- `schemas/` — 입력 검증 zod 스키마 (선택)

---

## 5. Component Interface 요약

| 컴포넌트 카테고리 | 입력 | 출력 | 외부 의존 |
|---|---|---|---|
| Controllers | HTTP Request | HTTP Response | Services |
| Services | 도메인 파라미터 | 도메인 객체 / 에러 | Repositories, EventBus |
| Repositories | Prisma 쿼리 파라미터 | Entity | Prisma Client |
| Middlewares | Request | Request+Context / Error | AuthService |
| EventBus | domain event | subscriber 호출 | EventEmitter (node:events) |
| Frontend Hooks | UI state / params | React Query data + mutations | fetch / EventSource |
