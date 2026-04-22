# 테이블오더 서비스 요구사항 문서

## 1. Intent Analysis Summary

| 항목 | 값 |
|---|---|
| **User Request** | "requirements 의 요구사항을 읽고 ai-dlc 를 시작해줘." |
| **Request Type** | New Project (Greenfield) |
| **Scope Estimate** | Multiple Components (Customer UI, Admin UI, Backend API, DB) |
| **Complexity Estimate** | Moderate (다중 사용자 역할, 실시간 통신(SSE), 세션/주문 상태 머신) |
| **Requirements Depth** | Standard |
| **Source Documents** | `requirements/table-order-requirements.md`, `requirements/constraints.md` |

---

## 2. 기술적 결정 사항 (Clarifying Questions 답변 기반)

| # | 항목 | 선택 |
|---|---|---|
| 1 | 배포 환경 | 로컬/개발 환경 중심 (추후 배포 결정) |
| 2 | Backend 스택 | Node.js + TypeScript (Express/NestJS) |
| 3 | Frontend 스택 | React + TypeScript |
| 4 | Database | SQLite (경량, PoC/개발용) |
| 5 | 규모 (MVP) | 소규모 (매장 1~10, 동시 테이블 50 이내) |
| 6 | 이미지 저장소 | 로컬 파일 시스템 |
| 7 | 초기 데이터 | DB 시드 스크립트로 예시 매장 + 메뉴 자동 생성 |
| 8 | 테이블 비밀번호 | 매장/테이블별 고정 (관리자 설정) |
| 9 | 주문 상태 머신 | 엄격한 순서 (대기중 → 준비중 → 완료, 역방향 불가) |
| 10 | 테스트 수준 | 핵심 비즈니스 로직 단위 테스트 (기본) |
| 11 | Security Extension | 비활성화 (PoC) |
| 12 | PBT Extension | 비활성화 |

---

## 3. Functional Requirements (기능 요구사항)

### 3.1 고객용 (Customer)

**FR-C-01 테이블 태블릿 자동 로그인 및 세션 관리**
- 초기 설정: 관리자가 매장 식별자 + 테이블 번호 + 테이블 비밀번호 입력, 로컬 저장
- 이후 자동 로그인 (저장된 정보 사용)
- 테이블 비밀번호는 매장/테이블별 고정 (관리자가 재설정)

**FR-C-02 메뉴 조회 및 탐색**
- 메뉴 화면이 기본 화면으로 항상 표시
- 카테고리별 메뉴 분류
- 메뉴 상세: 메뉴명, 가격, 설명, 이미지
- 카드 레이아웃, 터치 친화적 버튼 (최소 44x44px)

**FR-C-03 장바구니 관리**
- 메뉴 추가/삭제, 수량 증감, 총액 실시간 계산
- 장바구니 비우기
- 클라이언트 측 로컬 저장 (새로고침 시 유지)
- 서버 전송은 주문 확정 시에만 수행

**FR-C-04 주문 생성**
- 주문 내역 최종 확인 → 확정
- 성공 시: 주문 번호 표시, 장바구니 비우기, 5초 후 메뉴 화면 자동 리다이렉트
- 실패 시: 에러 메시지 표시, 장바구니 유지
- 주문 데이터: 매장ID, 테이블ID, 메뉴 목록(메뉴명/수량/단가), 총액, 세션ID

**FR-C-05 주문 내역 조회**
- 현재 테이블 세션의 주문만 표시 (이전 세션/이용 완료 주문 제외)
- 주문별: 번호, 시각, 메뉴/수량, 금액, 상태(대기중/준비중/완료)
- 주문 시간 역순 정렬

### 3.2 관리자용 (Admin)

**FR-A-01 매장 인증**
- 매장 식별자 + 사용자명 + 비밀번호 로그인
- JWT 기반 16시간 세션 유지
- 비밀번호 bcrypt 해싱
- 로그인 시도 제한

**FR-A-02 실시간 주문 모니터링**
- Server-Sent Events (SSE) 기반 실시간 주문 표시 (2초 이내)
- 그리드 레이아웃: 테이블별 카드, 총 주문액, 최근 주문 미리보기
- 카드 클릭 시 전체 메뉴 상세 보기
- 주문 상태 변경 (대기중 → 준비중 → 완료, **엄격한 순서**)
- 신규 주문 시각적 강조

**FR-A-03 테이블 관리**
- 테이블 초기 설정 (테이블 번호 + 비밀번호, 16시간 세션 생성)
- 주문 삭제 (직권 수정): 확인 팝업 → 삭제 → 총액 재계산
- 테이블 세션 관리:
  - 세션 시작: 해당 테이블의 첫 주문 시 자동 시작
  - 세션 종료 (이용 완료): 주문 내역 과거 이력 이동, 현재 주문/총액 0 리셋
- 과거 주문 내역 조회 (시간 역순, 날짜 필터링)

**FR-A-04 메뉴 관리**
- 카테고리별 메뉴 CRUD
- 필드: 메뉴명, 가격, 설명, 카테고리, 이미지 URL
- 노출 순서 조정
- 필수 필드 및 가격 범위 검증

### 3.3 주문 상태 머신 (엄격한 순서)

```
[대기중(PENDING)] → [준비중(PREPARING)] → [완료(COMPLETED)]
                                             ↓
                                    [이용 완료(ARCHIVED)]

추가 동작:
- 삭제(직권): 모든 상태에서 가능 (관리자만)
- 역방향 전환: 불가
```

### 3.4 시드 데이터
- 초기 구동 시 자동 실행되는 DB 시드 스크립트 제공
- 예시 매장 1개 + 관리자 계정 + 카테고리/메뉴 샘플 포함

---

## 4. Non-Functional Requirements (비기능 요구사항)

### 4.1 Performance
- **NFR-PERF-01**: 주문 생성 → 관리자 화면 반영까지 **2초 이내** (SSE)
- **NFR-PERF-02**: MVP 규모 — 매장 1~10개, 동시 테이블 50개 이내 처리
- **NFR-PERF-03**: 메뉴 화면 초기 로딩 3초 이내 (로컬 환경 기준)

### 4.2 Security (PoC 수준, Security Extension 비활성화)
- **NFR-SEC-01**: 관리자 비밀번호 bcrypt 해싱 저장
- **NFR-SEC-02**: 관리자 JWT 기반 인증 (16시간 만료)
- **NFR-SEC-03**: 로그인 시도 제한 (rate limiting)
- **NFR-SEC-04**: 테이블 비밀번호는 평문 저장 허용 (PoC 범위, 프로덕션에서는 해싱 권장)
- **비대상**: OAuth/SNS 로그인, 2FA/OTP (constraints.md 제외)

### 4.3 Scalability
- **NFR-SCALE-01**: 소규모 MVP — SQLite 단일 인스턴스 허용
- **NFR-SCALE-02**: 추후 RDBMS 교체가 용이하도록 ORM(예: Prisma, TypeORM) 사용 권장

### 4.4 Usability
- **NFR-UX-01**: 터치 친화적 UI (버튼 최소 44x44px)
- **NFR-UX-02**: 카드 레이아웃, 명확한 시각 계층
- **NFR-UX-03**: 신규 주문에 대한 시각적 강조 (색상/애니메이션)

### 4.5 Reliability
- **NFR-REL-01**: 주문 실패 시 장바구니 유지 (데이터 손실 방지)
- **NFR-REL-02**: 클라이언트 장바구니 로컬 저장 (새로고침 안전)
- **NFR-REL-03**: SSE 연결 끊김 시 재연결 로직

### 4.6 Testability
- **NFR-TEST-01**: 핵심 비즈니스 로직 단위 테스트 (주문 상태 머신, 장바구니 계산, 세션 라이프사이클, 인증)
- **NFR-TEST-02**: PBT, E2E, 통합 테스트는 MVP 범위 제외

### 4.7 Maintainability
- **NFR-MAINT-01**: TypeScript strict 모드
- **NFR-MAINT-02**: Layered architecture (controller / service / repository)
- **NFR-MAINT-03**: 코드 스타일 린터 적용 (ESLint + Prettier)

### 4.8 Deployment
- **NFR-DEPLOY-01**: 로컬 실행 가능 (npm scripts로 백엔드/프론트엔드 동시 기동)
- **NFR-DEPLOY-02**: 단일 저장소 (monorepo) 권장 — 공유 타입 정의 활용

---

## 5. 제외 항목 (constraints.md 기반)

- 실제 결제/PG 연동, 영수증, 환불, 포인트
- OAuth/SNS 로그인, 2FA/OTP
- 이미지 리사이징/최적화, CMS, 광고
- 푸시/SMS/이메일/소리 알림
- 주방 연동, 식재료 재고
- 매출 리포트, 재고/직원/예약/리뷰, 다국어
- 배달/POS/소셜/지도/번역 API 외부 연동

---

## 6. User Personas (요약)

| Persona | 사용 기기 | 주요 행동 |
|---|---|---|
| **고객 (Customer)** | 테이블 태블릿 (웹 UI) | 메뉴 탐색, 장바구니, 주문, 내 주문 조회 |
| **매장 관리자 (Admin)** | 관리자 PC/태블릿 (웹 UI) | 로그인, 실시간 주문 모니터링, 테이블 관리, 메뉴 CRUD, 직권 주문 삭제 |

---

## 7. 아키텍처 개요 (확정된 기술 결정)

```
┌─────────────────┐         ┌─────────────────┐
│ Customer Web UI │         │  Admin Web UI   │
│  React + TS     │         │  React + TS     │
└────────┬────────┘         └────────┬────────┘
         │ REST                      │ REST + SSE
         └───────────┬───────────────┘
                     │
          ┌──────────▼──────────┐
          │  Node.js + TS API   │
          │  (Express/NestJS)   │
          │ - Auth (JWT/bcrypt) │
          │ - Order/Menu/Table  │
          │ - SSE Broadcaster   │
          └──────────┬──────────┘
                     │ ORM
          ┌──────────▼──────────┐
          │      SQLite DB      │
          │ Stores/Menus/Orders/│
          │ Tables/OrderHistory │
          └─────────────────────┘

Local filesystem (./uploads) — 메뉴 이미지 저장
Seed script — 초기 매장/메뉴/관리자 계정 생성
```

---

## 8. Key Requirements Summary (핵심 요약)

- **역할**: 고객(태블릿) + 관리자(웹)
- **주요 도메인**: Store, Table, TableSession, Menu/Category, Order/OrderItem, OrderHistory, AdminUser
- **핵심 차별 기능**: 실시간 주문 모니터링 (SSE), 테이블 세션 라이프사이클, 주문 상태 머신 (엄격)
- **기술 스택**: Node.js + TypeScript / React + TypeScript / SQLite / bcrypt + JWT
- **테스트**: 단위 테스트 중심
- **범위**: PoC — 로컬 실행, 소규모
