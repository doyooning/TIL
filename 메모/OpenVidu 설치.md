
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

