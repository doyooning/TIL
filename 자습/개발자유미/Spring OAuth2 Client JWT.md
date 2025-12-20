#Java 
#Spring 
#SpringSecurity 
#OAuth

---
https://cafe.naver.com/xxxjjhhh/94
개발자 유미 강의 참조

https://spring.io/guides/gs/securing-web
스프링 시큐리티 공식 문서 가이드

**실습 목표**

OAuth2.0 클라이언트와 스프링 시큐리티 6 프레임워크를 활용하여 신뢰할 수 있는 외부 사이트(구글, 네이버)로 부터 인증을 받고 전달 받은 유저 데이터를 활용하여 JWT를 발급하고 인가를 진행하는 방법

인증 받은 데이터는 MySQL 데이터베이스를 활용하여 저장하고 관리



### OAuth2 세션 방식의 동작 원리
![[Pasted image 20251120082519.png]]
**JWT 방식에서 OAuth2 클라이언트 구성시 고민점**

​JWT 방식에서는 로그인(인증)이 성공하면 JWT 발급 문제와 웹/하이브리드/네이티브앱별 특징에 의해 OAuth2 Code Grant 방식 동작의 책임을 프론트엔드 측에 둘 것인지 백엔드 측에 둘 것인지 많은 고민을 함

**- 로그인(인증)이 성공하면 JWT를 발급해야 하는 문제**
- 프론트단에서 로그인 경로에 대한 하이퍼링크를 실행하면 소셜 로그인창이 등장하고 로그인 로직이 수행됨

- 로그인이 성공되면 JWT가 발급되는데 하이퍼링크로 실행했기 때문에 JWT를 받을 로직이 없음 
	(해당 부분에 대해 redirect_url 설정에 따라 많은 고민이 필요)

- API Client(axios, fetch)로 요청 -> 백엔드측으로 요청이 전송되지만 외부 서비스 로그인 페이지 확인 불가

**- 웹/하이브리드/네이티브앱별 특징**

- 웹에서 편하게 사용할 수 있는 웹페이지가 앱에서는 웹뷰로 보이기 때문에 UX적으로 안좋은 경험

- 앱 환경에서 쿠키 소멸 현상

위와 같은 문제로 OAuth2 Code Grant 방식 동작에 대한 redirect_url, Access 토큰 발급 문제를 어느단에서 처리해야 하는지에 대한 구현이 많고 넷상에 잘못된 구현 방법도 많이 존재

잘못된 구현 방법과 구현되어 있는 모든 방법:

#### **프론트/백 책임 분배 (웹에서)**

**- 모든 책임을 프론트가 맡음**
![[Pasted image 20251201080642.png]]
프론트단에서 (로그인 → 코드 발급 → Access 토큰 → 유저 정보 획득) 과정을 모두 수행한 뒤 백엔드단에서 (유저 정보 → JWT 발급) 방식으로 주로 네이티브앱에서 사용하는 방식.

→ 프론트에서 보낸 유저 정보의 진위 여부를 따지기 위해 추가적인 보안 로직이 필요

**- 책임을 프론트와 백엔드가 나누어 가짐** : 잘못된 방식 (대부분의 웹 블로그가 이 방식으로 구현)

**- 프론트단에서 (로그인 → 코드 발급) 후 코드를 백엔드로 전송,
  백엔드단에서 (코드 → 토큰 발급 → 유저 정보 획득 → JWT 발급)**

![[Pasted image 20251201080807.png]]

**-** **프론트단에서 (로그인 → 코드 발급 → Access 토큰) 후 Access 토큰을 백엔드로 전송,
  백엔드단에서 (Access 토큰 → 유저 정보 획득 → JWT 발급)**

![[Pasted image 20251201081306.png]]
카카오와 같은 대형 서비스 개발 포럼 및 보안 규격에서 위와 같은 코드/Access 토큰을 전송하는 방법을 지양 (하지만 토이로 구현하기 쉬워 자주 사용)

**- 모든 책임을 백엔드가 맡음***
![[Pasted image 20251201081446.png]]

프론트단에서 백엔드의 OAuth2 로그인 경로로 하이퍼링킹을 진행 후 백엔드단에서 (로그인 페이지 요청 → 코드 발급 → Access 토큰 → 유저 정보 획득 → JWT 발급) 방식으로 주로 웹앱/모바일앱 통합 환경 서버에서 사용하는 방식.

→ 백엔드에서 JWT를 발급하는 방식의 고민과 프론트측에서 받는 로직을 처리해야 한다.


**우리가 구현할 방식**

우리 채널은 백엔드에 초점
모든 책임을 백엔드에서 맡아 (로그인 페이지 요청 → 코드 발급 → Access 토큰 → 유저 정보 획득 → JWT 발급)로직을 모두 스프링 쪽에서 처리할 것

앱에 대해서는 모든 책임을 프론트가 일임하고 코드나 Access 토큰을 전달하는 행위 자체를 지양
추가적으로 다른 자료들에도 코드나 Access 토큰을 전달하는 행위를 금지중

### 동작 모식도
![[Pasted image 20251201083105.png]]
**OAuth2 소셜 로그인을 위한 변수 설정**
application.properties -> registration, provider 등록

**JWT Session과 동일한 과정 진행:**
소셜 로그인 구현(CustomOAuth2UserService 구현)
DB 연결, UserEntity와 UserRepository 구현
Service 계층에 User 정보 DB 저장 로직 추가


**JWT 발급과 검증**
- 로그인시 → 성공 → JWT 발급
- 접근시 → JWT 검증


**JWT 생성 원리**
[JWT.IO 공식 사이트 바로가기](https://jwt.io/)
JWT는 Header.Payload.Signature 구조로 이루어져 있다. 각 요소는 다음 기능을 수행한다.

**- Header**
- JWT임을 명시
- 사용된 암호화 알고리즘

**- Payload**
- 정보

**- Signature**
- 암호화알고리즘((BASE64(Header))+(BASE64(Payload)) + 암호화키)

JWT의 특징은 내부 정보를 단순 BASE64 방식으로 인코딩하기 때문에 외부에서 쉽게 디코딩 가능

외부에서 열람해도 되는 정보를 담아야 하며, 토큰 자체의 발급처를 확인하기 위해서 사용함
(지폐와 같이 외부에서 그 금액을 확인하고 금방 외형을 따라서 만들 수 있지만, 발급처에 대한 보장 및 검증은 확실하게 해야하는 경우에 사용하기에 토큰 내부에 비밀번호와 같은 값 입력 금지)

**JWT 암호화 방식**

**암호화 종류**

-양방향 
- 대칭키 - 이 프로젝트는 양방향 대칭키 방식 사용 : HS256
- 비대칭키

-단방향 

**로그인 성공**
OAuth2 로그인이 성공하면 실행될 성공 핸들러를 커스텀해서 내부에 JWT 발급 구현

**JWT 검증 필터**
스프링 시큐리티 filter chain에 요청에 담긴 JWT를 검증하기 위한 커스텀 필터를 등록해야 함

해당 필터를 통해 요청 쿠키에 JWT가 존재하는 경우 JWT를 검증하고 강제로 SecurityContextHolder에 세션을 생성 (이 세션은 STATLESS 상태로 관리되기 때문에 해당 요청이 끝나면 소멸됨)


**CORS 설정**
- SecurityConfig
- config > CorsMvcConfig

**소셜 로그인 요청**
![[Pasted image 20251202203939.png]]
강의 : React
나 : Vue.js

프론트엔드 구현을 위해 Vue 프로젝트 생성

App.vue
```vue
<script setup>  
import axios from "axios";  
  
const onNaverLogin = () => {  
  window.location.href = "http://localhost:8080/oauth2/authorization/naver"  
}  
  
axios  
    .get("http://localhost:8080/my", {withCredentials: true})  
    .then((res) => {  
      alert(JSON.stringify(res.data))  
    })  
    .catch((error) => alert(error))  
</script>  
  
<template>  
    <h1>Login</h1>  
    <button @click="onNaverLogin">naver login</button>  
</template>
```

백엔드에서 localhost:3000 -> localhost:5173으로 변경 !! (Vue) 
실행하면 처음에 axios는 에러가 나는데(Network Error)
로그인 버튼 누르면 네이버 로그인 잘 진행되는 것을 확인


**무한 루프 오류가 발생할 경우**

필터 위치에 따라 OAuth2 인증을 진행하는 필터보다 JWTFilter가 앞에 존재하는 경우 아래와 같은 오류가 발생할 수 있음

1. 재로그인
2. JWT 만료 → 거절
3. OAuth2 로그인 실패 → 재요청
4. 무한 루프

따라서 JWTFilter를 OAuth2LoginAuthenticationFilter 뒤에 위치 시키거나 JWTFilter 내부에 if문을 통해 특정 경로 요청은 넘어가도록 진행하면 됨

```
io.jsonwebtoken.ExpiredJwtException: JWT expired 253932 milliseconds ago at 2025-12-16T11:33:08.000Z. Current time: 2025-12-16T11:37:21.932Z. Allowed clock skew: 0 milliseconds.
...
```
