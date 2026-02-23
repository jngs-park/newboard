# NewBoard

Spring Boot + MySQL + Kafka + Redis + JWT 기반 게시판 API

- 게시글 CRUD
- Kafka Producer/Consumer (topic: `article-topic`)로 DB 로그 적재
- Redis 캐시 (마지막 게시글 제목 등)
- JWT 인증/인가
- Swagger(OpenAPI) 문서 제공

## 기술 스택
- Java 17, Spring Boot 3.5.6
- Spring Data JPA(Hibernate), MySQL 8.0
- Apache Kafka, Zookeeper
- Redis
- Spring Security + JWT
- Swagger UI (springdoc-openapi)

## 아키텍처
```mermaid
flowchart LR
  Client -->|HTTP| App[Spring Boot]
  subgraph Infra
    DB[(MySQL)]
    R[(Redis)]
    K[(Kafka)]
    Z[(Zookeeper)]
  end
  App --> DB
  App --> R
  App -->|produce/consume| K
  K --- Z


# NewBoard — Spring Boot 게시판 (Kafka/Redis/JWT/Docker/EC2)

간단하지만 실서비스 구조를 갖춘 게시판 백엔드입니다. 게시글 CRUD, JWT 인증, Kafka 로깅, Redis 캐시를 포함하며 Docker Compose로 로컬/EC2 어디서든 동일하게 실행됩니다.

## 🔎 개요
- **목표**: 인증/메시징/캐시/문서화/배포 흐름을 한 번에 보여주는 포트폴리오 프로젝트
- **주요 기능**: Article CRUD, 회원가입/로그인(JWT), Kafka 로그 적재, Redis 캐시, Swagger UI

## 🛠️ 기술 스택
- Java 17, Spring Boot 3.5.x (Security 6, JPA)
- MySQL 8, Redis, Kafka
- JWT(HS256), springdoc-openapi
- Docker/Compose, AWS EC2

## 🚀 빠른 시작 (로컬)
```bash
git clone https://github.com/jngs-park/newBoard.git
cd newBoard
./mvnw clean package -DskipTests
docker compose up -d --build
open http://localhost:8081/swagger-ui/index.html
```
