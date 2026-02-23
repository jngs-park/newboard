# 03. Kafka (이벤트/로그 적재)

## 목표
게시글 이벤트를 동기 로직에만 묶지 않고, **비동기 이벤트(카프카)** 로 확장 가능한 구조로 설계

- Producer: 게시글 생성 시 이벤트 발행
- Consumer: 이벤트 수신 후 DB에 로그 적재(감사/추적/분석)

## 토픽
- `article-topic`

## 검증 커맨드(컨테이너 내부)
```bash
docker exec -it newboard-kafka   kafka-console-consumer --bootstrap-server localhost:9092   --topic article-topic --from-beginning --timeout-ms 5000
```

## 흔한 장애 포인트
- 컨테이너 간 통신에서 `localhost` 사용 → 실패. **`newboard-kafka:9092`** 사용
