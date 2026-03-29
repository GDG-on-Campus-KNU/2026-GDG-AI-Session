[요구사항.md](https://github.com/user-attachments/files/26328860/default.md)

# 1. API 목록 및 엔드 포인트

```markdown
POST   /auth/signup
POST   /auth/login
GET    /users/me

GET    /items
GET    /items/{itemId}
GET    /items/{itemId}/availability

POST   /reservations
GET    /reservations/me
PATCH  /reservations/{reservationId}/cancel

GET    /admin/reservations
PATCH  /admin/reservations/{reservationId}/approve
PATCH  /admin/reservations/{reservationId}/reject
PATCH  /admin/rentals/{rentalId}/pickup
PATCH  /admin/rentals/{rentalId}/return
```

# KNU-Share API 목록

| 구분 | API 엔드포인트 | 기능 |
| --- | --- | --- |
| Auth | POST /auth/email/send | 학교 이메일 인증 코드 발송 |
| Auth | POST /auth/email/verify | 이메일 인증 코드 검증 |
| Auth | POST /auth/signup | 회원가입 |
| Auth | POST /auth/login | 로그인 및 토큰 발급 |
| User | GET /users/me | 내 사용자 정보 조회 |
| User | PATCH /users/me | 사용자 정보 수정 |
| Item | GET /items | 물품 목록 조회 |
| Item | GET /items/{itemId} | 물품 상세 조회 |
| Item | GET /items/{itemId}/availability | 특정 기간 예약 가능 여부 조회 |
| Reservation | POST /reservations | 물품 예약 신청 |
| Reservation | GET /reservations/me | 내 예약 목록 조회 |
| Reservation | GET /reservations/{reservationId} | 예약 상세 조회 |
| Reservation | PATCH /reservations/{reservationId}/cancel | 예약 취소 |
| Rental | GET /rentals/me/current | 현재 대여 중인 물품 조회 |
| Rental | GET /rentals/me/history | 과거 대여 이력 조회 |
| Admin Item | POST /admin/items | 물품 등록 |
| Admin Item | PATCH /admin/items/{itemId} | 물품 정보 수정 |
| Admin Item | PATCH /admin/items/{itemId}/status | 물품 대여 가능 상태 변경 |
| Admin Reservation | GET /admin/reservations | 예약 신청 목록 조회 |
| Admin Reservation | PATCH /admin/reservations/{reservationId}/approve | 예약 승인 |
| Admin Reservation | PATCH /admin/reservations/{reservationId}/reject | 예약 거절 |
| Admin Rental | PATCH /admin/rentals/{rentalId}/pickup | 물품 수령 처리 (대여 시작) |
| Admin Rental | PATCH /admin/rentals/{rentalId}/return | 반납 완료 처리 |
| Admin Rental | GET /admin/rentals/overdue | 연체 목록 조회 |
| Notification | GET /notifications | 알림 목록 조회 |
| Notification | PATCH /notifications/{notificationId}/read | 알림 읽음 처리 |

# KNU-Share MVP API 목록

| API 엔드포인트 | 기능 |
| --- | --- |
| POST /auth/signup | 회원가입 |
| POST /auth/login | 로그인 |
| GET /items | 물품 목록 조회 |
| GET /items/{itemId} | 물품 상세 조회 |
| POST /reservations | 예약 신청 |
| GET /reservations/me | 내 예약 조회 |
| PATCH /reservations/{reservationId}/cancel | 예약 취소 |
| GET /admin/reservations | 예약 목록 조회 |
| PATCH /admin/reservations/{reservationId}/approve | 예약 승인 |
| PATCH /admin/rentals/{rentalId}/return | 반납 처리 |

# 2. DB 스키마

```jsx
CREATE TABLE users (
		id              BIGINT PRIMARY KEY AUTO_INCREMENT,
		email           VARCHAR(100) NOT NULL UNIQUE,
		password        VARCHAR(255) NOT NULL,
		name            VARCHAR(50)  NOT NULL,
		student_id      VARCHAR(20)  NOT NULL UNIQUE,
		department      VARCHAR(100) NOT NULL,
		role            ENUM('USER', 'ADMIN') NOT NULL DEFAULT 'USER',
		penalty_ends_at DATETIME,
		created_at      DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

```jsx
CREATE TABLE items (
	id              BIGINT PRIMARY KEY AUTO_INCREMENT,
	name            VARCHAR(100) NOT NULL,
	category        ENUM('DIGITAL', 'EVENT', 'BOOK', 'ETC') NOT NULL,
	description     TEXT,
	total_quantity  INT          NOT NULL,
	max_rental_days INT          NOT NULL,
	status          ENUM('AVAILABLE', 'UNAVAILABLE') NOT NULL DEFAULT 'AVAILABLE',
	created_at      DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

```jsx
CREATE TABLE reservations (
	id          BIGINT PRIMARY KEY AUTO_INCREMENT,
	user_id     BIGINT   NOT NULL,
	item_id     BIGINT   NOT NULL,
	status      ENUM('RESERVED', 'RENTED', 'RETURNED', 'OVERDUE', 'CANCELLED') NOT NULL,
	start_date  DATE     NOT NULL,
	end_date    DATE     NOT NULL,
	returned_at DATETIME,
	condition   ENUM('NORMAL', 'DAMAGED'),
	created_at  DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
	FOREIGN KEY (user_id) REFERENCES users(id),
	FOREIGN KEY (item_id) REFERENCES items(id),
	INDEX idx_item_status (item_id, status),
	INDEX idx_user_status (user_id, status)
)
```

```jsx
CREATE TABLE email_verifications (
	id         BIGINT PRIMARY KEY AUTO_INCREMENT,
	email      VARCHAR(100) NOT NULL
	token      VARCHAR(255) NOT NULL UNIQUE,
	expires_at DATETIME     NOT NULL,
	verified   BOOLEAN      NOT NULL DEFAULT FALSE
);
```

```

```

# 3. 목표

```markdown
# BE 세부 목표

- 학교 이메일 인증 기반 회원가입 및 로그인 API 구현
- 물품 조회 및 예약 가능 여부 확인 API 구현
- 예약 생성, 조회, 취소 API 구현
- 관리자 승인/거절 및 반납 처리 API 구현
- 사용자, 물품, 예약 중심의 DB 스키마 설계
- 동시 예약 상황에서 데이터 정합성 보장
- 상태값 기반 예약/대여 프로세스 관리
- 향후 알림 및 연체 관리 기능 확장을 고려한 구조 설계
```

```markdown
# BE 파트 목표

백엔드 파트의 목표는 KNU-Share 서비스의 핵심 기능인 회원 인증, 물품 조회, 예약 신청, 관리자 승인, 반납 처리 과정을 안정적으로 지원할 수 있는 서버 시스템을 구축하는 것이다.  
이를 위해 사용자, 물품, 예약 데이터를 체계적으로 관리할 수 있는 데이터베이스 구조를 설계하고, 각 기능이 명확한 API 엔드포인트를 통해 동작하도록 구현한다.

## 1. 인증 및 사용자 관리
- 학교 이메일 기반 회원가입 및 로그인 기능을 제공한다.
- 사용자 정보를 안전하게 저장하고, 일반 사용자와 관리자 권한을 구분하여 접근을 제어한다.
- 인증된 사용자만 예약, 조회, 관리자 기능 등을 수행할 수 있도록 한다.

## 2. 물품 조회 및 예약 프로세스 지원
- 물품 목록, 상세 정보, 예약 가능 여부를 조회할 수 있는 API를 제공한다.
- 사용자가 원하는 기간에 대해 예약 신청을 할 수 있도록 한다.
- 예약 상태를 저장하고, 사용자가 자신의 예약 내역을 조회하거나 승인 전 예약을 취소할 수 있도록 한다.

## 3. 관리자 업무 처리 지원
- 관리자가 예약 요청 목록을 조회하고 승인 또는 거절할 수 있도록 한다.
- 물품 수령 시 대여 상태로 전환하고, 반납 시 반납 완료 처리할 수 있도록 한다.
- 물품 등록, 수정, 상태 변경 등 관리자 중심의 관리 기능을 지원한다.

## 4. 데이터 정합성 보장
- 동일 물품에 대해 동시에 여러 예약 요청이 들어와도 재고 수량과 예약 상태가 일관되게 유지되도록 한다.
- 예약, 대여, 반납 상태 변경 과정에서 비정상적인 흐름이 발생하지 않도록 상태 전이 규칙을 관리한다.
- 사용자, 물품, 예약 간의 관계를 데이터베이스에서 명확하게 연결하여 데이터 무결성을 보장한다.

## 5. 이력 및 운영 관리 기반 마련
- 현재 대여 중인 물품과 과거 대여 이력을 조회할 수 있는 구조를 제공한다.
- 연체 상태를 식별하고 관리할 수 있도록 데이터 모델을 설계한다.
- 향후 알림, 페널티, 대여 통계 기능으로 확장할 수 있도록 기본 구조를 마련한다.

## 6. 백엔드 구현 방향
- REST API 기반으로 클라이언트와 통신할 수 있는 구조를 설계한다.
- 사용자(users), 물품(items), 예약(reservations), 이메일 인증(email_verifications) 테이블을 중심으로 핵심 도메인을 관리한다.
- API와 DB 스키마가 일관되게 연결되도록 설계하여 유지보수성과 확장성을 확보한다.

## 7. 최종 목표
백엔드는 단순히 데이터를 저장하는 역할에 그치지 않고,  
KNU-Share 서비스의 핵심 흐름인 **조회 → 예약 → 승인 → 대여 → 반납** 전 과정을 안정적으로 처리하는 서버 기반을 구축하는 것을 최종 목표로 한다.
```

# 5. RPS 목표

대학 내 서비스 특성상 동시 사용자가 제한적이다. 아래 가정을 기준으로 산정한다.
```
가정
- 활성 사용자: 약 500명 (학과 단위 운영 기준)
- 피크 타임: 개강 직후, 축제 기간 (하루 2~3시간 집중)
- 피크 시간대 요청 집중률: 전체 일 요청의 40%가 1시간에 몰린다고 가정

일 요청 추정
- 물품 조회:    500명 × 3회  = 1,500 req/day
- 예약 신청:    500명 × 0.5회 = 250  req/day
- 관리자 처리:  소수, 무시

피크 RPS 계산
- 조회: 1,500 × 0.4 / 3,600 ≒ 0.17 RPS
- 예약: 250  × 0.4 / 3,600 ≒ 0.03 RPS

목표 RPS (안전 마진 10배 적용)
- 조회 API: 목표 2 RPS, 최대 허용 응답 시간 500ms
- 예약 API: 목표 1 RPS, 최대 허용 응답 시간 1,000ms
- 전체:     목표 5 RPS (여유 포함)
```

# 6. 아키텍처
```
[Client: Web / Android]
        │
        ▼
  [Nginx] ── HTTPS / Reverse Proxy
        │
        ▼
[Spring Boot Application]
  ┌─────────────────────────┐
  │  Controller             │
  │  Service                │
  │  ├─ 예약 시 Pessimistic │
  │  │  Lock (item_id 기준) │
  │  └─ Scheduler (1h)      │
  │     연체 → OVERDUE 전환  │
  │  Repository (JPA)       │
  └─────────────────────────┘
        │              │
        ▼              ▼
    [MySQL]         [Redis]
   - 영구 데이터     - JWT Refresh Token 저장
   - 상태 관리       - (선택) 이메일 인증 토큰 TTL 관리
```
