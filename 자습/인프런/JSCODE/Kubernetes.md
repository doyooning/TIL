
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
Enable Kubernetes 하면 Kubernetes를 설치해줌

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
![[Pasted image 20260328200240.png]]
Docker - 컨테이너 내부와 컨테이너 외부의 네트워크가 서로 독립적으로 분리
	컨테이너 내부 vs 컨테이너 외부
쿠버네티스 - 파드(Pod) 내부의 네트워크를 컨테이너가 공유해서 같이 사용
	파드 내부 vs 파드 외부

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
exit # 나가기
```

쿠버네티스에서는 **파드(Pod) 내부의 네트워크를 컨테이너가 공유**해서 같이 사용
때문에 파드로 접속해서 Nginx로 요청을 보냈을 때 정상적으로 응답이 날라옴

포트포워딩 활용하여 Nginx로 요청 전송
```shell
# Mac
# kubectl port-forward pod/[파드명] [로컬에서의 포트]/[파드에서의 포트]
sudo kubectl port-forward pod/nginx-pod 80:80

# Windows
kubectl port-forward pod/nginx-pod 80:80
# Sudo 사용이 제한됨
```

![[Pasted image 20260328200728.png]]
로컬에서의 포트 = 80, 파드에서의 포트 = 80

80 포트 접속 확인 
```shell 
curl localhost:80
```
Nginx 창 뜨면 성공

파드 삭제
```shell
# kubectl delete pod [파드명] 
kubectl delete pod nginx-pod # nginx-pod라는 파드 삭제
# 성공시 : pod "nginx-pod" deleted from default namespace 
kubectl get pods # 파드가 잘 삭제됐는 지 확인
# 문제 발생 : No resources found in default namespace.
```


# 백엔드(Spring Boot) 서버를 파드(Pod)로 띄워보기
기본적인 스프링부트 환경 세팅
localhost:8080에서 실행

Dockerfile 생성
: 스프링부트 프로젝트를 image로 만들기 위함
빌드한 파일(jar)을 묶어서 이미지로 만들 것임

```dockerfile
FROM eclipse-temurin:17-jdk

COPY build/libs/*SNAPSHOT.jar app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

스프링부트 프로젝트 빌드
```bash
./gradlew clean build
```

Dockerfile을 바탕으로 이미지 빌드
```shell
docker build -t spring-server .
```
끝에 .(마침표) 필수

이미지 생성 확인
```shell
docker image ls
```

manifest 파일 작성
spring-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: spring-pod
spec:
  containers:
    - name: spring-container
      image: spring-server
      ports:
        - containerPort: 8080
```

해당 파일 기반으로 파드 생성
```shell
kubectl apply -f spring-pod.yaml
```

결과 확인
```shell
kubectl get pods
```

이 때 status를 보면 ImagePullBackOff라고 떠있음
이미지 Pull 과정에서 문제가 생김 - 왜?
=> **이미지 풀 정책**

# Image Pull Policy
쿠버네티스가 yaml 파일을 읽어들여 파드 생성시, 이미지를 어떻게 받아올 것인지에 대한 정책

1. **`Always`**
    
    로컬에서 이미지를 가져오지 않고, 무조건 **레지스트리(= Dockerhub, ECR과 같은 원격 이미지 저장소)에서 가져온다.**
    
2. **`IfNotPresent`**
    
    로컬에서 이미지를 먼저 가져온다. 만약 로컬에 이미지가 없는 경우에만 레지스트리에서 가져온다.
    
3. **`Never`**
    
    로컬에서만 이미지를 가져온다.

manifest 파일에 이미지 풀 정책 추가
```yaml
imagePullPolicy: IfNotPresent # spring-server는 로컬에만 있음
```

이미지 풀 정책 기본값
- 이미지의 태그가 `latest`이거나 명시되지 않은 경우 : `imagePullPolicy`는 `Always`로 설정됨
- 이미지의 태그가 `latest`가 아닌 경우 : `imagePullPolicy`는 `IfNotPresent`로 설정됨

이미지 풀 정책을 수정하고 다시 파드 생성
```shell
kubectl delete pod spring-pod
kubectl apply -f spring-pod.yaml
kubectl get pods
```

상태가 Running

1. 파드 내부 접속 후 요청 보내기
```shell
kubectl exec -it spring-pod -- bash
curl localhost:8080
```

2. 포트포워딩
```shell
# 포트
kubectl port-forward pod/spring-pod 12345:8080 
```
로컬 12345 -> 파드 8080
localhost:12345로 접속해야 들어가짐


# 파드 디버깅
에러 메시지 확인
```shell
# kubectl describe pods [파드명]
kubectl describe pods nginx-pod # nginx-pod 파드의 세부 정보 조회
```

변경사항 적용
```shell
kubectl apply -f nginx-pod.yaml
```

파드 로그 확인
```shell
# kubectl logs [파드명]
kubectl logs nginx-pod # 파드 로그 확인하기
```

파드에 접속하고 싶을 때
```shell
# kubectl exec -it [파드명] -- bash
kubectl exec -it nginx-pod -- bash

# kubectl exec -it [파드명] -- sh
kubectl exec -it nginx-pod -- sh
```

`docker exec -it [컨테이너 ID] bash` 와 유사
bash가 안되면 sh로 접속

