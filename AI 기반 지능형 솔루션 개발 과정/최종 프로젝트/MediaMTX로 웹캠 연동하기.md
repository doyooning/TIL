
---
```
[웹캠이 있는 기기(노트북)]
  내장 웹캠
     ↓
  FFmpeg
     ↓ RTSP over TCP
     ↓ 인터넷
     ↓
[웹캠 수집 정보를 확인할 기기(PC)]
  Windows
     ↓
  WSL2
  ├─ MediaMTX :8554
  ├─ Stream Worker
  ├─ Inference Worker (GPU)
  └─ FastAPI :8000
```

### 1. GPU PC의 WSL에 MediaMTX를 띄우기

MediaMTX는 RTSP 서버를 기본적으로 `:8554`에서 열 수 있고, RTSP를 TCP 전용으로 제한할 수도 있음
이번처럼 인터넷을 넘기는 테스트에서는 UDP보다 **RTSP interleaved TCP**로 고정하는 편이 네트워크 구성이 훨씬 단순함

```bash
docker run -d \
  --name mediamtx \
  -p 8554:8554 \
  -p 8888:8888 \
  -p 8889:8889 \
  -p 8189:8189/udp \
  bluenviron/mediamtx:latest
```

### 2.  `GPU_PC_IP:8554 → WSL` 연결

여기서 GPU PC의 `1.xxx.xxx.110`이 **Windows PC가 인터넷에서 실제로 직접 갖는 공인 IP인지**, 아니면 공유기/기관 네트워크의 외부 IP인지에 따라 설정이 달라지는데,
WSL2 기본 네트워크는 NAT 구조라 Windows와 WSL이 별도 네트워크 계층을 가짐

Microsoft는 최신 Windows 11에서 `networkingMode=mirrored`를 지원하며, 이 경우 WSL의 외부 네트워크 접근 구성이 훨씬 단순해짐

GPU PC가 Windows 11이라면 **WSL mirrored networking을 먼저 사용**하는 쪽이 나음

```
C:\Users\<사용자명>\.wslconfig
```

이 경로에

```
[wsl2]
networkingMode=mirrored
```

를 넣고 wsl 재시작
(Docker Desktop 재시작)

> 파일이 없다면 생성하준다.
> `nano .wslconfig`

### 3. Windows 방화벽에서 8554/TCP를 허용

GPU PC의 관리자 PowerShell에서:

```powershell
New-NetFirewallRule `
  -DisplayName "MediaMTX RTSP 8554" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 8554 `
  -Action Allow
```

특정 포트에 대한 방화벽을 열어주는 작업

### 4. 서버 PC 앞에 공유기/NAT가 있다면 포트포워딩도 필요
```
Internet
   ↓
GPU PC IP
   ↓
공유기
   ↓
192.168.0.50
   ↓
GPU PC
```

이런 구조라면 공유기에서 

```
WAN TCP 8554
     ↓
GPU PC 내부 IP:8554
```

포트포워딩 필요

이러면 공유기 설정은

```
외부 8554/TCP
→ 192.168.0.50:8554
```

### 5. 노트북에서 웹캠을 FFmpeg로 GPU PC에 송출
노트북에는 FFmpeg만 있으면 됨

**FFmpeg 설치 방법**
Ubuntu:
```
sudo apt update
sudo apt install -y ffmpeg
```

Windows:
```
winget install Gyan.FFmpeg
```

웹캠 장치 이름 확인:

```
ffmpeg -list_devices true -f dshow -i dummy
```

여기서 "Integrated Camera" 가 나온다면

```powershell
ffmpeg `
  -f dshow `
  -i video="Integrated Camera" `
  -c:v libx264 `
  -preset ultrafast `
  -tune zerolatency `
  -pix_fmt yuv420p `
  -rtsp_transport tcp `
  -f rtsp `
  rtsp://1.xxx.xxx.110:8554/cam1
```

흐름:

```
Integrated Camera
       ↓
     FFmpeg
       ↓
H.264 encoding
       ↓
RTSP/TCP publish
       ↓
rtsp://GPU_PC_IP:8554/cam1
```

### 6. 서버에서 먼저 영상이 들어오는지만 확인
GPU PC의 WSL에서:

```
ffplay -rtsp_transport tcp rtsp://127.0.0.1:8554/cam1
```
(ffplay: 윈도우창 하나 띄워서 웹캠 영상 재생)

또는:

```
ffmpeg \
  -rtsp_transport tcp \
  -i rtsp://127.0.0.1:8554/cam1 \
  -f null -
```
(프레임 수신 현황을 터미널에서 확인)
