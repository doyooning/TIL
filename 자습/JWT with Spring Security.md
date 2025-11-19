#Java 
#Spring 

---
https://cafe.naver.com/xxxjjhhh/94
개발자 유미 강의 참조

https://spring.io/guides/gs/securing-web
스프링 시큐리티 공식 문서 가이드


### JWT 인증 방식 Spring Security 동작 원리

![[Pasted image 20251117192705.png]]
**- 회원가입** : 내부 회원 가입 로직은 세션 방식과 JWT 방식의 차이가 없다.
**- 로그인 (인증)** : 로그인 요청을 받은 후 세션 방식은 서버 세션이 유저 정보를 저장하지만 JWT 방식은 토큰을 생성하여 응답한다.
**- 경로 접근 (인가)** : JWT Filter를 통해 요청의 헤더에서 JWT를 찾아 검증을 하고 일시적 요청에 대한 Session을 생성한다. (생성된 세션은 요청이 끝나면 소멸됨)

JWT를 통한 인증/인가를 위해서 세션을 STATELESS 상태로 설정하는 것이 중요하다.


### Spring Security 필터 동작 원리
스프링 시큐리티는 클라이언트의 요청이 여러개의 필터를 거쳐 DispatcherServlet(Controller)으로 향하는 중간 필터에서 요청을 가로챈 후 검증(인증/인가)을 진행한다.

**- Delegating Filter Proxy**
서블릿 컨테이너 (톰캣)에 존재하는 필터 체인에 DelegatingFilter를 등록한 뒤 모든 요청을 가로챈다.
​
**- 서블릿 필터 체인의 DelegatingFilter → Security 필터 체인 (내부 처리 후) → 서블릿 필터 체인의 DelegatingFilter**
가로챈 요청은 SecurityFilterChain에서 처리 후 상황에 따른 거부, 리디렉션, 서블릿으로 요청 전달을 진행한다.

![[Pasted image 20251117201306.png]]
(왼쪽은 Servlet Filter, 오른쪽은 Security Filter)

**Form 로그인 방식에서 UsernamePasswordAuthenticationFilter**

Form 로그인 방식에서는 클라이언트단이 username과 password를 전송한 뒤 Security 필터를 통과하는데 UsernamePasswordAuthentication 필터에서 회원 검증을 진행을 시작한다.

(회원 검증의 경우 UsernamePasswordAuthenticationFilter가 호출한 AuthenticationManager를 통해 진행하며 DB에서 조회한 데이터를 UserDetailsService를 통해 받음)

우리의 JWT 프로젝트는 SecurityConfig에서 formLogin 방식을 disable 했기 때문에 기본적으로 활성화 되어 있는 해당 필터는 동작하지 않는다.

따라서 로그인을 진행하기 위해서 필터를 커스텀하여 등록해야 한다.

```java
public class LoginFilter extends UsernamePasswordAuthenticationFilter {  
  
    private final AuthenticationManager authenticationManager;  
  
    public LoginFilter(AuthenticationManager authenticationManager) {  
        this.authenticationManager = authenticationManager;  
    }  
  
    @Override  
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {  
  
        //클라이언트 요청에서 username, password 추출  
        String username = obtainUsername(request);  
        String password = obtainPassword(request);  
  
        //스프링 시큐리티에서 username과 password를 검증하기 위해서는 token에 담아야 함  
        UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(username, password, null);  
  
        //token에 담은 검증을 위한 AuthenticationManager로 전달  
        return authenticationManager.authenticate(authToken);  
    }  
  
    //로그인 성공시 실행하는 메소드 (여기서 JWT를 발급하면 됨)  
    @Override  
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) {  
  
    }  
  
    //로그인 실패시 실행하는 메소드  
    @Override  
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) {  
  
    }  
}
```

```java
http  
        .addFilterAt(new LoginFilter(authenticationManager(authenticationConfiguration)), UsernamePasswordAuthenticationFilter.class);
```
필터 추가, LoginFilter()는 인자를 받음 
(AuthenticationManager() 메소드에 authenticationConfiguration 객체를 넣어야 함) 따라서 등록 필요 

**로그인 성공시 JWT 반환**
**​**
로그인 성공시 successfulAuthentication() 메소드를 통해 JWT를 응답해야 한다. 따라서 JWT 응답 구문을 작성해야 하는데 JWT 발급 클래스를 아직 생성하지 않음


### DB 기반 로그인 검증

**UserDetailsService 커스텀 구현**

**- CustomUserDetailsService**
```java
@Service  
public class CustomUserDetailsService implements UserDetailsService {  
  
    private final UserRepository userRepository;  
  
    public CustomUserDetailsService(UserRepository userRepository) {  
        this.userRepository = userRepository;  
    }  
  
    @Override  
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {  
  
        //DB에서 조회  
        UserEntity userData = userRepository.findByUsername(username);  
  
        if (userData != null) {  
  
            //UserDetails에 담아서 return하면 AuthenticationManager가 검증 함  
            return new CustomUserDetails(userData);  
        }  
  
        return null;  
    }  
}
```

**UserDetails 커스텀 구현**

**- CustomUserDetails**
```java
public class CustomUserDetails implements UserDetails {  
  
    private final UserEntity userEntity;  
  
    public CustomUserDetails(UserEntity userEntity) {  
        this.userEntity = userEntity;  
    }  
  
    @Override  
    public Collection<? extends GrantedAuthority> getAuthorities() {  
        Collection<GrantedAuthority> collection = new ArrayList<>();  
        collection.add(new GrantedAuthority() {  
  
            @Override  
            public String getAuthority() {  
                return userEntity.getRole();  
            }  
        });  
  
        return collection;  
    }  
  
    @Override  
    public String getPassword() {  
        return userEntity.getPassword();  
    }  
  
    @Override  
    public String getUsername() {  
        return userEntity.getUsername();  
    }  
  
    @Override  
    public boolean isAccountNonExpired() {  
        return true;  
    }  
  
    @Override  
    public boolean isAccountNonLocked() {  
        return true;  
    }  
  
    @Override  
    public boolean isCredentialsNonExpired() {  
        return true;  
    }  
  
    @Override  
    public boolean isEnabled() {  
        return true;  
    }  
}
```
CustomUserDetails는 사용자 정보를 주고받을 DTO같은 느낌
대부분의 메서드가 Override라 정형화되어있음


### JWT 발급과 검증

**JWT 발급과 검증**
- 로그인시 → 성공 → JWT 발급
- 접근시 → JWT 검증

**JWT 생성 원리**
[https://www.jwt.io/](https://www.jwt.io/)

JWT는 Header.Payload.Signature 구조로 이루어져 있다. 각 요소는 다음 기능을 수행한다.

**- Header**
- JWT임을 명시
- 사용된 암호화 알고리즘
**- Payload**
- 정보
**- Signature**
- 암호화알고리즘((BASE64(Header))+(BASE64(Payload)) + 암호화키)

JWT의 특징은 내부 정보를 단순 BASE64 방식으로 인코딩하기 때문에 외부에서 쉽게 디코딩 할 수 있다.
외부에서 열람해도 되는 정보를 담아야하며, 토큰 자체의 발급처를 확인하기 위해서 사용한다.

(지폐와 같이 외부에서 그 금액을 확인하고 금방 외형을 따라서 만들 수 있지만 발급처에 대한 보장 및 검증은 확실하게 해야하는 경우에 사용한다. 따라서 토큰 내부에 비밀번호와 같은 값 입력 금지)

**JWT 암호화 방식**
**- 암호화 종류**
- 양방향
	대칭키 - 이 프로젝트는 양방향 대칭키 방식 사용 : HS256
	비대칭키
- 단방향

**암호화 키 저장**
암호화 키는 하드코딩 방식으로 구현 내부에 탑재하는 것을 지양하기 때문에 변수 설정 파일에 저장한다.
-> application.properties

###### JWTUtil
**- 토큰 Payload에 저장될 정보**
- username
- role
- 생성일
- 만료일

**- JWTUtil 구현 메소드**
- JWTUtil 생성자
- username 확인 메소드
- role 확인 메소드
- 만료일 확인 메소드
​
LoginFilter, SecurityConfig에 JWTUtil를 주입해서 사용
-> private final로 주입하고 생성자에도 추가

로그인 성공시:
```java
String token = jwtUtil.createJwt(username, role, 60*60*10L); response.addHeader("Authorization", "Bearer " + token);
```
토큰에 사용자명, 권한, 만료기간이 포함됨
응답헤더에는 토큰 정보가 들어감

HTTP 인증 방식은 RFC 7235 정의에 따라 아래 인증 헤더 형태를 가져야 한다.
​Authorization: 타입 인증토큰 
(예시 -  Authorization: Bearer 인증토큰string)

로그인 실패시:
```java
response.setStatus(401);
```
401 반환

**발급 테스트**
POSTMAN으로 /login 경로로 username과 password를 포함한 POST 요청을 보낸 후 응답 헤더에서 Authorization 키에 담긴 JWT를 확인한다.


###### JWT 검증 필터
로그인 후 메인 페이지는 접근이 가능(별도의 토큰 검증 필요X)
하지만 /admin같은 경우는 403 오류 반환(토큰은 있는데 검증이 안 됨)
=> 토큰 검증 필터 필요

**JWT 검증 필터**
스프링 시큐리티 filter chain에 요청에 담긴 JWT를 검증하기 위한 커스텀 필터를 등록해야 한다.

해당 필터를 통해 요청 헤더 Authorization 키에 JWT가 존재하는 경우 JWT를 검증하고 강제로SecurityContextHolder에 세션을 생성한다.
(이 세션은 STATLESS 상태로 관리되기 때문에 해당 요청이 끝나면 소멸 된다.)

JWTFilter를 OncePerRequestFilter를 상속받아 구현
(요청당 한번만 실행하는 필터)

**검증**
헤더 검증 : 헤더의 Authorization이 null이 아니고 접두사가 "Bearer "이면 성공
실패시 검증 메서드 종료

토큰 소멸 시간 검증 : 접두사 제거한 토큰을 isExpired 메서드에 넘겨서 확인
실패시 검증 메서드 종료

```java
//userEntity를 생성하여 값 set 
UserEntity userEntity = new UserEntity(); 
userEntity.setUsername(username); 
userEntity.setPassword("temppassword"); 
userEntity.setRole(role);
```
토큰에서 username, role 얻어서 값 세팅
비밀번호는 토큰에 없는데 userEntity에는 비밀번호를 세팅해주어야 하므로 임의의 값을 넣어준다.
(비번을 DB에서 조회를 하게 되면 매번 요청이 올 때마다 DB조회를 해야 하는 상황이 발생)

**SecurityConfig JWTFilter 등록**
```java
http.addFilterBefore(new JWTFilter(jwtUtil), LoginFilter.class);
```
LoginFilter 앞에 등록해준다.

**JWT 요청 인가 테스트**
POSTMAN으로 로그인 먼저 진행, admin 토큰을 얻음
GET 방식으로 /admin 접근할 때 요청헤더의 Authorization에 토큰을 넣고 접근
-> 200 OK

### (정보) JWT 기반 Stateless 인증의 장점
- **서버 확장에 유리함 (Scalability)**
    - 서버가 사용자 상태를 저장하지 않기 때문에, 새로운 서버 인스턴스를 추가해도 별도 세션 공유 작업 없이 바로 서비스 가능
    - 마이크로서비스나 분산 시스템 환경에서 특히 효과적
    
- **세션 저장소 불필요**
    - 기존의 세션 기반 인증은 서버나 DB에 사용자 상태를 저장해야 하지만, JWT는 토큰 자체에 사용자 정보를 담고 있어 별도 저장소가 필요 없음
    
- **빠른 인증 처리**
    - JWT는 클라이언트가 요청 시 토큰을 함께 보내고, 서버는 공개키로 로컬에서 유효성만 검증하면 되므로 속도가 빠름
    
- **자격 증명 노출 최소화**
    - 로그인 시 한 번만 아이디/비밀번호를 보내고 이후에는 JWT만 전송하므로 네트워크 상에서 민감 정보 노출이 줄어들게 됨
    
- **토큰 만료로 보안 강화**
    - JWT는 만료 시간(`exp`)을 설정할 수 있어, 토큰이 탈취되더라도 일정 시간이 지나면 자동으로 무효화 처리
    
- **Self-contained 구조**
    - 사용자 정보(예: 권한, 발급일시 등)가 토큰 내부에 포함되어 있어, 별도 조회 없이 인증 및 인가 가능

#### ⚠️ 주의할 점
- **토큰 무효화 어려움**
	- 서버가 상태를 저장하지 않기 때문에, 이미 발급된 JWT를 서버 측에서 임의로 무효화하기 어려움
	- 이를 보완하려면 Refresh Token 전략이나 블랙리스트 관리가 필요

- **보안 설정 필수**
	- 토큰 저장 위치(예: 쿠키 vs 로컬스토리지)와 쿠키 옵션(SameSite, HttpOnly, Secure 등)을 잘 설정해야 XSS/CSRF 공격 방지 가능

