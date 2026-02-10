
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

### 설정 파일
.env에 다음과 같이 설정:
```env
DOMAIN_OR_PUBLIC_IP=localhost
OPENVIDU_SECRET=openvidu1123
CERTIFICATE_TYPE=selfsigned
HTTP_PORT=80
HTTPS_PORT=443
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
요약하면 network_mode= host 지우기




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

