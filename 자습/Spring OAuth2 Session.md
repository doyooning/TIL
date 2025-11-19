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


