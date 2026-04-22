# 요구사항 확인 질문

테이블오더 서비스 요구사항 문서를 분석한 결과, 구현에 필요한 몇 가지 기술적 선택 사항과 세부 사항을 명확히 하기 위한 질문입니다.

각 [Answer]: 태그 뒤에 선택한 옵션 문자를 입력해 주세요. 적절한 옵션이 없으면 마지막 "Other" 옵션을 선택하고 [Answer]: 태그 뒤에 설명을 작성해 주세요.

모든 질문에 답변 완료 후 "완료" 또는 "done"이라고 알려주세요.

---

## Question 1: 배포 환경 (Deployment Target)
이 서비스를 어디에 배포할 계획인가요?

A) AWS 클라우드 (ECS/EKS/Lambda + RDS/DynamoDB)
B) 온프레미스 서버 (자체 서버에서 Docker로 구동)
C) 단순 로컬/개발 환경 중심 (추후 배포 결정)
D) 기타 클라우드 (Azure, GCP)
X) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Question 2: Backend 기술 스택
어떤 Backend 기술 스택을 선호하시나요?

A) Node.js + TypeScript (Express/NestJS)
B) Python (FastAPI/Django)
C) Java/Kotlin (Spring Boot)
D) Go
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 3: Frontend 기술 스택
Frontend (고객용 + 관리자용 웹 UI)는 어떤 기술을 사용하나요?

A) React + TypeScript
B) Vue.js + TypeScript
C) Next.js (React 기반 full-stack)
D) Vanilla JS/HTML/CSS (최소 의존성)
X) Other (please describe after [Answer]: tag below)

[Answer]:  A

---

## Question 4: 데이터베이스
어떤 데이터베이스를 사용하나요?

A) PostgreSQL (관계형, 기능 풍부)
B) MySQL (관계형, 널리 사용)
C) DynamoDB (AWS 네이티브 NoSQL)
D) SQLite (경량, 개발/PoC용)
X) Other (please describe after [Answer]: tag below)

[Answer]: D

---

## Question 5: 예상 동시 접속 규모 (MVP 기준)
MVP 단계에서 예상되는 동시 사용 규모는 어느 정도인가요? (성능/인프라 설계에 영향)

A) 소규모 (매장 1~10개, 동시 테이블 50개 이내)
B) 중규모 (매장 10~100개, 동시 테이블 500개 이내)
C) 대규모 (100개 이상 매장, 1000+ 테이블)
D) 워크샵/데모 수준 (최소 동시 사용자 테스트만)
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 6: 메뉴 이미지 저장소
요구사항에 "이미지 URL" 필드가 있습니다. 이미지는 어떻게 관리하나요?

A) AWS S3에 업로드하여 URL 관리 (이미지 업로드 기능 포함)
B) 외부 URL만 저장 (관리자가 외부 호스팅 이미지 URL을 직접 입력)
C) 로컬 파일 시스템에 저장
D) 이미지 기능은 MVP에서 제외 (placeholder만 사용)
X) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## Question 7: 초기 데이터 (시드 데이터)
매장/관리자 계정/메뉴 등 초기 데이터를 어떻게 구성하나요?

A) 개발 시 DB 시드 스크립트로 예시 매장 + 메뉴 자동 생성
B) 관리자 회원가입 API/UI 제공 (요구사항에는 명시 안 됨 - 추가 기능)
C) 관리자 수동 등록 (수동 SQL/DB 조작)
D) 슈퍼 관리자 계정만 시드, 매장/메뉴는 관리자가 직접 등록
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 8: 테이블 비밀번호 변경 및 세션 보안
고객용 테이블 태블릿의 "테이블 비밀번호"는 어떻게 운영하나요?

A) 매장/테이블별 고정 (관리자가 초기 설정, 변경 시 관리자가 재설정)
B) 테이블 세션 종료 시 자동 회전 (새 고객 대비 보안 강화)
C) 단순 PIN 방식, 변경 기능 없음 (PoC 수준)
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 9: 주문 상태 머신
주문 상태(대기중/준비중/완료) 전환 규칙을 명확히 하고 싶습니다.

A) 엄격한 순서 (대기중 → 준비중 → 완료만 가능, 역방향 불가)
B) 유연 (관리자가 자유롭게 모든 상태로 변경 가능, 삭제 포함)
C) 엄격한 순서 + 삭제(직권 수정)는 모든 상태에서 가능
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 10: 테스팅 수준
테스트 커버리지는 어느 수준까지 요구되나요?

A) 핵심 비즈니스 로직 단위 테스트 (기본)
B) 단위 테스트 + API 통합 테스트
C) 단위 + 통합 + E2E 테스트 (전체)
D) 최소한의 스모크 테스트만 (PoC/워크샵)
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## Question 11: Security Extensions
Should security extension rules be enforced for this project?

A) Yes — enforce all SECURITY rules as blocking constraints (recommended for production-grade applications)
B) No — skip all SECURITY rules (suitable for PoCs, prototypes, and experimental projects)
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## Question 12: Property-Based Testing Extension
Should property-based testing (PBT) rules be enforced for this project?

A) Yes — enforce all PBT rules as blocking constraints (recommended for projects with business logic, data transformations, serialization, or stateful components)
B) Partial — enforce PBT rules only for pure functions and serialization round-trips (suitable for projects with limited algorithmic complexity)
C) No — skip all PBT rules (suitable for simple CRUD applications, UI-only projects, or thin integration layers with no significant business logic)
X) Other (please describe after [Answer]: tag below)

[Answer]: C

---
