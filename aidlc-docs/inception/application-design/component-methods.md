# Component Methods — 테이블오더 서비스

**Note**: 메서드 시그니처와 고수준 목적만 정의. 구체적 비즈니스 규칙은 **Functional Design (per-unit Construction)** 에서 정의.

---

## Backend — Services

### AuthService (관리자 인증)
```ts
class AuthService {
  // 관리자 로그인: storeId + username + password
  login(input: { storeId: string; username: string; password: string }): Promise<{ token: string; admin: AdminDto }>;
  // JWT 검증 → payload 반환
  verifyToken(token: string): AdminTokenPayload;
  // rate limit 체크
  checkLoginAttempts(key: string): void; // throws if exceeded
}
```

### TableAuthService (테이블 자동 로그인)
```ts
class TableAuthService {
  // 매장ID + 테이블번호 + 테이블 비밀번호 검증, 테이블 토큰 발급
  login(input: { storeId: string; tableNumber: string; password: string }): Promise<{ token: string; table: TableDto }>;
  verifyToken(token: string): TableTokenPayload;
}
```

### MenuService
```ts
class MenuService {
  listMenusForCustomer(storeId: string): Promise<MenuWithCategoryDto[]>;
  listMenusForAdmin(storeId: string): Promise<MenuDto[]>;
  createMenu(storeId: string, input: MenuCreateInput): Promise<MenuDto>;
  updateMenu(menuId: string, input: MenuUpdateInput): Promise<MenuDto>;
  deleteMenu(menuId: string): Promise<void>; // soft delete if referenced
  reorderMenus(storeId: string, ordering: { menuId: string; order: number }[]): Promise<void>;
}
```

### OrderService
```ts
class OrderService {
  // 주문 생성 (세션 연결 — 없으면 자동 시작)
  createOrder(input: {
    storeId: string;
    tableId: string;
    items: { menuId: string; quantity: number }[];
  }): Promise<OrderDto>;

  listCurrentSessionOrders(tableId: string): Promise<OrderDto[]>;   // 고객 화면
  listActiveOrdersByStore(storeId: string): Promise<OrderDto[]>;    // 관리자 대시보드 초기 로드

  // 상태 전환 (엄격 머신 검증 — Functional Design에서 상세 규칙)
  changeStatus(orderId: string, next: OrderStatus): Promise<OrderDto>;

  // 직권 삭제 (모든 상태 허용)
  deleteOrder(orderId: string): Promise<void>;

  getOrderDetail(orderId: string): Promise<OrderDetailDto>;
}
```

### TableService
```ts
class TableService {
  listTables(storeId: string): Promise<TableDto[]>;
  setupTable(storeId: string, input: { tableNumber: string; password: string }): Promise<TableDto>;
  resetTable(tableId: string, newPassword: string): Promise<TableDto>; // 활성 세션 있으면 거부
}
```

### SessionService
```ts
class SessionService {
  // 주문 생성 시 내부 호출 — 활성 세션 없으면 새로 생성
  ensureActiveSession(tableId: string): Promise<TableSessionDto>;

  // 매장 이용 완료 — 주문들을 OrderHistory로 이동, 세션 close
  closeSession(tableId: string): Promise<{ archivedOrderCount: number }>;

  listHistory(tableId: string, filter?: { from?: Date; to?: Date }): Promise<OrderHistoryDto[]>;
}
```

### OrderEventBus
```ts
class OrderEventBus {
  publish(event: OrderEvent): void;   // order.created | order.updated | order.deleted | table.closed
  subscribe(storeId: string, handler: (event: OrderEvent) => void): Unsubscribe;
}
```

---

## Backend — Controllers (시그니처 요약)

```ts
// AuthController
POST   /api/auth/login                    -> AuthService.login
GET    /api/auth/me                       -> returns admin from token

// TableAuthController
POST   /api/table/login                   -> TableAuthService.login

// MenuController
GET    /api/menus                         -> MenuService.listMenusForCustomer
GET    /api/admin/menus                   -> MenuService.listMenusForAdmin
POST   /api/admin/menus                   -> MenuService.createMenu
PUT    /api/admin/menus/:id               -> MenuService.updateMenu
DELETE /api/admin/menus/:id               -> MenuService.deleteMenu
PUT    /api/admin/menus/reorder           -> MenuService.reorderMenus

// OrderController
POST   /api/orders                        -> OrderService.createOrder (table auth)
GET    /api/orders/current                -> OrderService.listCurrentSessionOrders
GET    /api/admin/orders                  -> OrderService.listActiveOrdersByStore
GET    /api/admin/orders/:id              -> OrderService.getOrderDetail
PATCH  /api/admin/orders/:id/status       -> OrderService.changeStatus
DELETE /api/admin/orders/:id              -> OrderService.deleteOrder

// TableController
GET    /api/admin/tables                  -> TableService.listTables
POST   /api/admin/tables/:id/setup        -> TableService.setupTable
POST   /api/admin/tables/:id/reset        -> TableService.resetTable

// SessionController
POST   /api/admin/tables/:id/close        -> SessionService.closeSession
GET    /api/admin/tables/:id/history      -> SessionService.listHistory

// SseController
GET    /api/admin/orders/stream           -> OrderEventBus.subscribe, streams SSE
```

---

## Backend — Repositories (Prisma 기반 메서드 개요)

### OrderRepository
```ts
create(data): Promise<Order>
findById(id): Promise<Order | null>
findBySessionId(sessionId): Promise<Order[]>
findActiveByStore(storeId): Promise<Order[]>
updateStatus(id, status): Promise<Order>
delete(id): Promise<void>
sumTotalByTable(tableId): Promise<number>
```

### TableSessionRepository
```ts
findActiveByTable(tableId): Promise<TableSession | null>
create(tableId): Promise<TableSession>
close(sessionId): Promise<TableSession>
```

### MenuRepository
```ts
findAllByStore(storeId, opts?): Promise<Menu[]>
findById(id): Promise<Menu | null>
create(data): Promise<Menu>
update(id, data): Promise<Menu>
softDelete(id): Promise<void>
updateOrder(menuId, order): Promise<void>
```

(AdminUser/Store/Table/Category/OrderHistory Repository들도 동일 패턴 CRUD)

---

## Frontend — Customer (Hooks)

```ts
useTableAuth(): { isAuthenticated, login, logout, tableInfo }
useMenus(): { data: MenuWithCategory[], isLoading }
useCart(): { items, add(menu), increment, decrement, remove, clear, total }
usePlaceOrder(): { mutate(items), isLoading, error }
useMyOrders(): { data: Order[], isLoading, refetch }
```

---

## Frontend — Admin (Hooks)

```ts
useAdminAuth(): { isAuthenticated, login, logout, admin }
useOrdersStream(): { orders, connectionState }   // SSE EventSource 래퍼
useTables(): { tables, refetch }
useOrderActions(): {
  changeStatus(orderId, next),
  deleteOrder(orderId)
}
useTableHistory(tableId, dateRange): { history, isLoading }
useMenuAdmin(): {
  list, create, update, remove, reorder
}
```
