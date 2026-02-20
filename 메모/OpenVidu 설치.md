#OpenVidu
#Docker 

---
# 로컬에 설치
PowerShell -> WSL 터미널 접속

###### OpenVidu 설치
```bash
curl https://s3-eu-west-1.amazonaws.com/aws.openvidu.io/install_openvidu_latest.sh | bash
```

자동으로 openvidu 디렉토리 생성하므로 그냥 /home/dynii에서 바로 실행하면 됨
docker desktop이 실행된 상태에서 명령어 실행해야 함

**1. .env 파일 수정**

**2. docker-compose.override.yml, docker-compose.yml 파일 수정**

**3. default.conf 파일 호스트로 복사 및 수정**

**4. localhost:4443/dashboard 접속해서 테스트 진행**


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

### 설정 파일
.env에 다음과 같이 설정:
```env
DOMAIN_OR_PUBLIC_IP=localhost
OPENVIDU_SECRET=openvidu1123
CERTIFICATE_TYPE=selfsigned
HTTP_PORT=80
HTTPS_PORT=4443
```

docker-compose.override.yml 설정:
```yml
version: '3.1'
services:
    # --------------------------------------------------------------
    #    Change this if your want use your own application.
    #    It's very important expose your application in port 5442
    #    and use the http protocol.
    #
    #    Default Application
    #
    #    Openvidu-Call Version: 2.32.1
    # --------------------------------------------------------------
    nginx:
      ports:
        - "4443:443"
        - "8080:80"

    openvidu-server:
      environment:
        - KMS_URIS=["ws://kms:8888/kurento"]
```
기존 내용 삭제 후 위의 내용으로 수정

docker-compose.yml 설정:
```yml
version: '3.1'
services:
    openvidu-server:
        image: openvidu/openvidu-server:2.32.1
        restart: on-failure
        entrypoint: ['/usr/local/bin/entrypoint.sh']
        volumes:
            - ./coturn:/run/secrets/coturn
            - /var/run/docker.sock:/var/run/docker.sock
            - ${OPENVIDU_RECORDING_PATH}:${OPENVIDU_RECORDING_PATH}
            - ${OPENVIDU_RECORDING_CUSTOM_LAYOUT}:${OPENVIDU_RECORDING_CUSTOM_LAYOUT}
            - ${OPENVIDU_CDR_PATH}:${OPENVIDU_CDR_PATH}
        env_file:
            - .env
        environment:
            - SERVER_SSL_ENABLED=false
            - SERVER_PORT=5443
            - KMS_URIS=["ws://localhost:8888/kurento"]
            - COTURN_IP=${COTURN_IP:-auto-ipv4}
            - COTURN_PORT=${COTURN_PORT:-3478}
        logging:
            options:
                max-size: "${DOCKER_LOGS_MAX_SIZE:-100M}"

    kms:
        image: ${KMS_IMAGE:-kurento/kurento-media-server:7.3.0}
        restart: always
        ulimits:
            core: -1
            nofile:
                soft: 65536
                hard: 65536
        volumes:
            - /opt/openvidu/kms-crashes:/opt/openvidu/kms-crashes
            - ${OPENVIDU_RECORDING_PATH}:${OPENVIDU_RECORDING_PATH}
            - /opt/openvidu/kurento-logs:/opt/openvidu/kurento-logs
        environment:
            - KMS_MIN_PORT=40000
            - KMS_MAX_PORT=57000
            - GST_DEBUG=${KMS_DOCKER_ENV_GST_DEBUG:-}
            - KURENTO_LOG_FILE_SIZE=${KMS_DOCKER_ENV_KURENTO_LOG_FILE_SIZE:-100}
            - KURENTO_LOGS_PATH=/opt/openvidu/kurento-logs
        logging:
            options:
                max-size: "${DOCKER_LOGS_MAX_SIZE:-100M}"

    coturn:
        image: openvidu/openvidu-coturn:2.32.1
        restart: on-failure
        ports:
            - "${COTURN_PORT:-3478}:${COTURN_PORT:-3478}/tcp"
            - "${COTURN_PORT:-3478}:${COTURN_PORT:-3478}/udp"
        env_file:
            - .env
        volumes:
            - ./coturn:/run/secrets/coturn
        command:
            - --log-file=stdout
            - --listening-port=${COTURN_PORT:-3478}
            - --fingerprint
            - --min-port=${COTURN_MIN_PORT:-57001}
            - --max-port=${COTURN_MAX_PORT:-65535}
            - --realm=openvidu
            - --verbose
            - --use-auth-secret
            - --static-auth-secret=$${COTURN_SHARED_SECRET_KEY}
        logging:
            options:
                max-size: "${DOCKER_LOGS_MAX_SIZE:-100M}"

    nginx:
        image: openvidu/openvidu-proxy:2.32.1
        restart: always
        volumes:
		    - ./nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf:ro
            - ./certificates:/etc/letsencrypt
            - ./owncert:/owncert
            - ./custom-nginx-vhosts:/etc/nginx/vhost.d/
            - ./custom-nginx-locations:/custom-nginx-locations
            - ${OPENVIDU_RECORDING_CUSTOM_LAYOUT}:/opt/openvidu/custom-layout
        environment:
            - DOMAIN_OR_PUBLIC_IP=${DOMAIN_OR_PUBLIC_IP}
            - CERTIFICATE_TYPE=${CERTIFICATE_TYPE}
            - LETSENCRYPT_EMAIL=${LETSENCRYPT_EMAIL}
            - PROXY_HTTP_PORT=${HTTP_PORT:-}
            - PROXY_HTTPS_PORT=${HTTPS_PORT:-}
            - PROXY_HTTPS_PROTOCOLS=${HTTPS_PROTOCOLS:-}
            - PROXY_HTTPS_CIPHERS=${HTTPS_CIPHERS:-}
            - PROXY_HTTPS_HSTS=${HTTPS_HSTS:-}
            - ALLOWED_ACCESS_TO_DASHBOARD=${ALLOWED_ACCESS_TO_DASHBOARD:-}
            - ALLOWED_ACCESS_TO_RESTAPI=${ALLOWED_ACCESS_TO_RESTAPI:-}
            - PROXY_MODE=CE
            - WITH_APP=true
            - SUPPORT_DEPRECATED_API=${SUPPORT_DEPRECATED_API:-false}
            - REDIRECT_WWW=${REDIRECT_WWW:-false}
            - WORKER_CONNECTIONS=${WORKER_CONNECTIONS:-10240}
            - PUBLIC_IP=${PROXY_PUBLIC_IP:-auto-ipv4}
        logging:
            options:
                max-size: "${DOCKER_LOGS_MAX_SIZE:-100M}"
```
기본 설정인 network_mode= host 지우기

### 설정 Tip
각 컨테이너 전용 터미널: docker desktop에서 확인
메인 터미널: PowerShell(관리자 말고 일반) -> WSL(Ubuntu) 터미널 열기
각종 설정 파일 열기: MobaXTerm 텍스트 에디터에서 열기

### 문제 해결을 위한 Ubuntu 명령어

포트가 잘 열려있는지 확인
```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"
```
openvidu-nginx와 openvidu-server가 열려있으면 됨

override 파일 열기
```bash
nano docker-compose.override.yml
```

openvidu 서버 로그 보기
```bash
docker logs openvidu-openvidu-server-1 --tail=120
```

연결 테스트
```bash
curl -k -u OPENVIDUAPP:openvidu1123 -o /dev/null -w "%{http_code}\n" https://localhost:4443/openvidu/api/config
```

컨테이너를 다시 생성하여 재시작하기
```bash
docker compose down
docker compose up -d --force-recreate
```

openvidu-server가 컨테이너 안에서 어떤 포트로 LISTEN 하는지 확인
```bash
docker exec -it openvidu-openvidu-server-1 sh -lc 'ss -lntp | egrep ":(4443|5443)\s" || true'
```

컨테이너 완전히 제거 후 재시작
```bash
docker rm -f openvidu-app-1
docker compose down --remove-orphans
docker compose up -d
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

**502 Bad Gateway 관련**
다음과 같은 오류 로그 확인시:
```bash
[error] 83#83: *1 connect() failed (111: Connection refused) while connecting to upstream, client: 172.18.0.1, server: localhost, request: "GET /openvidu/api/config HTTP/1.1", upstream: "http://127.0.0.1:5443/openvidu/api/config", host: "localhost:4443"
```
nginx 컨테이너 기준 'localhost' = nginx 컨테이너 자신이므로
자기 자신에서 5443 포트를 찾으면 없음(=502)

때문에 openvidu-server 컨테이너에서 5443 포트를 찾게 해줘야 함
이를 위해서는 nginx 설정 파일을 수정해야 하고
수정 편의 + 컨테이서 재생성시 초기화 방지를 위해 호스트에 설정 파일 가져온 뒤,
docker-compose.yml에서 해당 설정 파일을 사용하도록 설정 필요

###### docker-compose.yml에 nginx conf를 볼륨으로 고정 마운트하기
호스트에 conf 파일 하나 만들어서, 컨테이너가 매번 그걸 쓰게 하기

openvidu 디렉토리에서
```bash
mkdir -p ./nginx/conf.d
docker cp openvidu-nginx-1:/etc/nginx/conf.d/default.conf ./nginx/conf.d/default.conf
```

default.conf 열어서 수정
```bash
nano nginx/conf.d/default.conf
```

server localhost:5443 -> server openvidu-server:5443 
```bash
upstream openviduserver {
    server openvidu-openvidu-server-1:5443;
    # 또는 서비스명이 있다면 server openvidu-server:5443;
}
```
컨테이너명 또는 서비스명인데 일단 컨테이너명이 확실함

###### docker-compose.yml에 마운트 추가

services - nginx - volumes에 추가
```bash
services:
  nginx:
    volumes:
      - ./nginx/conf.d/default.conf:/etc/nginx/conf.d/default.conf:ro
```

적용
```bash
docker compose up -d
# 또는
docker compose up -d --force-recreate openvidu-nginx
```

 테스트
```bash
SECRET="설정한 비밀번호"
curl -vk -u OPENVIDUAPP:$SECRET https://localhost:4443/openvidu/api/sessions \
  -H "Content-Type: application/json" \
  -d '{"customSessionId":"test"}' 2>&1 | head -n 60
```
중간에 HTTP/1.1 200 또는 JSON 반환하면 성공

---
# 서버에 배포
OpenVidu 미디어 서버 전용 EC2 인스턴스 만들어주기

도메인 하나 장만하기
https://xn--220b31d95hq8o.xn--3e0b707e/

EC2에 Docker, Docker Compose 설치
```bash
sudo apt update -y
sudo apt install -y ca-certificates curl gnupg lsb-release

# Docker 설치 (공식 방식)
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo systemctl enable --now docker
docker --version
docker compose version

```

OpenVidu 설치 스크립트 실행 (공식 방식)
```bash
sudo su
cd /opt
curl https://s3-eu-west-1.amazonaws.com/aws.openvidu.io/install_openvidu_latest.sh | bash
```

`/opt/openvidu/.env` 핵심 설정
```bash
cd /opt/openvidu
nano .env
```

필수 예시
```bash
DOMAIN_OR_PUBLIC_IP=openvidu.yourdomain.com
OPENVIDU_SECRET=아주강력한비밀번호

# HTTPS (권장: letsencrypt)
CERTIFICATE_TYPE=letsencrypt
LETSENCRYPT_EMAIL=you@example.com
```

OpenVidu 실행
```bash
cd /opt/openvidu
./openvidu start
```

정상 기동되면 보통 아래로 접속
- OpenVidu Server: `https://DOMAIN_OR_PUBLIC_IP/`
- Dashboard: `https://DOMAIN_OR_PUBLIC_IP/dashboard/`

OpenVidu Call(기본 데모 앱)
```
ID : admin
PW : OPENVIDU_SECRET과 동일
```

세션 생성 및 중단 가능

녹화 기능 컨테이너 추가 설치
```bash
docker pull openvidu/openvidu-recording:2.32.1
```

---
## 1) 네트워크/NAT/방화벽 체크 (가장 중요해요)
로컬호스트에서 성공한 건 “내 PC/서버 내부”에서는 된다는 뜻이고, **외부 사용자(다른 네트워크, 모바일 LTE, 회사망)** 에서는 TURN/방화벽 때문에 실패할 수 있어요.

- **UDP 3478**(coturn) 외부에서 열려 있는지
- 기업망/학교망처럼 UDP 막히는 환경에서 **TURN TCP(보통 443/80/3478 TCP)** 로라도 붙는지
- 실제 도메인으로 운영하면 `DOMAIN_OR_PUBLIC_IP`, `OPENVIDU_PUBLICURL`이 외부 도메인 기준으로 맞는지
👉 추천 테스트: 휴대폰 LTE로 접속해서 1:1, 1:3 통화 해보기

## 2) 인증/보안 (OPENVIDU_SECRET 고정 + 접근제어)
- `OPENVIDU_SECRET`은 **랜덤 생성이 아니라 고정**(환경변수/.env)으로 관리하세요.
- OpenVidu REST API(`/openvidu/api/**`)는 **외부에 그대로 노출하면 안 좋아요.**
    - 프론트에서 직접 REST API 호출하지 말고
    - **백엔드가 세션/토큰을 발급**해주는 구조로 가는 게 안전해요.
- nginx에서 `/dashboard`는 운영환경에선 IP 제한 또는 Basic Auth 거는 걸 추천해요.

## 3) TLS/도메인/웹소켓 정리
- 실제 서비스는 `localhost` 말고 도메인 + 유효한 인증서로 운영해야 브라우저 정책에서 자유로워요.
- 프론트가 다른 도메인이라면 CORS/쿠키/WS 경로도 함께 정리해야 해요.
- 프록시를 이미 쓰고 있으니, WebSocket/HTTP2/업그레이드 헤더 설정은 그대로 유지 점검해두면 좋아요.

## 4) 리소스/성능 (KMS 서버가 병목입니다)
OpenVidu CE에서 미디어는 **KMS(Kurento)** 가 먹어요.
- 동시 접속 늘릴수록 **CPU, 네트워크, 디스크(녹화 시)** 가 빠르게 증가
- 서버 스펙 대비 **동시 참가자 목표치**를 정하고 부하 테스트를 해두는 게 좋아요.
- 최소 체크:
    - `docker stats`로 openvidu-kms CPU/RAM 추적
    - 실제로 4~10명 방 만들어서 5~10분 테스트

## 5) 운영 관측성(로그/모니터링)
장애 나면 “왜 안 붙는지”가 바로 보여야 해요.
- nginx: 4xx/5xx, upstream error 로그
- openvidu-server: session/token/REST 에러
- coturn: TURN allocation/permission 이슈
- kms: 미디어 파이프라인 에러
👉 추천: 컨테이너 로그를 파일/수집기로 모으거나 최소 `docker logs` tail 기준을 정해두기

## 6) 장애 대비(재시작/원복 방지)
이번에 겪은 것처럼 “재기동하면 설정 원복”이 가장 흔한 운영 사고예요.
- **docker-compose/.env를 소스관리(Git)** 로 고정
- nginx conf, letsencrypt 경로 등은 **볼륨 마운트로 영속화**
- 재기동 시나리오(서버 재부팅, 컨테이너 재시작) 한 번씩 실제로 해보기

# 서버 이전시 핵심 포인트
## 1) 아키텍처 맵핑
- **NCP 서버(Compute)** → **EC2** (또는 ECS)
- **NCP Object Storage** → **S3**
- **NCP 로드밸런서/도메인/SSL** → **ALB + ACM** 또는 **Nginx + Let’s Encrypt**
- **DB 서버** → **RDS(MySQL)** 또는 EC2 자체 운영
- **Redis** → **ElastiCache** 또는 EC2
- **OpenVidu(KMS/coturn/nginx 포함)** → **EC2 단독 인스턴스**(초기엔 이게 제일 쉬움)

## 2) OpenVidu를 AWS에서 띄울 때 제일 중요한 것: 방화벽/포트/네트워크
AWS 보안 그룹(Security Group)에서 최소 아래를 확실히 열어야 해요(구성/버전에 따라 조금 다름).
- **TCP 443(또는 4443)**: 대시보드/REST/웹 접속
- **UDP 3478**: TURN(코어)
- **(필수 가능성 큼) UDP 미디어 포트 범위**: WebRTC RTP/RTCP (OpenVidu/Kurento가 쓰는 포트 대역)
    - 이걸 안 열면 “대시보드 테스트는 되는데 외부/회사망에서 자주 실패”가 터져요.
- 회사망/학교망 대응을 위해 **TURN TCP도 고려**(UDP 막히는 환경 대비)
✅ 권장: EC2에 띄운 다음 **휴대폰 LTE/다른 집 인터넷**에서 접속 테스트까지 해야 “진짜 성공”이에요.

## 3) PUBLIC URL/도메인/SSL 설정 (이전 때 502 겪었으니까 특히)
AWS에서 운영 도메인을 쓰면:
- `DOMAIN_OR_PUBLIC_IP` / `OPENVIDU_PUBLICURL`이 **반드시 “외부에서 접속하는 도메인” 기준**으로 맞아야 해요.
- 포트도 `:4443` 같은 임시 포트는 운영에선 피하고, 가능하면 **443 표준**으로 정리하는 게 좋아요.
- SSL은 2가지 선택:
    - **ALB + ACM(추천)**: 인증서 관리가 편함
    - **EC2 Nginx + Let’s Encrypt**: 단독 구성 간단하지만 갱신/운영 부담

## 4) “썸네일 업로드가 클라우드 서버로 박혀있는 코드”를 S3로 바꿀 때
보통 기존 코드가 이런 형태 중 하나예요:
- 서버 로컬 경로에 저장(`/var/www/...`, `uploads/`)
- 특정 클라우드 스토리지 엔드포인트로 업로드(NCP Object Storage 등)
- CDN URL을 하드코딩
AWS로 옮길 때 체크:
- S3는 **Public 공개 업로드를 지양**하고, 보통
    1. 백엔드가 S3에 업로드  
        또는
    2. **Presigned URL**로 프론트가 직접 업로드(백엔드는 서명만 발급)  
        둘 중 하나로 가요.
- 이미지 썸네일은 조회가 많으니
    - S3 **객체 키 규칙(방송ID/날짜/UUID)** 통일
    - **CloudFront** 붙일지(권장)
    - 캐시/무효화 전략
- 업로드 실패/재시도/파일 크기 제한(멀티파트)도 함께 정리

## 5) IAM/키 관리(진짜 많이 터지는 부분)
- EC2/ECS에서 S3 접근할 때 **Access Key를 서버에 박는 방식보다**  
    **IAM Role(Instance Profile)** 로 권한 부여하는 게 베스트예요.
- 권한은 최소화:
    - 특정 버킷/특정 prefix에만 `PutObject`, `GetObject` 등

## 6) 배포 파이프라인 / 환경변수 교체
NCP에서 사용하던 것들이 AWS로 바뀌면 환경변수도 같이 바뀌어요.
- `S3_BUCKET`, `S3_REGION`, `S3_BASE_URL`(또는 CloudFront URL)
- OpenVidu 도메인/포트
- CORS 허용 도메인(프론트 도메인 변경)
- OAuth redirect URI(도메인이 바뀌면 다시 등록 필요)

## 7) 인증/토큰 얘기(지금 설계 방향 좋아요)
말씀하신 것처럼 **access/refresh 토큰은 백엔드 인증용으로 유지**하고,  
OpenVidu는 별도로 **세션/토큰(OpenVidu Token)** 이 필요해요.

권장 플로우:
1. 프론트가 access token으로 백엔드 호출
2. 백엔드가 사용자 권한 확인 후 OpenVidu 세션 생성/조회
3. 백엔드가 OpenVidu token 발급해서 프론트에 내려줌
4. 프론트는 그 OpenVidu token으로 join

즉, “우리 JWT(access/refresh)”는 **권한 확인**, “OpenVidu token”은 **미디어 세션 입장권**이에요.

# 인증서 발급
HTTPS 연결을 위해 필요
Let's encrypt 방식 + Certbot
