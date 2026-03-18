
---
강의 자료
https://jscode.notion.site/12711062ff078055bd91e22b3f3e8992

# 쿠버네티스(Kubernetes)란?
다수의 컨테이너를 효율적으로 배포, 확장, 관리하기 위한 오픈소스 시스템
Docker Compose와 비슷한 느낌

**장점**
컨테이너 관리 자동화(배포, 확장, 업데이트)
부하 분산(로드 밸런싱)
스케일링이 쉬움
셀프 힐링

# 설치
직접 설치는 거의 안하고 Docker Desktop 이용

설치 방법 Windows (적용 방법)
https://hong-yp-ml-records.tistory.com/127

Docker Desktop에서 제공하는 MiniKube 사용

Windows Powershell에서
```shell
kubectl cluster-info
```

kubectl 설치
https://kubernetes.io/ko/docs/tasks/tools/install-kubectl-windows/

설치 완료 확인
```shell
kubectl version
```


# 파드(Pod)란?
하나의 프로그램 실행 단위
Docker : 컨테이너
쿠버네티스 : 파드

일반적으로 하나의 파드가 하나의 컨테이너를 가짐
하나의 파드가 여러 개의 컨테이너를 가지는 경우도 있긴 함

![[Pasted image 20260318231152.png]]
2개의 결제 서버가 띄워져있다
= 2개의 결제 서버 파드(Pod)가 띄워져있다

1개의 결제 서버가 죽었다 
= 1개의 결제 서버 파드(Pod)가 죽었다

업로드 서버를 하나 띄우자 
= 업로드 서버 하나를 파드(Pod)로 띄우자

쿠버네티스도 도커처럼 이미지를 기반으로 파드를 띄워 실행함
![[Pasted image 20260318231322.png]]

