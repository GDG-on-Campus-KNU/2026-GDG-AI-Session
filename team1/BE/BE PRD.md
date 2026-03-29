
# BE

### 1. 백엔드 관점의 기능 우선순위 (Priority)

백엔드는 프론트엔드/안드로이드 개발자가 API를 연결할 수 있도록 핵심 도메인을 먼저 뚫어주는 것이 중요합니다.

- **P0 (블로커 방지 및 핵심 파이프라인):** * DB 모델링 (User, Item, Reservation, Penalty 엔티티 설계)
    - 인증/인가 로직 (학교 이메일 인증, JWT 발급)
    - 물품 CRUD API 및 상태 머신(State Machine)의 뼈대 구현
- **P1 (서비스 신뢰도 및 관리자 기능):**
    - 재고 차감 시 동시성 제어 (Redis 분산 락 또는 DB 락)
    - 연체 및 페널티 부여 자동화 (배치 스케줄러)
    - 관리자용 대여/반납 승인 API (QR/바코드 페이로드 검증)
- **P2 (사용성 및 부가 기능):**
    - 검색 최적화 및 카테고리 필터링
    - 푸시 알림 발송 트리거 (FCM 연동)
        - 과거 대여 이력 조회 API

---

### 2. 4주 스프린트 백엔드 개발 목표 및 구현 명세

### **Sprint 1: 기반 인프라 구축 및 인증 도메인 (Foundation & Auth)**

- **목표:** 서버 환경 세팅을 완료하고, 클라이언트가 로그인/가입 API를 연동할 수 있도록 제공합니다.
- **주요 구현 사항:**
    - **프로젝트 초기화:** Spring Boot 프로젝트 세팅, GitHub Repository 구축, DB(MySQL/PostgreSQL) 연결.
    - **DB 스키마 설계:** JPA/Hibernate를 활용한 엔티티(Entity) 설계 및 연관관계 매핑 (N:1, 1:N).
    - **인증/인가(Auth):** 학교 메일 발송 및 인증 번호 검증 API 구현, Spring Security와 JWT를 활용한 로그인 유지 및 권한(일반 유저 vs 관리자) 분리.
    - **API 문서화:** Swagger(Springdoc) 또는 Spring RestDocs 초기 세팅.

### **Sprint 2: 핵심 비즈니스 로직 및 API (Core Domain)**

- **목표:** 서비스의 알파이자 오메가인 '물품'과 '예약' 도메인의 생명주기를 구현합니다.
- **주요 구현 사항:**
    - **물품(Item) 도메인:** 카테고리별 물품 조회, 상세 조회, 다건 등록(Bulk Insert) API 구현.
    - **예약(Reservation) 도메인:** 대여 시작/종료일에 따른 예약 가능 여부 검증 로직 구현.
    - **상태 머신(State Machine):** 물품의 상태(`AVAILABLE` -> `RESERVED` -> `RENTED` -> `RETURNED`/`MAINTENANCE`) 전이를 제어하는 서비스 계층 로직 작성.

### **Sprint 3: 동시성 제어 및 관리자 스케줄러 (Advanced Logic)**

- **목표:** 예약 폭주 시 데이터 정합성을 보장하고, 관리자의 수동 작업을 자동화합니다.
- **주요 구현 사항:**
    - **동시성 처리 (Concurrency):** 인기 물품 대여 시 초과 예약이 발생하지 않도록 비관적 락(Pessimistic Lock)이나 Redis(Redisson) 분산 락을 적용하여 동시성 이슈 해결.
    - **배치 작업 (Batch Job):** `@Scheduled` 또는 Spring Batch를 활용해 매일 자정 연체자를 집계하고 상태를 `OVERDUE`로 변경 및 페널티(7일 정지) 부여 로직 구현.
    - **QR 검증 API:** 현장 스캔 시 예약 정보의 위변조를 막기 위해 QR코드에 담긴 암호화 페이로드를 검증하고 즉시 상태를 업데이트하는 API 개발.

### **Sprint 4: 외부 연동, 테스트 및 안정화 (Integration & QA)**

- **목표:** 푸시 알림을 연동하고, 실제 운영 환경과 유사한 부하 테스트를 통해 버그를 잡습니다.
- **주요 구현 사항:**
    - **알림 트리거 발송:** 반납 1시간 전, 예약 취소, 연체 발생 등의 이벤트 발생 시 Firebase Cloud Messaging(FCM)을 통해 앱으로 푸시 알림 발송 API 구현.
    - **테스트 및 리팩토링:** 도메인 핵심 로직에 대한 단위 테스트(JUnit) 작성. JMeter 등을 활용해 동시성 처리 로직의 부하 테스트 및 병목 구간 쿼리 튜닝.
    - **CI/CD 파이프라인:** GitHub Actions 등을 통한 자동 배포 환경 구축.

## 3. 시스템 아키텍처 및 성능 목표

- **트래픽 목표 (Target RPS):** * 평시: `~50 RPS`
    - 피크 타임 (인기 물품 예약 오픈 시): `~500 RPS`
- **Tech Stack:**
    - **Backend:** `Java 17+`, `Spring Boot 3.x`
    - **Database:** `MySQL 8.0` (상태 머신 영속성 보장)
    - **Cache & Lock:** `Redis` (조회 캐싱 및 Redisson 분산 락)
    - **Infra:** `AWS EC2` (또는 교내 서버), `Nginx`, `Github Actions` (CI/CD)
    

## **🗄️ 4. 백엔드 데이터베이스 설계 (ERD 초안)**

| **테이블명 (Table)** | **주요 컬럼 (Fields)** | **역할 및 설명** |
| --- | --- | --- |
| **`users`** | `id` (PK), `email`, `student_id`, `role`, `penalty_end_date` | 유저 정보 및 권한(USER/ADMIN), 페널티 관리 |
| **`items`** | `id` (PK), `category_id`, `name`, `description` | 물품 메타데이터 (예: '보조배터리 10000mAh') |
| **`item_units`** | `id` (PK), `item_id` (FK), `sku_code`, `status` | ⭐️ 실제 대여되는 개별 기기 (QR 매핑, 상태 관리) |
| **`reservations`** | `id` (PK), `user_id`, `item_unit_id`, `start_time`, `status` | 예약/대여 이력, **상태 머신 전이의 핵심** |
| **`categories`** | `id` (PK), `name` | 카테고리 (디지털/IT, 행사/레저 등) |

## 5. 주요 API 명세서 (API Endpoints)

**[🔒 Auth & User]**

- `POST` `/api/v1/auth/login` : 학교 이메일 로그인 (JWT 발급)
- `POST` `/api/v1/auth/register` : 회원가입 (초기엔 무조건 `ROLE_USER` 가입, 관리자는 수동 승격)
- `GET` `/api/v1/users/me` : 내 프로필 및 페널티 조회

**[🔍 Discovery (물품 조회)]**

- `GET` `/api/v1/items` : 전체 물품 리스트 조회 (카테고리/검색/페이지네이션)
- `GET` `/api/v1/items/{itemId}` : 상세 조회 및 **실시간 대여 가능 수량** 확인

**[📅 Reservation (예약)]**

- `POST` `/api/v1/reservations` : 대여 예약 신청 (**🚨 Redis Lock 동시성 제어 구간**)
- `GET` `/api/v1/reservations/me` : 나의 대여/예약 진행 목록
- `PATCH` `/api/v1/reservations/{reservationId}/cancel` : 사용자 직접 예약 취소

**[🛠️ Admin (관리자 전용)]**

- `POST` `/api/v1/admin/items` : 신규 물품 다건(Bulk) 등록
- `PATCH` `/api/v1/admin/item-units/{unitId}/status` : 개별 기기 상태 변경 (수리 중, 분실 등)
- `PATCH` `/api/v1/admin/reservations/{reservationId}/approve` : QR 스캔 후 대여 승인 (`RESERVED` → `RENTED`)
- `PATCH` `/api/v1/admin/reservations/{reservationId}/return` : QR 스캔 후 반납 처리 (`RENTED` → `RETURNED`)
