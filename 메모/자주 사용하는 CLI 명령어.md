#Linux 
#Docker 
#nginx

---
# Docker
**실행중인 컨테이너 확인**
docker ps

**최근 로그 확인**
docker logs --tail 100 <컨테이너_ID_or_NAME>

**이미지 다시 빌드 (캐시 무시 추천)**
docker build --no-cache -t deskit:latest

**redis-stack 컨테이너에 접속**
docker exec -it <컨테이너이름> redis-cli

**컨테이너 시작/중지**
docker start
docker stop

**컨테이너 삭제/생성**
docker rm <컨테이너이름>
docker run -d \
  --name redis-stack \
  -p 6379:6379 \
(docker desktop이면 이미지에서 컨테이너 생성 가능)


### Docker Compose
**도커 서버 끄기**
docker compose down

**켜는 방법**
docker compose up -d

---
# Nginx
**nginx 테스트**
nginx -t

**nginx 재업**
nginx -s reload


/var/www/deskit/
-> 프론트 파일 배포 경로

/etc/nginx/sites-enabled/
 -> nginx 폴더 경로