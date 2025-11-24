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

### 유저 정보 저장

유저 정보 DB저장 모식도 
![[Pasted image 20251123233032.png]]
MySQL 데이터베이스 셋업 해주고 Driver 의존성 다시 On
이후 UserRepository와 UserEntity 구현

**CustomOAuth2UserService 유저 정보 DB 저장 로직 작성**

service > CustomOAuth2UserService
```java
@Service 
public class CustomOAuth2UserService extends DefaultOAuth2UserService {
	...
	private final UserRepository userRepository; 
	public CustomOAuth2UserService(UserRepository userRepository) { 
		this.userRepository = userRepository; 
	}
	...
	String username = oAuth2Response.getProvider()+" "+oAuth2Response.getProviderId();
	UserEntity existData = userRepository.findByUsername(username);
	
	String role = "ROLE_USER"; 
	if (existData == null) { 
		UserEntity userEntity = new UserEntity(); 
		userEntity.setUsername(username);
		userEntity.setEmail(oAuth2Response.getEmail()); 
		userEntity.setRole(role); 
		userRepository.save(userEntity); 
	} else { 
		existData.setUsername(username); 
		existData.setEmail(oAuth2Response.getEmail()); 
		role = existData.getRole(); 
		userRepository.save(existData); 
	}
	...
```
Spring Security때처럼 유저 정보 저장 로직을 Service에 추가

### 로그인 페이지 구현

로그인 페이지 구현 및 로그인 컨트롤러 구현
`<a href="/oauth2/authorization/google">google login</a>` 링크로 연결

**SecurityConfig OAuth2 커스텀 로그인 페이지 등록**

- config > SecurityConfig
```java
http.oauth2Login((oauth2) -> oauth2 
	.loginPage("/login") 
	.userInfoEndpoint((userInfoEndpointConfig) -> 
	userInfoEndpointConfig.userService(customOAuth2UserService)));
```


### Client Registration

**OAuth2 서비스 변수 등록 방법**

기존에 application.properties 변수 설정 파일에서 설정했던 소셜 로그인 제공 서비스에 대한 정보 기입을 관련 클래스를 통해 직접 진행하는 방법

앞으로 커스텀을 진행하기 위해 클래스를 직접 구현하는 것이 필수적임

**- ClientRegistration**

서비스별 OAuth2 클라이언트의 등록 정보를 가지는 클래스

**- ClientRegistrationRepository**

ClientRegistration의 저장소로 서비스별 ClientRegistration들을 가짐


### OAuth2AuthorizationRequestRedirectFilter
**로그인 페이지에서 : /oauth2/authorization/서비스**

로그인 페이지에서 `GET : /oauth2/authorization/서비스 경로`로 요청을 할 경우 OAuth2 의존성에 의해 `OAuth2AuthorizationRequestRedirectFilter`에서 해당 요청을 받고 내부 프로세서를 진행

`OAuth2AuthorizationRequestRedirectFilter`
: 요청을 받은 후 해당하는 서비스의 로그인 URI로 요청을 리디렉션 시킴
이때 서비스의 정보는 지난 시간 등록한 `ClientRegistrationRepository`에서 가져옴


### OAuth2LoginAuthenticationFilter
**로그인 성공 redirect_uri 필터 : /login/oauth2/code/서비스명**

`OAuth2LoginAuthenticationFilter`
: 인증 서버에서 로그인을 성공한 뒤 우리 서버측으로 발급되는 CODE를 획득하고, 
CODE를 통해 Access 토큰과 User 정보를 획득하는 `OAuth2LoginAuthenticationProvider`를 호출하는 일련의 과정을 시작하는 필터

**OAuth2LoginAuthenticationFilter**

`OAuth2LoginAuthenticationFilter`
: `/login/oauth2/code/서비스명` 경로에 대해서 CODE를 받는 필터
이후 검증을 진행하고 완료되면 `OAuth2LoginAuthenticationProvider`를 호출

**OAuth2LoginAuthenticationProvider**

`OAuth2LoginAuthenticationFilter`로부터 호출 받아 전달 받은 정보를 통해 외부 인증 서버를 호출하여 Access 토큰을 발급 받음

이후 Access 토큰을 통해 외부 리소스 서버에서 유저 정보를 `OAuth2UserService`로 받음


### OAuth2AuthorizedClientService
**OAuth2AuthorizedClientService**

OAuth2 소셜 로그인을 진행하는 사용자에 대해 우리의 서버는 인증 서버에서 발급 받은 Access 토큰과 같은 정보를 담을 저장소가 필요함

기본적으로 인메모리 방식으로 관리되는데 소셜 로그인 사용자 수가 증가하고 서버의 스케일 아웃 문제로 인해 인메모리 방식은 실무에서 사용하지 않음

따라서 DB에 해당 정보를 저장하기 위해서는 `OAuth2AuthorizedClientService`를 직접 작성해야 함

**OAuth2AuthorizedClientService 클래스 생성**

- oauth2 > CustomOAuth2AuthorizedClientService
- JDBC 의존성 추가 후 진행
```java
@Configuration 
public class CustomOAuth2AuthorizedClientService { 
	public OAuth2AuthorizedClientService oAuth2AuthorizedClientService(JdbcTemplate jdbcTemplate, ClientRegistrationRepository clientRegistrationRepository) { 
	return new JdbcOAuth2AuthorizedClientService(jdbcTemplate ,clientRegistrationRepository); 
	} 
}
```
return new를 통해 Jdbc 방식 JdbcOAuth2AuthorizedClientService() 클래스를 응답

**SecurityConfig 등록**
```java
@Configuration 
@EnableWebSecurity 
public class SecurityConfig { 
	private final CustomOAuth2UserService customOAuth2UserService; 
	private final CustomClientRegistrationRepo customClientRegistrationRepo; 
	private final CustomOAuth2AuthorizedClientService customOAuth2AuthorizedClientService; 
	private final JdbcTemplate jdbcTemplate; 
public SecurityConfig(CustomOAuth2UserService customOAuth2UserService, CustomClientRegistrationRepo customClientRegistrationRepo, CustomOAuth2AuthorizedClientService customOAuth2AuthorizedClientService, 
JdbcTemplate jdbcTemplate) 
	{ this.customOAuth2UserService = customOAuth2UserService; this.customClientRegistrationRepo = customClientRegistrationRepo; this.customOAuth2AuthorizedClientService = customOAuth2AuthorizedClientService; this.jdbcTemplate = jdbcTemplate; 
	}
```
생성자 주입 추가해줌
임의로 커스텀한 필터들은 다 SecurityConfig에 등록해줘야 작동 가능

```java
http.oauth2Login((oauth2) -> oauth2 
.loginPage("/login") .clientRegistrationRepository(customClientRegistrationRepo.clientRegistrationRepository()) .authorizedClientService(customOAuth2AuthorizedClientService.oAuth2AuthorizedClientService(jdbcTemplate, customClientRegistrationRepo.clientRegistrationRepository())) .userInfoEndpoint((userInfoEndpointConfig) -> userInfoEndpointConfig.userService(customOAuth2UserService)));
```
로그인 -> `clientRegistrationRepository` -> `authorizedClientService`(승인되었는지?)
-> `userInfoEndpoint`(받은 유저 정보 다루기)


**데이터베이스에 해당 테이블 생성**

우리가 사용할 MySQL 데이터베이스에 `OAuth2AuthorizedClientService`가 사용할 테이블이 존재해야 함
```SQL
CREATE TABLE oauth2_authorized_client ( 
	client_registration_id varchar(100) NOT NULL, 
	principal_name varchar(200) NOT NULL, 
	access_token_type varchar(100) NOT NULL, 
	access_token_value blob NOT NULL, 
	access_token_issued_at timestamp NOT NULL, 
	access_token_expires_at timestamp NOT NULL, 
	access_token_scopes varchar(1000) DEFAULT NULL, 
	refresh_token_value blob DEFAULT NULL, 
	refresh_token_issued_at timestamp DEFAULT NULL, 
	created_at timestamp DEFAULT CURRENT_TIMESTAMP NOT NULL, 
	PRIMARY KEY (client_registration_id, principal_name) 
);
```

​


---

**의문사항**

(client_registration_id, principal_name)가 겹치는 경우가 존재하면 테이블에 값이 오버 라이팅되는 현상이 발생하는데 운영 환경에서 발생하는 문제가 없을지 의문점이 든다.

**- 예시**
user : username / client_registration_id / principal_name user1 : bbbbbbbb / naver / 개발자유미 user2 : aaaaaaaaa / naver / 개발자유미

위와 같이 client_registration_id와 principal_name이 겹칠 경우 새로운 row가 생성되지 않고 기존 데이터 위에 덮어 씌어짐

만약 해당 경우가 동시에 로그인을 진행한다면 로그인이 실패할 수 있을거 같다. (Access 토큰을 발급 받고 추후에 사용하지는 않지만, 동시에 로그인을 진행한다면 오류가 발생할 수 있지 않을까.)

따라서 스프링 시큐리티 공식 깃허브에 Issue로 질문을 했고, 개발자측에서 OAuth2AuthorizedClientService를 알아서 구현해서 사용하라고 답변이 왔습니다.

