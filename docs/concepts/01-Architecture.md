# 01. Architecture (구성/흐름)

## 목표
newBoard는 “CRUD 게시판”을 실서비스에 가까운 형태로 확장하기 위해 아래 요소를 통합했습니다.

- **인증/인가**: Spring Security + JWT
- **메시징/이벤트**: Kafka (Producer/Consumer)
- **캐시**: Redis
- **문서화/테스트**: Swagger UI (springdoc-openapi)
- **실행/배포 표준화**: Docker Compose + EC2 배포

## 구성 요소
- **Spring Boot API (8080 컨테이너 / 8081 호스트 매핑)**
  - REST API 제공
  - JWT 인증 필터로 인증 처리
  - 게시글 이벤트 Kafka 발행
  - Kafka Consumer로 로그 적재
  - Redis 캐시 저장/조회
- **MySQL 8**
  - User / Article / ArticleLog 등 영속화
- **Kafka + Zookeeper**
  - `article-topic` 토픽으로 메시지 발행/소비
- **Redis**
  - 최신 게시글 제목 등 빠른 조회용 캐시

## 요청 흐름 예시

### A) 회원가입/로그인
1. `POST /api/auth/signup` → 비밀번호 BCrypt 해싱 → DB 저장
2. `POST /api/auth/login` → 비밀번호 검증 → JWT 발급(HS256)

### B) 게시글 생성(인증 필요)
1. 클라이언트가 `Authorization: Bearer <token>` 포함해 `POST /api/articles`
2. `JwtAuthenticationFilter`가 토큰 검증 후 SecurityContext에 인증 설정
3. Article 저장(MySQL)
4. Kafka Producer가 `article-topic`에 이벤트 메시지 발행
5. Kafka Consumer가 메시지 소비 → ArticleLog 저장(MySQL)
6. (옵션) Redis에 “최근 게시글 제목” 캐시 저장

### C) 조회/수정/삭제
- 조회: 일반 공개 또는 인증 필요 정책에 따라 설정
- 수정/삭제: **작성자만 가능**(권한 체크)

## 운영 관점 체크포인트
- Swagger UI: `/swagger-ui/index.html`
- OpenAPI JSON: `/v3/api-docs`
- Kafka 이벤트 확인: `kafka-console-consumer`
- Redis 키 확인: `redis-cli GET lastArticle`
