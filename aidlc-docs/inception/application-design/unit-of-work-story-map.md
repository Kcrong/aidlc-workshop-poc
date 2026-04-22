# Unit of Work — Story Map

22개 User Stories를 4개 Unit으로 매핑합니다. 각 스토리는 **주 소유 유닛**을 갖고, 연관 유닛이 있는 경우 함께 표기합니다.

---

## Summary

| Unit | Primary Stories | Supporting Stories |
|---|---|---|
| Unit 1 — Backend | 22개 모두 (API/로직 구현) | — |
| Unit 2 — Customer FE | 10개 (US-C-01 ~ US-C-10) | — |
| Unit 3 — Admin FE | 13개 (US-C-01, US-A-01 ~ US-A-12) | — |
| Unit 4 — Shared | 모든 스토리의 DTO/enum 제공 | — |

**Note**: US-C-01 (테이블 초기 설정)은 관리자(Manager Mia)가 Admin FE 또는 별도 설정 UI를 통해 수행 → Admin FE에 포함.

---

## Detailed Story ↔ Unit Mapping

### Epic 1 — Customer: Table Session & Login

| Story | Backend | Customer FE | Admin FE | Shared |
|---|---|---|---|---|
| US-C-01 테이블 태블릿 최초 설정 (관리자) | ✅ `TableService.setupTable`, `POST /api/admin/tables/:id/setup` | — | ✅ Setup UI | ✅ TableDto |
| US-C-02 저장된 정보로 자동 로그인 | ✅ `TableAuthService`, `POST /api/table/login` | ✅ `useTableAuth` + localStorage | — | ✅ TableToken |

### Epic 2 — Customer: Menu Browsing

| Story | Backend | Customer FE | Admin FE | Shared |
|---|---|---|---|---|
| US-C-03 카테고리별 메뉴 조회 | ✅ `GET /api/menus` | ✅ `MenuPage`, `CategoryTabs`, `useMenus` | — | ✅ MenuDto, CategoryDto |
| US-C-04 메뉴 상세 정보 | ✅ (동일 endpoint) | ✅ `MenuDetailModal` | — | ✅ MenuDto |

### Epic 3 — Customer: Cart

| Story | Backend | Customer FE | Admin FE | Shared |
|---|---|---|---|---|
| US-C-05 장바구니 담기 | — | ✅ `useCart.add/increment` | — | ✅ CartItem (client-only) |
| US-C-06 장바구니 수정 + 새로고침 유지 | — | ✅ `useCart` localStorage 동기화 | — | ✅ |

### Epic 4 — Customer: Order Placement

| Story | Backend | Customer FE | Admin FE | Shared |
|---|---|---|---|---|
| US-C-07 주문 확정 | ✅ `OrderService.createOrder`, `POST /api/orders` | ✅ `usePlaceOrder`, 5초 성공 화면 | — | ✅ OrderCreateInput, OrderDto |
| US-C-08 첫 주문 시 세션 자동 시작 | ✅ `SessionService.ensureActiveSession` | — (투명) | — | ✅ TableSessionDto |

### Epic 5 — Customer: Order History

| Story | Backend | Customer FE | Admin FE | Shared |
|---|---|---|---|---|
| US-C-09 현재 세션 주문 내역 | ✅ `GET /api/orders/current` | ✅ `OrderHistoryPage`, `useMyOrders` | — | ✅ OrderDto |
| US-C-10 주문 상태 실시간(또는 폴링) 갱신 | ✅ (기존 endpoint + optional customer SSE/폴링) | ✅ refetch or simple SSE | — | ✅ OrderStatus enum |

### Epic 6 — Admin: Authentication

| Story | Backend | Customer FE | Admin FE | Shared |
|---|---|---|---|---|
| US-A-01 관리자 로그인 (JWT 16h) | ✅ `AuthService.login`, `POST /api/auth/login`, `RateLimiter` | — | ✅ `AdminLoginPage`, `useAdminAuth` | ✅ AdminDto, AuthToken |
| US-A-02 16시간 후 자동 로그아웃 | ✅ JWT 만료, 401 응답 | — | ✅ 401 intercept → 로그인 리다이렉트 | ✅ |

### Epic 7 — Admin: Real-time Order Monitoring

| Story | Backend | Customer FE | Admin FE | Shared |
|---|---|---|---|---|
| US-A-03 테이블별 실시간 대시보드 | ✅ `SseController`, `OrderEventBus`, `GET /api/admin/orders/stream` | — | ✅ `DashboardPage`, `TableCard`, `useOrdersStream` | ✅ OrderEvent discriminated union |
| US-A-04 주문 상세 조회 | ✅ `GET /api/admin/orders/:id` | — | ✅ `OrderDetailModal` | ✅ OrderDetailDto |
| US-A-05 주문 상태 전환 (엄격) | ✅ `OrderService.changeStatus`, `OrderStatusMachine` | — | ✅ `OrderStatusTransitionButtons` | ✅ OrderStatus enum |
| US-A-06 직권 주문 삭제 | ✅ `OrderService.deleteOrder` | — | ✅ `useOrderActions.deleteOrder` + 확인 팝업 | ✅ |

### Epic 8 — Admin: Table & Session Management

| Story | Backend | Customer FE | Admin FE | Shared |
|---|---|---|---|---|
| US-A-07 테이블 이용 완료 (세션 종료) | ✅ `SessionService.closeSession` (트랜잭션) | — | ✅ `TableCard` → "이용 완료" 버튼 + 확인 | ✅ TableSessionDto |
| US-A-08 테이블별 과거 이력 | ✅ `GET /api/admin/tables/:id/history` | — | ✅ `TableHistoryPage`, `useTableHistory` | ✅ OrderHistoryDto |
| US-A-09 테이블 재설정 | ✅ `TableService.resetTable` | — | ✅ `TableAdminPage` | ✅ |

### Epic 9 — Admin: Menu Management

| Story | Backend | Customer FE | Admin FE | Shared |
|---|---|---|---|---|
| US-A-10 메뉴 등록 | ✅ `MenuService.createMenu` | — | ✅ `MenuAdminPage`, `MenuForm` | ✅ MenuCreateInput |
| US-A-11 메뉴 수정/삭제 | ✅ `MenuService.updateMenu/deleteMenu` (soft) | — | ✅ `MenuAdminPage` CRUD UI | ✅ |
| US-A-12 메뉴 노출 순서 | ✅ `MenuService.reorderMenus` | — | ✅ `OrderReorderList` | ✅ |

---

## Coverage Check

- **총 스토리**: 22
- **Backend 커버**: 22 (모든 스토리의 API/로직)
- **Customer FE 커버**: 10 (Customer Epic 1~5 + 일부 상태)
- **Admin FE 커버**: 13 (US-C-01 설정 + US-A-01~A-12 + Epic 7~9)
- **Shared 커버**: 모든 DTO/enum

모든 스토리가 ≥1 유닛에 할당됨 ✅
