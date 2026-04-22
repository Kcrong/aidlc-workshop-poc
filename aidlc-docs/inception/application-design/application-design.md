# Application Design (Consolidated) — 테이블오더 서비스

본 문서는 Application Design 단계의 네 개 산출물을 통합한 요약 문서입니다. 상세는 각 원본 문서를 참조하세요.

## 확정된 기술 결정 (8개 응답, 모두 권장안)
| 항목 | 선택 |
|---|---|
| Backend 아키텍처 | **Layered** (Controller → Service → Repository) |
| Backend Framework | **Express** + TypeScript |
| ORM | **Prisma** (SQLite) |
| SSE 브로드캐스트 | **In-memory EventEmitter** (단일 프로세스) |
| 관리자 인증 | **JWT** (Authorization header, 16h 단일 토큰) |
| Frontend 상태관리 | **React Query + Context** |
| Repo 구조 | **npm workspaces monorepo** |
| API 스타일 | **REST + JSON** (+ SSE) |

---

## 1. Components (요약 · 원본: `components.md`)

### Backend
- **Controllers** (HTTP): `AuthController`, `TableAuthController`, `MenuController`, `OrderController`, `TableController`, `SessionController`, `SseController`
- **Services** (Business): `AuthService`, `TableAuthService`, `MenuService`, `OrderService`, `TableService`, `SessionService`, `OrderEventBus`
- **Repositories** (Persistence): `AdminUserRepository`, `StoreRepository`, `TableRepository`, `TableSessionRepository`, `MenuRepository`, `OrderRepository`, `OrderHistoryRepository`
- **Cross-cutting**: `AuthMiddleware`, `TableAuthMiddleware`, `RateLimiter`, `ErrorHandler`, `Logger`, `SeedRunner`

### Customer Frontend
- Pages: `SetupPage`, `MenuPage`, `CartPage`, `OrderConfirmPage`, `OrderHistoryPage`
- Hooks: `useTableAuth`, `useMenus`, `useCart`, `usePlaceOrder`, `useMyOrders`

### Admin Frontend
- Pages: `AdminLoginPage`, `DashboardPage`, `OrderDetailModal`, `TableHistoryPage`, `MenuAdminPage`, `TableAdminPage`
- Hooks: `useAdminAuth`, `useOrdersStream`, `useTables`, `useOrderActions`, `useMenuAdmin`, `useTableHistory`

### Shared (monorepo packages/shared)
- `types/` (DTO), `constants/` (OrderStatus enum), `schemas/` (zod)

---

## 2. Component Methods (요약 · 원본: `component-methods.md`)

주요 서비스 시그니처 (business rules는 Functional Design에서):

```ts
AuthService.login({ storeId, username, password }): Promise<{ token, admin }>

TableAuthService.login({ storeId, tableNumber, password }): Promise<{ token, table }>

OrderService.createOrder({ storeId, tableId, items }): Promise<OrderDto>
OrderService.changeStatus(orderId, next): Promise<OrderDto>
OrderService.deleteOrder(orderId): Promise<void>

SessionService.ensureActiveSession(tableId): Promise<TableSessionDto>
SessionService.closeSession(tableId): Promise<{ archivedOrderCount }>

MenuService: CRUD + reorder
TableService: setup, reset, list
OrderEventBus: publish / subscribe
```

REST endpoints (요약):
```
POST /api/auth/login | GET /api/auth/me
POST /api/table/login
GET/POST/PUT/DELETE /api/[admin/]menus
POST /api/orders | GET /api/orders/current
GET /api/admin/orders | GET /api/admin/orders/:id
PATCH /api/admin/orders/:id/status | DELETE /api/admin/orders/:id
GET /api/admin/tables | POST /api/admin/tables/:id/setup | POST .../reset
POST /api/admin/tables/:id/close | GET /api/admin/tables/:id/history
GET /api/admin/orders/stream                   (SSE)
```

---

## 3. Services / Orchestration (요약 · 원본: `services.md`)

핵심 orchestration 패턴:

### OrderService.createOrder (orchestrator)
```
1) table context 검증 (TableAuthMiddleware)
2) MenuRepository.findByIds
3) SessionService.ensureActiveSession
4) OrderRepository.create
5) OrderEventBus.publish(order.created) → SSE 구독자로 전파
```

### SessionService.closeSession (트랜잭션)
```
Prisma.$transaction:
  - findBySessionId orders
  - for each: OrderHistory 생성 + Order 삭제
  - TableSession close
OrderEventBus.publish(table.closed)
```

### SSE pattern
```
SseController
  → OrderEventBus.subscribe(storeId, handler)
  → handler: res.write(`data: ${event}\n\n`)
  → on close: unsubscribe
```

---

## 4. Component Dependency (요약 · 원본: `component-dependency.md`)

### Communication Patterns
| Pattern | 사용처 |
|---|---|
| REST (JSON) | FE ↔ BE 모든 요청 |
| SSE | `GET /api/admin/orders/stream` (BE → Admin) |
| In-process Pub/Sub | `OrderEventBus` (EventEmitter) |
| DB Transaction | `SessionService.closeSession` |

### High-level Backend Layering
```
HTTP Request
  → Middleware (Auth, RateLimit, Error)
  → Controller
  → Service (Orchestration + Domain Logic)
  → Repository (Prisma)
  → SQLite
```

### Unit Boundary (Units Generation 예고)
| Unit | 포함 |
|---|---|
| **Unit 1: Backend** | Express API 전체 (Controllers, Services, Repositories, Middleware, EventBus, Prisma, Seed) |
| **Unit 2: Customer FE** | React + TS 고객 태블릿 UI |
| **Unit 3: Admin FE** | React + TS 관리자 UI |
| **(Shared)** | `packages/shared`의 타입/상수/스키마 (각 Unit에 import) |

---

## 5. 다음 단계 예고

**Units Generation** 단계에서:
- 위 3 + 1 Unit 구조 확정 및 각 Unit 의 책임/인터페이스/의존 명확화
- Monorepo 디렉토리 구조 (`apps/backend`, `apps/customer-frontend`, `apps/admin-frontend`, `packages/shared`) 결정
- Unit별 Construction 순서 (의존성 반영: Shared → Backend → FE(병렬)) 제안
