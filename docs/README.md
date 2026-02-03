# LostPing - 분실물 실시간 알림 서비스

## 서비스 소개

LostPing은 LOST112 공공 API를 활용하여 분실물 등록 시 실시간 알림을 제공하는 모니터링 서비스입니다. 사용자가 등록한 키워드에 맞는 분실물이 경찰서나 지하철역 등에 등록되면 즉시 알림을 받을 수 있습니다.

## 핵심 가치 제안

### 문제 정의
- 분실물을 찾기 위해 매일 LOST112 사이트를 방문하여 수동으로 확인해야 하는 불편함
- 새로운 분실물 등록 시 즉각적인 인지 불가
- 여러 키워드를 동시에 모니터링하기 어려움

### 해결 방법
- **자동 모니터링**: 주기적으로 LOST112 API를 폴링하여 새로운 분실물 자동 확인
- **실시간 알림**: 키워드 매칭 시 이메일, 카카오톡, SMS로 즉시 알림 발송
- **멀티 키워드**: 여러 키워드를 동시에 등록하여 다양한 물건 추적 가능
- **상세 정보 제공**: 분실물 상세 정보, 보관 장소, 연락처 등 즉시 확인

## 주요 기능

### 1. 키워드 관리
- 사용자별 키워드 등록/수정/삭제
- 키워드별 알림 채널 설정 (이메일, 카카오톡, SMS)
- 키워드 활성화/비활성화 제어
- 예시 키워드: "갤럭시 버즈", "검정 지갑", "AirPods", "파란색 우산"

### 2. 분실물 모니터링
- LOST112 API 주기적 폴링 (10분 간격)
- 신규 등록 분실물 자동 감지
- 키워드 매칭 알고리즘 (부분 일치, 완전 일치)
- 중복 알림 방지 (이미 알림 발송된 항목 필터링)

### 3. 알림 발송
- **이메일**: 상세 정보 포함한 알림 발송
- **카카오톡 알림톡**: 간편한 확인 및 링크 제공
- **SMS**: 긴급 알림 (선택적)
- 알림 발송 이력 관리

### 4. 분실물 정보 조회
- 사용자별 매칭된 분실물 목록
- 분실물 상세 정보 (이름, 습득 장소, 보관 장소, 습득 날짜)
- 분실물 이미지 (가능한 경우)
- 보관 기관 연락처

### 5. 사용자 관리
- 회원 가입/로그인
- 프로필 관리
- 알림 설정 관리
- 구독 이력 조회

## 기술 스택

### Backend
- **Framework**: Spring Boot 3.5.7
- **Language**: Java 21
- **Build Tool**: Gradle (Multi-module)
- **ORM**: Spring Data JPA / Hibernate
- **Database**: MySQL 8.0
- **Cache/Queue**: Redis 7.0 (분산 락, 캐싱, Redis Streams)
- **Batch**: Spring Batch (LOST112 API 폴링, 알림 발송)

### Infrastructure
- **Container**: Docker & Docker Compose
- **Load Balancer**: Nginx
- **Monitoring**: Spring Boot Actuator
- **API Documentation**: Swagger/OpenAPI 3.0

### External APIs
- **LOST112 공공 API**: 분실물 정보 조회
- **카카오톡 알림톡 API**: 카카오톡 알림 발송
- **이메일 SMTP**: 이메일 알림 발송

## 프로젝트 구조

```
lostping/
├── common/                       # 공통 모듈
│   └── src/main/java/com/phm/lostping/
│       ├── domain/               # 도메인 엔티티
│       │   ├── user/             # User, UserRole
│       │   ├── keyword/          # Keyword, NotificationChannel
│       │   ├── lostitem/         # LostItem, ItemStatus
│       │   └── notification/     # Notification, NotificationHistory
│       ├── infrastructure/       # 인프라 구현
│       │   ├── repository/       # JPA 리포지토리
│       │   ├── external/         # 외부 API 클라이언트
│       │   │   ├── lost112/      # LOST112 API 클라이언트
│       │   │   ├── kakao/        # 카카오톡 API 클라이언트
│       │   │   └── email/        # 이메일 서비스
│       │   └── cache/            # Redis 캐시
│       └── config/               # 공통 설정
├── api-server/                   # REST API 서버
│   └── src/main/java/com/phm/lostping/
│       ├── presentation/         # 컨트롤러 및 DTO
│       ├── application/          # UseCase 및 서비스
│       └── config/               # API 서버 설정
└── batch-server/                 # 배치 작업 서버
    └── src/main/java/com/phm/lostping/batch/
        └── job/                  # 배치 작업
            ├── Lost112PollingJob # LOST112 API 폴링
            ├── KeywordMatchingJob # 키워드 매칭
            └── NotificationJob   # 알림 발송
```

## 시작하기

### 사전 요구사항
- Docker & Docker Compose
- Java 21
- Gradle 8.x

### 로컬 환경 실행

```bash
# 1. 전체 서비스 시작 (인프라 + 애플리케이션)
docker-compose up -d --build

# 2. 데이터베이스 스키마 초기화
docker exec -i lostping-mysql mysql -uroot -proot lostping < docs/schema.sql

# 3. Spring Batch 메타데이터 테이블 생성
docker exec -i lostping-mysql mysql -uroot -proot lostping < docs/spring-batch-schema.sql

# 4. API 서버 접속
curl http://localhost:8085/actuator/health

# 5. Swagger UI 접속
open http://localhost:8085/swagger-ui.html
```

### 빌드 및 테스트

```bash
# 전체 빌드
./gradlew build

# 테스트 실행
./gradlew test

# 특정 모듈 빌드
./gradlew :api-server:build
./gradlew :batch-server:build
```

## 문서 목록

- [비즈니스 요구사항](requirements.md) - 상세 기능 요구사항 및 사용자 시나리오
- [시스템 아키텍처](architecture.md) - 전체 시스템 구조 및 모듈 설계
- [API 명세](api-specification.md) - REST API 엔드포인트 및 스키마
- [데이터 모델](data-model.md) - ERD 및 테이블 정의
- [외부 연동 가이드](integration-guide.md) - LOST112, 카카오톡, 이메일 연동
- [배치 작업 설계](batch-jobs.md) - 배치 작업 스케줄 및 처리 로직
- [구현 로드맵](roadmap.md) - 단계별 개발 계획

## 라이선스

이 프로젝트는 개인 프로젝트이며, LOST112 공공 API 이용 약관을 준수합니다.

## 기여

이슈 및 Pull Request는 언제든지 환영합니다.

## 연락처

- 개발자: Park Hyungmin
- 이메일: (프로젝트 담당자 이메일)
