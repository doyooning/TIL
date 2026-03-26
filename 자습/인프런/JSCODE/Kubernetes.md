
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

# 웹 서버(Nginx)를 파드(Pod)로 띄우기
yaml 파일로 많이 생성함

yaml 파일 생성: nginx-pod.yaml
```yaml
apiVersion: v1 # Pod를 생성할 때는 v1이라고 기재한다. (공식 문서)
kind: Pod # Pod를 생성한다고 명시
metadata:
 name: nginx-pod # Pod에 이름 붙이는 기능
spec:
 containers:
 - name: nginx-container # 생성할 컨테이너의 이름
 image: nginx # 컨테이너를 생성할 때 사용할 Docker 이미지
 ports:
 - containerPort: 80 # 해당 컨테이너가 어떤 포트를 사용하는 지 명시적으로 표현
```
yaml 문법은 들여쓰기 할 때 tab 말고 띄어쓰기로
`spec.containers.ports.containerPort` : 실제 작동에는 영향을 미치지 않음
단순히 컨테이너가 어떤 포트를 사용하는 지 명시적으로 나타내 기 위한 문서화용 (Dockerfile의 EXPOSE 와 비슷한 역할)

yaml 파일 기반 파드 생성
```shell
kubectl apply -f nginx-pod.yaml # yaml 파일에 적혀져있는 리소스(파드)를 생성
```

파드 잘 생성되었는지 확인
```shell
kubectl get pods # 파드(Pod) 조회
```

Nginx 접속 확인
-> 접속 안 됨

쿠버네티스에서는 위에서 작성한 yaml 파일을 보고 **매니페스트 파일(Manifest File)** 이라고 부름
이 매니페스트 파일은 쿠버네티스에서 다양한 리소스(파드, 서비스, 볼륨 등)를 생성하고 관리하기 위해 사용하는 파일이라고 기억하면 좋음

# 왜 접속 안됨?
Docker - 컨테이너 내부와 컨테이너 외부의 네트워크가 서로 독립적으로 분리
쿠버네티스 - 파드(Pod) 내부의 네트워크를 컨테이너가 공유해서 같이 사용

**파드(Pod)의 네트워크**는 로컬 컴퓨터의 네트워크와는 **독립적으로 분리**됨
이 때문에 파드(Pod)로 띄운 Nginx에 아무리 요청을 보내도 응답이 없던 것 !

Nginx가 띄우는 웹 페이지에 접근하기(2가지 방법) 
1. 파드(Pod) 내부로 들어가서 접근하기 
2. 파드(Pod)의 내부 네트워크를 외부에서도 접속할 수 있도록 포트 포워딩(= 포트 연결시키기) 활용하기


# 파드 내부로 들어가서 Nginx 요청 보내기
```shell
# kubectl exec -it [파드명] -- bash
# 도커에서 컨테이너로 접속하는 명령어(docker exec -it [컨테이너 ID] bash)와 비슷하다.
kubectl exec -it nginx-pod -- bash # nginx-pod 내부 환경으로 접속
# ---Pod 내부---
curl localhost:80 # Nginx로 요청보내기
```

쿠버네티스에서는 **파드(Pod) 내부의 네트워크를 컨테이너가 공유**해서 같이 사용
때문에 파드로 접속해서 Nginx로 요청을 보냈을 때 정상적으로 응답이 날라옴

포트포워딩 활용하여 Nginx로 요청 전송
```shell
# kubectl port-forward pod/[파드명] [로컬에서의 포트]/[파드에서의 포트]
sudo kubectl port-forward pod/nginx-pod 80:80
```

파드 삭제
```shell
# kubectl delete pod [파드명] 
kubectl delete pod nginx-pod # nginx-pod라는 파드 삭제 
kubectl get pods # 파드가 잘 삭제됐는 지 확인
```


