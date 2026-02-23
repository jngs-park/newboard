# 04. Redis (캐시)

## 목표
DB를 매번 조회하지 않아도 되는 “가벼운 값”을 Redis에 저장해 빠르게 응답

- 예: 최근 게시글 제목(`lastArticle`)

## 검증 커맨드
```bash
docker exec -it newboard-redis redis-cli GET lastArticle
```

## 흔한 장애 포인트
- 포트 매핑(호스트 6380 → 컨테이너 6379) 착각
- 직렬화 설정 혼용으로 값이 깨져 보임
