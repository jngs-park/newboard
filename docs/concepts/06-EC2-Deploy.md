# 06. EC2 Deploy (배포/운영)

## 배포 루틴
```bash
cd ~/newBoard
git pull origin master
./mvnw clean package -DskipTests
sudo docker compose up -d --build
sudo docker logs -f newboard-app
```

## 디스크 부족(no space left on device) 대응
- EBS 볼륨 확장 + 파티션/파일시스템 확장
- 도커 정리: `docker system prune -a`

## Swagger “하얀 화면” 체크
HTML이 200이어도 JS/CSS가 404면 빈 화면처럼 보일 수 있음.
- `/swagger-ui/index.html` 200
- `/swagger-ui/swagger-ui-bundle.js` 200
- `/v3/api-docs` 200
브라우저 DevTools(Network/Console)에서 404/CORS 확인
