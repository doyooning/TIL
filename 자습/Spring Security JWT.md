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

