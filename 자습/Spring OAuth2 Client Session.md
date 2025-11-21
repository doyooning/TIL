#Java 
#Spring 

---
https://cafe.naver.com/xxxjjhhh/94
개발자 유미 강의 참조

https://spring.io/guides/gs/securing-web
스프링 시큐리티 공식 문서 가이드


### 동작 원리
![[Pasted image 20251120082519.png]]
###### 각각의 필터가 동작하는 주소 (관습)
**- OAuth2AuthorizationRequestRedirectFilter**

```
/oauth2/authorization/서비스명 
/oauth2/authorization/naver 
/oauth2/authorization/google
```

**-** **OAuth2LoginAuthenticationFilter : 외부 인증 서버에 설정할 redirect_uri**

```
/login/oauth2/code/서비스명 
/login/oauth2/code/naver 
/login/oauth2/code/google
```

**OAuth2 인증 및 동작을 위한 변수들**

변수 설정만 진행하면 OAuth2AuthorizationRequestRedirectFilter → OAuth2LoginAuthenticationFilter → OAuth2LoginAuthenticationProvider 까지의 과정을 추가 설정하지 않아도 자동으로 진행

따라서 사용자는 UserDetailsService와 UserDetails만 구현하면 됨

**OAuth2 소셜 로그인을 위한 변수 설정**

위의 각각의 로직이 동작을 하기 위해서는 서비스별로 특정 값이 필요하기 때문에 변수 설정이 필요함
```
#registration  
spring.security.oauth2.client.registration.서비스명.client-name=서비스명  
spring.security.oauth2.client.registration.서비스명.client-id=서비스에서 발급 받은 아이디  
spring.security.oauth2.client.registration.서비스명.client-secret=서비스에서 발급 받은 비밀번호  
spring.security.oauth2.client.registration.서비스명.redirect-uri=서비스에 등록한 우리쪽 로그인 성공 URIspring.security.oauth2.client.registration.서비스명.authorization-grant-type=authorization_code  
spring.security.oauth2.client.registration.서비스명.scope=리소스 서버에서 가져올 데이터 범위  
  
#provider  
spring.security.oauth2.client.provider.서비스명.authorization-uri=서비스 로그인 창 주소  
spring.security.oauth2.client.provider.서비스명.token-uri=토큰 발급 서버 주소  
spring.security.oauth2.client.provider.서비스명.user-info-uri=사용자 정보 획득 주소  
spring.security.oauth2.client.provider.서비스명.user-name-attribute=응답 데이터 변수
```

**registration과 provider**

registration은 외부 서비스에서 우리 서비스를 특정하기 위해 등록하는 정보여서 등록이 필수
하지만 provider의 경우 서비스별로 정해진 값이 존재하며 OAuth2 클라이언트 의존성이 유명한 서비스의 경우 내부적으로 데이터를 가지고 있음
(구글, Okta, 페이스북, 깃허브, 등등)


### 네이버 소셜 로그인
**네이버 로그인 API 신청**

네이버 개발자 센터 -> 네이버 로그인 API


**구글 OAuth2 클라이언트 신청**

### OAuth2UserService 응답 받기
성공적으로 JWT 토큰 발급하면 유저 정보를 userDetailsService로 가져와야 사용 가능
-> 응답 전달할 DTO 구현(네이버, 구글), SecurityConfig에 OAuth2UserService 등록 


