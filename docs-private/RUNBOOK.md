주요 엔드포인트
•	인증
◦	POST /api/auth/signup : 회원가입
◦	POST /api/auth/login : 로그인 → {"token":"<JWT>"}
•	게시글
◦	GET /api/articles : 목록
◦	GET /api/articles/{id} : 단건
◦	POST /api/articles : 생성 (JWT 필요)
◦	PUT /api/articles/{id} : 수정 (JWT 필요)
◦	DELETE /api/articles/{id} : 삭제 (JWT 필요)
•	로그/캐시
◦	GET /api/logs : Kafka Consumer 적재 로그
◦	GET /api/logs/latest : 최신 로그(캐시)
Swagger
•	로컬: http://localhost:8081/swagger-ui
•	EC2: http://<EC2_IP>:8081/swagger-ui
주의: 끝에 슬래시(/)를 붙이면 빈 화면일 수 있어요. .../swagger-ui 또는 .../swagger-ui/index.html로 접속하세요.
빠른 시작 (로컬)

./mvnw clean package -DskipTests
docker compose up -d --build
curl -I http://localhost:8081/swagger-ui/swagger-ui.css   # 200이면 정상
JWT 빠른 테스트

# 회원가입(중복ID 피해서)
curl -s -X POST http://localhost:8081/api/auth/signup \
-H "Content-Type: application/json" \
-d '{"username":"tester","password":"pass1234"}'

# 로그인 → 토큰 추출
TOKEN=$(curl -s -X POST http://localhost:8081/api/auth/login \
-H "Content-Type: application/json" \
-d '{"username":"tester","password":"pass1234"}' | jq -r .token)

# 생성 (JWT 필요)
curl -s -X POST http://localhost:8081/api/articles \
-H "Content-Type: application/json" \
-H "Authorization: Bearer $TOKEN" \
-d '{"title":"JWT 게시글","content":"내용"}' | jq .

# 목록
curl -s http://localhost:8081/api/articles | jq .
포트/매핑
•	App: 8080 → 8081
•	MySQL: 3306 → 3308
•	Redis: 6379 → 6380
•	Kafka: 9092 → 9093
•	Zookeeper: 2181 → 2182
라이선스
MIT

### docs/RUNBOOK.md (복붙)
```markdown
# RUNBOOK — NewBoard 운영/점검 가이드

## 1. 구동/재기동
```bash
git pull
sudo docker compose down
sudo docker compose up -d --build
sudo docker compose ps
sudo docker logs --tail=200 -f newboard-app
헬스 확인

curl -I http://<EC2_IP>:8081/swagger-ui/swagger-ui.css
curl -s http://<EC2_IP>:8081/v3/api-docs | head
2. 기능 점검(스모크 테스트)
인증/JWT

curl -s -X POST http://<EC2_IP>:8081/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"username":"smoke","password":"pass1234"}'

TOKEN=$(curl -s -X POST http://<EC2_IP>:8081/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"smoke","password":"pass1234"}' | jq -r .token)
echo $TOKEN

curl -s -X POST http://<EC2_IP>:8081/api/articles \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"스모크","content":"ok"}' | jq .

curl -s http://<EC2_IP>:8081/api/articles | jq .
Kafka / Redis

sudo docker exec -it newboard-kafka \
  kafka-console-consumer --bootstrap-server localhost:9092 \
  --topic article-topic --from-beginning

sudo docker exec -it newboard-redis redis-cli GET lastArticle
3. 자주 겪는 오류 & 처리
Swagger가 빈 화면
	•	URL 끝에 / 붙었는지 확인
	◦	.../swagger-ui/ ❌
	◦	.../swagger-ui 또는 .../swagger-ui/index.html ✅
403 Forbidden
	•	Authorization: Bearer <토큰> 헤더 누락/오타 확인
	•	토큰 만료 시 재로그인
500 Internal Server Error
	•	앱 로그: sudo docker logs --tail=200 newboard-app
	•	DB 연결/엔티티 직렬화/마이그레이션 확인
톰캣 로그에 Invalid character found in method name
	•	HTTPS 요청이 HTTP 포트(8081)로 들어온 경우의 경고 → 무시 가능
디스크 부족
	•	EBS 볼륨 확장 + 파일시스템 리사이즈 후 재시도

---

# 2) IntelliJ에서 커밋 & 푸시 (master)

1. 우상단 **Git 브랜치** 확인 → `master` 인지 체크.  
   - 아니라면: **Git** 메뉴 → **Branches…** → `master` 체크아웃.
2. **Commit**(Mac: `⌘K`, Win/Linux: `Ctrl+K`) → `README.md`, `docs/RUNBOOK.md` 선택 → 메시지 입력:  
   - `docs: add README and RUNBOOK`  
   → **Commit** 또는 **Commit and Push**.
3. 커밋만 했다면 **Push**(Mac: `⌘⇧K`, Win/Linux: `Ctrl+Shift+K`) → 원격 `origin`의 `master`로 푸시.

> 원격이 설정되어 있어야 해요. 이미 깃허브 레포와 연결되어 있으면 생략됩니다. (주소: `https://github.com/jngs-park/newBoard.git`)

---

# (선택) 터미널로 한 방에 하기
프로젝트 루트에서:

```bash
# 브랜치 확인
git branch

# docs 폴더 없으면 생성
mkdir -p docs

# 파일 저장
cat > README.md <<'EOF'
(위의 README 내용 전체 붙여넣기)
EOF

cat > docs/RUNBOOK.md <<'EOF'
(위의 RUNBOOK 내용 전체 붙여넣기)
EOF

# 커밋/푸시
git add README.md docs/RUNBOOK.md
git commit -m "docs: add README and RUNBOOK"
git push origin master





-------
# RUNBOOK — 운영/점검/트러블슈팅

## 빌드 & 기동
```bash
./mvnw clean package -DskipTests
docker compose up -d --build
```

## 상태 확인
```bash
docker compose ps
docker logs -f newboard-app
curl -I http://localhost:8081/swagger-ui/index.html
curl http://localhost:8081/v3/api-docs | jq .
```

## Kafka/Redis 점검
```bash
docker exec -it newboard-kafka kafka-console-consumer --bootstrap-server localhost:9092   --topic article-topic --from-beginning --timeout-ms 5000
docker exec -it newboard-redis redis-cli GET lastArticle
```


