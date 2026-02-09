#Linux 
#Docker 
#nginx

---
# Docker
**실행중인 컨테이너 확인**
```
docker ps
```


**최근 로그 확인**
```
docker logs --tail 100 <컨테이너_ID_or_NAME>
```


**이미지 다시 빌드 (캐시 무시 추천)**
```
docker build --no-cache -t deskit:latest
```


**redis-stack 컨테이너에 접속**
```
docker exec -it <컨테이너이름> redis-cli
```


**컨테이너 시작/중지**
```
docker start
docker stop
```


**컨테이너 삭제/생성**
```
docker rm <컨테이너이름>
docker run -d \
  --name redis-stack \
  -p 6379:6379 \
```
(docker desktop이면 이미지에서 컨테이너 생성 가능)


### Docker Compose
**도커 서버 켜기/끄기**
```
docker compose up -d
docker compose down
```


# Redis CLI
**인덱스 지우기**
```
redis-cli --scan --pattern "eval-doc:*" | xargs redis-cli del
```



---
# Nginx
**nginx 테스트**
```
nginx -t
```


**nginx 재업**
```
nginx -s reload
```


/var/www/deskit/
-> 프론트 파일 배포 경로

/etc/nginx/sites-enabled/
 -> nginx 폴더 경로


---
# WSL
윈도우에서 리눅스 명령어 사용해야 할 때 WSL로 들어감
첫 진입할 때는 윈도우 탐색기에서 Linux 디렉토리 들어가면 됨

###### 기본 사용
**PowerShell -> WSL 터미널**
```powershell
wsl
```

이후에는 리눅스 명령어 사용 가능
###### 주의사항
docker desktop이 설치되어 있는 경우 기본으로 docker-desktop WSL 배포판으로 진입함
이 경우 bash/curl 없음
따라서 Ubuntu에서 설치를 진행해야 함

###### 기본 위치
```
home/dynii
```

###### Ubuntu 세팅
계정명: dynii
비번: 11ehdsb23

**WSL 확인**
```powershell
wsl -l -v
```

**Ubuntu로 지정해서 들어가기**
```powershell
wsl -d Ubuntu
```

**Ubuntu 설치**
```powershell
wsl --install -d Ubuntu
```

**Ubuntu WSL 안에서 curl/bash 설치**
```bash
sudo apt update
sudo apt install -y curl bash
```


**docker 권한 문제 해결 (permission denied)**
```bash
sudo usermod -aG docker $USER
```

WSL 재시작 필요

PowerShell에서 
```powershell
wsl --shutdown
wsl -d Ubuntu
```

위치 pwd로 확인, mnt/...이면
```bash
cd ~
```

다시 pwd하면 /home/dynii

Ubuntu WSL에서
```
docker ps
```

컨테이너 목록 출력되면 성공

###### OpenVidu 설치
```bash
curl https://s3-eu-west-1.amazonaws.com/aws.openvidu.io/install_openvidu_latest.sh | bash
```

자동으로 openvidu 디렉토리 생성하므로 그냥 /home/dynii에서 바로 실행하면 됨
docker desktop이 실행된 상태에서 명령어 실행해야 함

**.env 파일 수정**
```env
# OpenVidu 서버 비밀번호 (중요)
OPENVIDU_SECRET=MY_SECRET_KEY

# 로컬에서는 https 인증서 필요 없음
DOMAIN_OR_PUBLIC_IP=localhost

# HTTPS 비활성화 (로컬 개발용)
CERTIFICATE_TYPE=none

# 포트
HTTP_PORT=4444
HTTPS_PORT=443

# 녹화 사용 안 하면 false
OPENVIDU_RECORDING=false
```

OpenVidu가 정상 실행된 상태
```powershell
PS C:\Users\dynii> docker ps
CONTAINER ID   IMAGE                                COMMAND                  CREATED         STATUS                          PORTS                                                                                      NAMES
a8510857acce   openvidu/openvidu-proxy:2.32.1       "/docker-entrypoint.…"   5 minutes ago   Restarting (0) 56 seconds ago                                                                                              openvidu-nginx-1
328b3373f7f1   openvidu/openvidu-call:2.32.1        "docker-entrypoint.s…"   5 minutes ago   Up 5 minutes                                                                                                               openvidu-app-1
d910c689e46b   openvidu/openvidu-coturn:2.32.1      "docker-entrypoint.s…"   5 minutes ago   Up 5 minutes                    0.0.0.0:3478->3478/tcp, 0.0.0.0:3478->3478/udp, [::]:3478->3478/tcp, [::]:3478->3478/udp   openvidu-coturn-1
d23febd61a98   openvidu/openvidu-server:2.32.1      "/usr/local/bin/entr…"   5 minutes ago   Up 5 minutes                                                                                                               openvidu-openvidu-server-1
56f9961eb00b   kurento/kurento-media-server:7.3.0   "/entrypoint.sh"         5 minutes ago   Up 5 minutes (healthy)                                                                                                     openvidu-kms-1
```

