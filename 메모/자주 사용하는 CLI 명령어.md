#Linux 
#Docker 
#nginx

---
# Docker
**실행중인 컨테이너 확인**
docker ps

**도커 서버 끄기**
docker compose down

**켜는 방법**
docker compose up -d

**최근 로그 확인**
docker logs --tail 100 <컨테이너_ID_or_NAME>

**이미지 다시 빌드 (캐시 무시 추천)**
docker build --no-cache -t deskit:latest .


# Nginx
**nginx 테스트**
nginx -t

**nginx 재업**
nginx -s reload


/var/www/deskit/
-> 프론트 파일 배포 경로

/etc/nginx/sites-enabled/
 -> nginx 폴더 경로