
---
# 로컬에 설치
PowerShell -> WSL 터미널 접속

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

### 문제 해결

포트가 잘 열려있는지 확인
```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"
```
openvidu-nginx와 openvidu-server가 열려있으면 됨

override 파일 열기
```bash
nano docker-compose.override.yml
```

포트 매핑 추가
```yaml
services:
  nginx:
    ports:
      - "4443:443"
      - "8080:80"
```

