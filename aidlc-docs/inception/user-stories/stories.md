# User Stories — 테이블오더 서비스

## Overview
- **Format**: As a <persona>, I want <capability>, so that <value>
- **Acceptance Criteria**: Given / When / Then (Gherkin 스타일), Negative 케이스 포함
- **Priority**: MoSCoW (M=Must, S=Should, C=Could, W=Won't)
- **INVEST**: Independent / Negotiable / Valuable / Estimable / Small / Testable
- **Personas**: Diner Dave (고객), Manager Mia (관리자) — `personas.md` 참고

---

# Epic 1 — Customer: Table Session & Login

## US-C-01 테이블 태블릿 최초 설정 (관리자 1회)
**As a** Manager Mia,  
**I want** 새 테이블 태블릿에 매장 식별자/테이블 번호/테이블 비밀번호를 한 번 입력해 설정하고,  
**so that** 이후 고객이 별도 로그인 없이 사용할 수 있다.  
**Priority**: Must

**Acceptance Criteria**
- Given 새 태블릿의 초기 설정 화면
- When 관리자가 매장 ID, 테이블 번호, 테이블 비밀번호를 입력하고 저장
- Then 로컬 저장소에 자격 정보가 저장되고 다음 화면은 고객 메뉴 화면
- And **Negative**: 필수 필드 누락 시 저장 버튼 비활성, 비밀번호 불일치(서버 검증 실패) 시 에러 메시지 표시

## US-C-02 저장된 정보로 자동 로그인
**As a** Diner Dave,  
**I want** 태블릿을 켜자마자 바로 메뉴 화면을 보고,  
**so that** 기다림 없이 바로 주문을 시작할 수 있다.  
**Priority**: Must

**Acceptance Criteria**
- Given 로컬에 유효한 테이블 자격 정보가 저장되어 있음
- When 태블릿 웹 페이지 로드
- Then 자동으로 세션 인증 후 메뉴 화면 표시 (사용자 입력 불필요)
- And **Negative**: 저장 정보가 없거나 서버에서 거부되면 초기 설정 화면으로 이동

---

# Epic 2 — Customer: Menu Browsing

## US-C-03 카테고리별 메뉴 조회
**As a** Diner Dave,  
**I want** 카테고리별로 메뉴를 한눈에 보고,  
**so that** 원하는 음식을 쉽게 찾을 수 있다.  
**Priority**: Must

**Acceptance Criteria**
- Given 로그인된 테이블 세션
- When 메뉴 화면에 진입
- Then 카테고리 탭/리스트가 표시되고 각 카테고리의 메뉴 카드(이름/가격/이미지)가 표시됨
- And 카테고리 전환 시 3초 이내 해당 메뉴 목록 표시
- And **Negative**: 메뉴가 없는 카테고리는 "메뉴 없음" 메시지 표시

## US-C-04 메뉴 상세 정보 확인
**As a** Diner Dave,  
**I want** 메뉴의 이미지/설명/가격을 자세히 보고,  
**so that** 무엇을 주문할지 정확히 판단할 수 있다.  
**Priority**: Must

**Acceptance Criteria**
- Given 메뉴 화면의 메뉴 카드
- When 카드를 탭함
- Then 메뉴명, 가격, 설명, 이미지가 상세 뷰로 표시됨
- And 터치 버튼은 최소 44x44px 준수
- And **Negative**: 이미지 URL 로드 실패 시 placeholder 표시

---

# Epic 3 — Customer: Cart Management

## US-C-05 장바구니에 메뉴 담기
**As a** Diner Dave,  
**I want** 메뉴를 장바구니에 담고 수량을 조절하고,  
**so that** 한 번에 여러 품목을 주문할 수 있다.  
**Priority**: Must

**Acceptance Criteria**
- Given 메뉴 상세 뷰
- When "담기" 버튼 탭
- Then 장바구니에 해당 메뉴가 추가되고 총 금액이 즉시 갱신
- And 같은 메뉴 재추가 시 수량이 +1
- And **Negative**: 가격이 숫자가 아닌 비정상 메뉴는 담기 불가

## US-C-06 장바구니 수정/비우기 + 새로고침 유지
**As a** Diner Dave,  
**I want** 장바구니 항목을 수정/삭제하고 새로고침에도 내용이 유지되기를,  
**so that** 실수로 페이지를 리로드해도 다시 담을 필요가 없다.  
**Priority**: Must

**Acceptance Criteria**
- Given 장바구니에 2개 이상 항목이 담김
- When 수량 -/+ 버튼 조작 또는 삭제 버튼 탭
- Then 항목/총액이 즉시 갱신되고 localStorage에 저장
- And 페이지 새로고침 후에도 장바구니 상태 유지
- When "비우기" 버튼 탭 및 확인
- Then 장바구니가 모두 삭제됨
- And **Negative**: 수량을 0 이하로 내리려 하면 해당 항목 삭제로 처리

---

# Epic 4 — Customer: Order Placement

## US-C-07 주문 확정
**As a** Diner Dave,  
**I want** 장바구니 내용을 주문으로 확정하고,  
**so that** 주방/관리자에게 전달되어 조리가 시작된다.  
**Priority**: Must

**Acceptance Criteria**
- Given 장바구니에 1개 이상 항목이 있음
- When "주문 확정" 버튼 탭
- Then 서버가 주문을 생성하고 주문 번호를 반환
- And 화면에 주문 번호 + 성공 메시지가 5초간 표시
- And 장바구니가 자동으로 비워지고 5초 후 메뉴 화면으로 자동 리다이렉트
- And **Negative**: 서버 에러 시 에러 메시지 표시되고 장바구니는 유지됨 (재시도 가능)

## US-C-08 첫 주문 시 테이블 세션 자동 시작
**As a** Diner Dave,  
**I want** 내가 첫 주문을 하면 새 테이블 세션이 자동으로 시작되기를,  
**so that** 내 주문이 이전 고객의 주문과 섞이지 않는다.  
**Priority**: Must

**Acceptance Criteria**
- Given 해당 테이블에 현재 활성 세션 없음 (이전 고객 이용 완료 상태)
- When 고객이 첫 주문을 확정
- Then 서버가 새 TableSession을 생성하고 세션 ID를 주문에 연결
- And 이후 같은 테이블의 주문들은 같은 세션 ID에 묶임
- And **Negative**: 이미 활성 세션이 있다면 기존 세션에 묶어야 함 (새 세션 생성 금지)

---

# Epic 5 — Customer: Order History (Current Session)

## US-C-09 현재 세션 주문 내역 조회
**As a** Diner Dave,  
**I want** 지금 내 테이블 세션의 주문 목록과 상태를 확인하고,  
**so that** 조리 진행 상황을 알 수 있다.  
**Priority**: Must

**Acceptance Criteria**
- Given 현재 세션에 1개 이상의 주문이 있음
- When 고객이 "주문 내역" 메뉴에 진입
- Then 현재 세션 주문이 시간 역순으로 표시됨 (번호, 시각, 메뉴, 금액, 상태)
- And 이전 세션/이용 완료된 주문은 표시되지 않음
- And **Negative**: 주문이 없으면 "주문 내역 없음" 메시지 표시

## US-C-10 주문 상태 실시간(또는 폴링) 갱신
**As a** Diner Dave,  
**I want** 주문 상태가 변경되면 내 화면에도 반영되기를,  
**so that** 언제 음식이 준비되는지 알 수 있다.  
**Priority**: Should

**Acceptance Criteria**
- Given 주문 내역 화면
- When 관리자가 주문 상태를 변경
- Then 고객 화면의 상태가 수 초 내 갱신 (폴링 또는 SSE)
- And **Negative**: 네트워크 단절 시 마지막 상태를 유지하며 재연결 시 최신 상태로 동기화

---

# Epic 6 — Admin: Authentication

## US-A-01 관리자 로그인 (JWT 16h)
**As a** Manager Mia,  
**I want** 매장ID + 아이디 + 비밀번호로 로그인하고,  
**so that** 매장 관리 기능을 사용할 수 있다.  
**Priority**: Must

**Acceptance Criteria**
- Given 유효한 관리자 자격 정보
- When 로그인 폼 제출
- Then JWT가 발급되고 16시간 세션이 시작됨
- And 브라우저 새로고침 후에도 유효 기간 내라면 세션 유지
- And **Negative**: 잘못된 자격 → 실패 메시지; 5회 이상 실패 시 rate limit 적용

## US-A-02 16시간 후 자동 로그아웃
**As a** Manager Mia,  
**I want** 세션이 만료되면 자동 로그아웃되기를,  
**so that** 매장 PC를 두고 퇴근해도 보안이 유지된다.  
**Priority**: Must

**Acceptance Criteria**
- Given 16시간이 경과한 세션
- When API 호출 또는 페이지 접근 시도
- Then 401 반환 및 로그인 화면으로 리다이렉트
- And **Negative**: 만료 임박 시 선택적으로 경고 표시 (구현 선택)

---

# Epic 7 — Admin: Real-time Order Monitoring (SSE)

## US-A-03 테이블별 실시간 주문 대시보드
**As a** Manager Mia,  
**I want** 모든 테이블의 현재 주문 상황을 한눈에 보고,  
**so that** 신규 주문을 놓치지 않는다.  
**Priority**: Must

**Acceptance Criteria**
- Given 관리자가 로그인한 상태
- When 대시보드에 진입
- Then 테이블별 카드(테이블 번호/총액/최근 주문 미리보기)가 그리드로 표시됨
- And SSE 연결이 수립되어 신규 주문은 **2초 이내** 카드에 반영
- And 신규 주문은 시각적 강조(색상/애니메이션) 적용
- And **Negative**: SSE 연결 끊김 시 자동 재연결 시도 (exponential backoff)

## US-A-04 주문 상세 조회
**As a** Manager Mia,  
**I want** 주문 카드를 눌러 전체 메뉴 목록을 확인하고,  
**so that** 조리 준비를 정확히 할 수 있다.  
**Priority**: Must

**Acceptance Criteria**
- Given 대시보드의 주문 카드
- When 카드를 클릭
- Then 주문 번호, 시각, 전체 메뉴/수량/단가, 총액, 상태가 모달 또는 상세 화면으로 표시됨
- And **Negative**: 해당 주문이 이미 삭제된 경우 "주문 없음" 메시지 + 대시보드 새로고침

## US-A-05 주문 상태 전환 (엄격한 순서)
**As a** Manager Mia,  
**I want** 주문 상태를 대기중 → 준비중 → 완료로 전환하고,  
**so that** 조리 진행 상황을 기록하고 고객에게 공유한다.  
**Priority**: Must

**Acceptance Criteria**
- Given 주문 상세 뷰 (현재 상태: 대기중)
- When "준비중으로 변경" 버튼 클릭
- Then 상태가 준비중으로 변경되고 대시보드/고객 화면에 반영
- When 준비중 상태에서 "완료로 변경" 클릭
- Then 완료 상태로 변경됨
- And **Negative**: 완료 → 준비중 / 준비중 → 대기중 등 **역방향 전환은 거부 (400)**

## US-A-06 직권 주문 삭제
**As a** Manager Mia,  
**I want** 어떤 상태의 주문이든 직권으로 삭제할 수 있고,  
**so that** 잘못 들어온 주문을 즉시 정리할 수 있다.  
**Priority**: Must

**Acceptance Criteria**
- Given 주문 상세 뷰 (상태 불문)
- When "삭제" 버튼 클릭 → 확인 팝업에서 확인
- Then 주문이 즉시 삭제되고 테이블 총액이 재계산되어 대시보드에 반영
- And **Negative**: 확인 팝업에서 취소 시 아무 변화 없음, 이미 삭제된 주문 재삭제 시 404 에러 표시

---

# Epic 8 — Admin: Table & Session Management

## US-A-07 테이블 이용 완료 (세션 종료)
**As a** Manager Mia,  
**I want** 테이블의 이용 완료 처리를 하고,  
**so that** 새 고객이 깨끗한 상태로 주문을 시작할 수 있다.  
**Priority**: Must

**Acceptance Criteria**
- Given 테이블 카드에서 현재 활성 세션이 있는 상태
- When "이용 완료" 버튼 클릭 → 확인
- Then 해당 세션의 주문들이 OrderHistory로 이동 (완료 시각 기록)
- And 테이블 카드의 현재 주문/총액이 0으로 리셋
- And 새 고객이 주문 시 새로운 세션이 자동 시작됨 (US-C-08)
- And **Negative**: 활성 세션이 없는 테이블에서는 "이용 완료" 버튼 비활성

## US-A-08 테이블별 과거 주문 이력 조회
**As a** Manager Mia,  
**I want** 특정 테이블의 과거 주문 내역을 조회하고,  
**so that** 고객 문의나 정산 시 참고할 수 있다.  
**Priority**: Should

**Acceptance Criteria**
- Given 대시보드 또는 테이블 카드
- When "과거 내역" 버튼 클릭
- Then 해당 테이블의 OrderHistory가 시간 역순으로 표시됨 (번호/시각/메뉴/총액/완료시각)
- And 날짜 필터를 적용하면 기간 내 주문만 표시
- And "닫기" 버튼으로 대시보드로 복귀
- And **Negative**: 이력이 없으면 "과거 내역 없음" 메시지

## US-A-09 테이블 태블릿 재설정
**As a** Manager Mia,  
**I want** 분실/변경된 테이블 태블릿 설정을 재설정하고,  
**so that** 태블릿 교체나 비밀번호 변경에 대응할 수 있다.  
**Priority**: Could

**Acceptance Criteria**
- Given 관리자가 관리자 UI에서 테이블을 선택
- When "테이블 재설정" 버튼 클릭 → 새 비밀번호 입력 저장
- Then 기존 태블릿의 자동 로그인이 무효화되고 재설정 필요 상태가 됨
- And **Negative**: 활성 세션이 있는 테이블은 이용 완료 후에만 재설정 가능 (경고 메시지)

---

# Epic 9 — Admin: Menu Management

## US-A-10 메뉴 등록
**As a** Manager Mia,  
**I want** 새 메뉴를 등록하고,  
**so that** 고객 화면에 노출할 수 있다.  
**Priority**: Must

**Acceptance Criteria**
- Given 메뉴 관리 화면
- When 필수 필드(메뉴명, 가격, 카테고리)를 입력하고 저장
- Then 메뉴가 DB에 생성되고 카테고리 하위 목록에 추가됨
- And 고객 메뉴 화면에도 즉시(또는 다음 조회 시) 반영
- And **Negative**: 필수 필드 누락 / 가격이 음수 또는 비정상적으로 큰 값 → 검증 에러

## US-A-11 메뉴 수정/삭제
**As a** Manager Mia,  
**I want** 기존 메뉴를 수정하거나 삭제하고,  
**so that** 가격 변경/단종 메뉴를 관리할 수 있다.  
**Priority**: Must

**Acceptance Criteria**
- Given 메뉴 관리 화면의 기존 메뉴
- When 수정 버튼 → 필드 변경 후 저장
- Then 변경 사항이 저장되고 목록에 반영됨
- When 삭제 버튼 → 확인 팝업 → 삭제
- Then 메뉴가 삭제됨 (Soft delete 권장)
- And **Negative**: 주문에 참조된 메뉴를 hard delete 시도할 경우 에러, soft delete로 대체

## US-A-12 메뉴 노출 순서 조정
**As a** Manager Mia,  
**I want** 카테고리 내 메뉴의 노출 순서를 바꾸고,  
**so that** 추천 메뉴를 상단에 배치할 수 있다.  
**Priority**: Could

**Acceptance Criteria**
- Given 메뉴 관리 화면의 메뉴 리스트
- When 순서 변경 UI(드래그앤드롭 또는 순서 입력)로 순서를 조정하고 저장
- Then 고객 메뉴 화면에 조정된 순서로 표시됨
- And **Negative**: 중복된 순서 값 입력 시 서버에서 자동 재배치 또는 검증 에러

---

# Summary

| Epic | # of Stories | Must | Should | Could |
|---|---|---|---|---|
| 1. Customer — Session & Login | 2 | 2 | 0 | 0 |
| 2. Customer — Menu Browsing | 2 | 2 | 0 | 0 |
| 3. Customer — Cart | 2 | 2 | 0 | 0 |
| 4. Customer — Order Placement | 2 | 2 | 0 | 0 |
| 5. Customer — Order History | 2 | 1 | 1 | 0 |
| 6. Admin — Auth | 2 | 2 | 0 | 0 |
| 7. Admin — Monitoring | 4 | 4 | 0 | 0 |
| 8. Admin — Table & Session | 3 | 1 | 1 | 1 |
| 9. Admin — Menu Management | 3 | 2 | 0 | 1 |
| **Total** | **22** | **18** | **2** | **2** |

## INVEST 준수 체크
- **Independent**: 각 스토리는 독립적으로 구현/테스트 가능 (일부 세션 관련 스토리는 US-C-08과 연계)
- **Negotiable**: 수락 기준은 조정 가능
- **Valuable**: 모두 고객 또는 관리자에게 가치 제공
- **Estimable**: 각 1~2일 규모로 추산 가능
- **Small**: Small granularity 유지
- **Testable**: Given/When/Then AC로 테스트 가능
