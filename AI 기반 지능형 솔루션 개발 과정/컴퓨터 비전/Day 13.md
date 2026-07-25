#KDT

---
# CUDA 세팅
### NVIDIA Studio 드라이버 설치
nvidia.com/ko-kr/drivers/details/257330
PC 사양에 맞는 걸로 검색해서 들어감
(CUDA 최소 사양은 그래픽카드 3GB GTX 1060)

### Anaconda3 설치
그냥 설치

### CUDA Toolkit 11.8 설치

기본 설치 경로:
`C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v버전`

보통은 Toolkit 설치에서 오류가 발생하는데,
이럴 때 NVIDIA Frameview SDK를 삭제하고 다시 진행

### cuDNN 설치
8.6 for CUDA 11.x 로 설치
zip파일 압축 풀고 안에 있던 폴더+파일들을 Toolkit 설치 경로에 덮어쓰기

### 환경변수 확인
제어판 -> 사용자 계정 -> 환경변수 도 있고
시스템 환경변수에서도 확인
CUDA ... 2가지 정도


