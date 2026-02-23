# 05. Docker Compose (로컬/EC2 동일 실행)

## 목표
**App/MySQL/Kafka/Zookeeper/Redis**를 한 번에 기동해 환경 차이를 최소화

## 실행 순서
```bash
./mvnw clean package -DskipTests
docker compose up -d --build
```

## jar COPY 전략(Dockerfile) 주의
Dockerfile이 `COPY target/*.jar` 방식이면,
- EC2에서도 반드시 mvn package로 jar 생성 후 compose build 해야 함.
- jar가 없으면: `COPY target/...jar not found`
