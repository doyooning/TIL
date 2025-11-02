#Java 
#Spring 

---
https://cafe.naver.com/xxxjjhhh/94
개발자 유미 강의 참조

https://spring.io/guides/gs/securing-web
스프링 시큐리티 공식 문서 가이드


### Spring Security 시작하기
Spring Initializr(=Spring Boot) 프로젝트 생성

의존성 추가 : Spring Web, Lombok, Mustache, Spring Security, Spring Data JPA, MySQL Driver
(Mustache는 템플릿 엔진 역할)

###### Thymeleaf나 Mustache같은 템플릿 엔진 사용 이유?
⚠️ 템플릿 엔진 없이 HTML만 사용할 때 주의할 점

- **동적 데이터 바인딩 불가**: 서버에서 사용자 이름, 권한 등 동적 삽입 데이터를 HTML에 직접 넣을 수 없음
- **보안 관련 처리 어려움**: Spring Security의 `sec:authorize` 같은 태그를 사용하려면 Thymeleaf가 필요함
- **폼 처리 불편**: 서버에서 폼 데이터를 처리하고 다시 보여주는 작업이 번거로움

✨ 템플릿 엔진을 쓰면 좋은 점

- Thymeleaf / Mustache 사용 시
	동적 데이터 삽입 : 가능(`th:text`, `{{name}}`)
	인증/권한 기반 뷰 제어 : 가능(`sec:authorize`)
	폼 처리 : 편리(`th:field`, `th:object`)
	유지보수 : 템플릿 구조로 관리 용이

- HTML만 사용할 때
	동적 데이터 삽입 : 불가능
	인증/권한 기반 뷰 제어 : 불가능
	폼 처리 : 수동 처리 필요
	유지보수 : 반복되는 HTML 수동 관리


controller / MainController.java 생성
@Controller 해주고 안에 @GetMapping으로 메서드 아무거나 만들어줌
```
@Controller  
public class MainController {  
  
    @GetMapping("/")  
    public String mainP() {  
        return "main";  
    }  
}
```

resources / templates / main.mustache 생성
html 기본 구조 만들어줌

처음 어플리케이션 실행하면 로그창에 임시 비밀번호 발급해줌
localhost:8080/ 접속하면 로그인창 하나 뜸
ID : user / PW : 로그창에 발급해준 비밀번호 복붙
-> main.mustache로 정상 연결


### Security Config


![](https://cafeptthumb-phinf.pstatic.net/MjAyNTA3MzBfNDgg/MDAxNzUzODE5NDIxOTkx.Q6sDRyh2AMLqf5_6L-GctUuvgvq9RHxtrHH5eGD8Ec8g.qBu1K1RViMaO9KQpBJUSyN1ApQh6Oz3xhefddIieqZAg.PNG/image-34d1339d-51fa-4c6a-aa13-394edf5d47bc.png?type=w1600)

​**인가(Authentication)**

특정한 경로에 요청이 오면 Controller 클래스에 도달하기 전 필터에서 Spring Security가 검증을 함

1. 해당 경로의 접근은 누구에게 열려 있는지
2. 로그인이 완료된 사용자인지
3. 해당되는 role을 가지고 있는지

**Security Configuration**

인가 설정을 진행하는 클래스
(엄밀하게 정의하면 SecurityFilterChain 설정을 진행함)

Securiy Config 클래스 기본 틀
```
@Configuration  
@EnableWebSecurity  
public class SecurityConfig {  
    @Bean  
    public SecurityFilterChain filterChain (HttpSecurity http) throws Exception {  
        http.authorizeHttpRequests( 
                (auth) -> auth  
                        .requestMatchers("/", "/login").permitAll()  
                        .requestMatchers("/admin").hasRole("ADMIN")  
                        .requestMatchers("/my/**").hasAnyRole("USER", "ADMIN")  
                        .anyRequest().authenticated()  
        );  
        return http.build();  
    }  
}
```

==@Configuration== + ==@EnableWebSecurity== -> Spring이 Security Config 클래스로 인식함
내부에 메서드는 SecurityFilterChain 타입을 반환

로직은 정형화된 구조라서 기본 틀 그대로 참조해서 역할, 권한 범위에 따라 수정해주면 됨

상단부터 순차 실행 
-> 동일한 요청을 맨 위에서 permitAll 하면 밑에는 소용없다 

`.requestMatchers("/", "/login").permitAll()`
: 메인 페이지, 로그인 페이지는 모두에게

`.anyRequest().authenticated()`
: 위 요청 외에는 인가된 사용자만

### Login 페이지 생성

**Config 설정 후 로그인 페이지**

스프링 시큐리티 Config 클래스 설정 후 특정 경로에 대한 접근 권한이 없는 경우,
자동으로 로그인 페이지로 리다이렉팅 되지 않고 오류 페이지가 발생한다.

위 문제를 해결하기 위해 Config 클래스를 설정하면 로그인 페이지 설정도 진행해야 한다.
```
<form action="/loginProc" method="post" name="loginForm">  
    <input id="username" type="text" name="username" placeholder="id"/>  
    <input id="password" type="password" name="password" placeholder="password"/>  
    <input type="submit" value="login"/>  
</form>
```
로그인 폼을 만들어준다.

Config에 로그인 페이지에 대한 설정 진행
```
http.formLogin((auth) -> auth.loginPage("/login") // 로그인 페이지를 연결해줌  
                .loginProcessingUrl("/loginProc") // 로그인이 완료되면 연결할 페이지  
                .permitAll() // 로그인되면 permitAll 상태  

http.csrf((auth) -> auth.disable()); // CSRF는 잠시 비활성화
);
```


### Security 암호화
스프링 시큐리티는 사용자 인증(로그인)시 비밀번호에 대해 단방향 해시 암호화를 진행,
저장되어 있는 비밀번호와 대조한다.

따라서 회원가입시 비밀번호 항목에 대해서 암호화를 진행해야 한다.

스프링 시큐리티는 암호화를 위해 BCrypt Password Encoder를 제공하고 권장한다. 따라서 해당 클래스를 return하는 메소드를 만들어 @Bean으로 등록하여 사용하면 된다.

###### 단방향 해시 암호화

**- 양방향**
- 대칭키
- 비대칭키
**- 단방향**
- 해시

SecurifyConfig.java에 추가
```
...
@Bean  
public BCryptPasswordEncoder bCryptPasswordEncoder() {  
    return new BCryptPasswordEncoder();  
}
...
```
앞으로 작업할 때 비밀번호는 계속 암호화 필요
어디서든 패스워드 인코더 메서드를 호출하면 된다.

### DB 종료와 ORM

**데이터베이스 종류와 ORM**

회원 정보를 저장하기 위한 데이터베이스는 MySQL 엔진의 데이터베이스를 사용한다. 
DB 접근은 Spring Data JPA를 사용한다.

![](https://cafeptthumb-phinf.pstatic.net/MjAyNTA3MzBfODYg/MDAxNzUzODIxMDYwOTc2.WwS5iPVKzmIN8RUOROz-cyRAF6jLleLCRfpblT-AyHkg.ImTc6pU0TQormIhRwqQ2WO51x2BQKWG9YaOBMfk68SIg.PNG/image-0eafa548-c244-4fc8-a9a2-e5c17f0954e8.png?type=w1600)
DB 접근을 위해 build.gradle에서 다시 의존성 2개(spring data랑 mysql connector)해제 필요
join = 회원가입

### DB 연결
application.properties에 datasource 정보 넣어준다.

### 회원가입(join) 구현

회원가입 페이지 생성(join.mustache)
```
...
<form action="/joinProc" method="post" name="joinForm">  
    <input type="text" name="username" placeholder="Username"/>  
    <input type="password" name="password" placeholder="Password"/>  
    <input type="submit" value="Join"/>  
</form>
...
```

JoinDTO 객체 생성
간단하게 회원가입시 입력받을 데이터(ID, PW) 정의
```
@Setter  
@Getter  
public class JoinDTO {  
    private String username;  
    private String password;  
}
```

JoinController 생성
```
...
@Autowired  
private JoinService joinService;  
// 지금은 간편하게 필드 주입 방식, 생성자 방식으로 하는 걸 권장  
  
@GetMapping("/join")  
public String joinP() {  
  
    return "join";  
}  
  
  
@PostMapping("/joinProc")  
public String joinProcess(JoinDTO joinDTO) {  
  
    System.out.println(joinDTO.getUsername());  
  
    joinService.joinProcess(joinDTO);  
  
    return "redirect:/login";  
}
```
GET으로 /join 
-> join페이지, 폼 작성 
-> POST로 /joinProc 
-> joinProcess 후 /login으로 리디렉트

joinProcess에서 수행할 로직 구현
=> joinService 생성

```
@Autowired  
private UserRepository userRepository;  
  
@Autowired  
private BCryptPasswordEncoder bCryptPasswordEncoder;  
  
public void joinProcess(JoinDTO joinDTO) {  
    UserEntity data = new UserEntity();  
  
    data.setUsername(joinDTO.getUsername());  
    data.setPassword(bCryptPasswordEncoder.encode(joinDTO.getPassword()));  
    data.setRole("ROLE_USER"); // ROLE_역할  
  
    userRepository.save(data);  
}
```
사용자로부터 직접 입력받은 데이터가 있는 DTO를 DB에 담기 전,
처리 과정을 거쳐서 DB에 담아야 함(비번 암호화, 역할 부여, ...)
=> 회원 가입 프로세싱 과정에서 수행할 것

UserEntity 생성
```
@Setter  
@Getter  
@Entity  
public class UserEntity {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private int id;  
    private String username;  
    private String password;  
    private String role;  
}
```
==@Entity== 로 Spring이 Entity 객체로 인식할 수 있게 명시
(Entity 객체는 ==@Id== 어노테이션이 달린 id 필드 필수)
==@GeneratedValue== 로 직접 데이터를 insert하지 않고 자동 생성(생성 타입=ID)

Entity는 데이터 바구니 역할, 이전에 쓰던 VO와 유사한 역할
DB 저장 전 id와 role을 추가해주고 저장할 것임

DB에 Entity 객체를 자동으로 테이블 생성해주기
=> Hibernate DDL 설정

application.properties 설정
```
spring.jpa.hibernate.ddl-auto=none 
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

`ddl-auto=none` 부분을 `ddl-auto=update`로 바꿔주고 프로젝트 실행하면
자동으로 Entity를 DB에 테이블로 넣어준다.

(주의!) 테이블 생성 후 다시 none으로 바꿔주기.
update로 놔두면 계속 덮어쓰기함

UserRepository 생성
```
public interface UserRepository extends JpaRepository<UserEntity, Integer> {  
    // repository는 인터페이스로 생성, jparepository 상속받도록 작성  
    // 맵핑은 entity:id -> UserEntity, Integer}
```

JpaRepository를 상속받음 -> JPA 관리

SecurityConfig에서 접근 권한 수정
: /join, /joinProc 은 회원가입이므로 비회원도 접근 가능해야 함 -> permitAll에 추가

### 회원 중복 검증 및 처리 로직
**회원 중복 검증**
username에 대해서 중복된 가입이 발생하면 서비스에서 아주 치명적인 문제가 발생하기 때문에 백엔드단에서 중복 검증과 중복 방지 로직을 작성해야 한다.

UserEntity Unique 설정
-> 아이디 정의에 ==@Column==(unique = true) 기입해준다.

UserRepository에 중복 검사 메서드 정의
-> `boolean existsByUsername(String username);`

###### 중복 검사 로직 구현 관련
아이디 중복 검사는 백엔드단 + 프론트엔드단 모두 수행하는 것이 바람직하다.

1. 프론트엔드단에서 1차 검증(정규식, 가입 불가 이름 등)
2. 백엔드단에서 2차 검증(중복 검사 등)

프론트단에서만 구현되어 있으면 GET 방식 SQL 인젝션 등으로 강제 가입이 가능하기 때문에
백엔드단에서 구현은 필수로 해 준다.
다만 백엔드단에서 다 구현해놓으면 코드가 복잡해지니까
프론트단에서는 정규식 활용으로 함

### DB 기반 로그인 검증 로직
**인증**
시큐리티를 통해 인증을 진행하는 방법 :
사용자가 Login 페이지를 통해 아이디, 비밀번호를 POST 요청
-> 스프링 시큐리티가 데이터베이스에 저장된 회원 정보를 조회 후 비밀번호를 검증
-> 서버 세션 저장소에 해당 아이디에 대한 세션을 저장

UserRepository에 추가: `UserEntity findByUsername(String username);`
=> 이름으로 객체 찾기

UserDetailsService 인터페이스를 통해서 사용자 정보를 가져와야 함
우리가 자체적으로 아이디 중복 검사 로직을 만들 것이므로 인터페이스 구현체를 만들어줌
-> CustomUserDetailsService.java
```
@Service  
public class CustomUserDetailsService implements UserDetailsService {  
  
    @Autowired  
    private UserRepository userRepository;  
  
    @Override  
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {  
  
        UserEntity userEntity = userRepository.findByUsername(username);  
  
        if (userEntity != null) {  
            return new CustomUserDetails(userEntity);  
        }  
  
        return null;  
    }  
}
```
사용자 이름(ID)으로 Entity 정보를 가져옴
Entity가 null이 아니면 CustomUserDetails 객체에 userEntity를 넘김

CustomUserDetails.java
```
public class CustomUserDetails implements UserDetails {  
    private UserEntity userEntity;  
  
    public CustomUserDetails(UserEntity userEntity) {  
        this.userEntity = userEntity;  
    }  
    
    @Override  
    public Collection<? extends GrantedAuthority> getAuthorities() {  
        // 권한 관련  
        Collection<GrantedAuthority> authorities = new ArrayList<>();  
        authorities.add(new GrantedAuthority() {  
            @Override  
            public String getAuthority() {  
                return userEntity.getRole();  
            }  
        });  
  
        return authorities;  
    }  
  
    @Override  
    public String getPassword() {  
        return userEntity.getPassword();  
    }  
  
    @Override  
    public String getUsername() {  
        return userEntity.getUsername();  
    }
    ...
}    
```
UserDetails를 상속받으면 좀 많은 메서드들을 오버라이딩함
지금 구현할 수 있는 부분만 구현하고(return값 준다)
나머지는 false 리턴하면 계정이 잠기므로 true로 다 바꿔준다.

getAuthorities()는 권한 정보를 가져오는 메서드
규칙은 거의 정해져 있음

### 세션 정보 얻어오기(ID, 권한)
로그인 후 Welcome창에 ID를 출력한다던가
권한 정보를 통해 해당 요청에 대한 권한 검증을 한다던가 할 때 필요

결과값 표시용 main.mustache 
-> 변수 추가
```
<body>  
main page  
<hr>  
{{id}}  
{{role}}  
</body>
```
`{{id}}` : mustache 문법, model 객체 속성에서 id를 표시

MainController.java
```
@GetMapping("/")  
public String mainP(Model model) {  
        
    String id = SecurityContextHolder.getContext().getAuthentication().getName();  
  
    // 권한 정보 얻는 로직 -> 권한 필요한 곳에 붙여다 써서 검증하면 됨  
    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();  
    Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();  
    Iterator<? extends GrantedAuthority> iter = authorities.iterator();  
    GrantedAuthority auth = iter.next();  
      
    String role = auth.getAuthority();  
  
    model.addAttribute("id", id);  
    model.addAttribute("role", role);  
  
    return "main";  
}
```
`SecurityContextHolder.getContext().getAuthentication().getName();`
Spring Security 제공 SecurityContext 관련 기능
여기서 id와 역할에 해당하는 정보 get -> model 속성으로 전달

정상 결과 : 
관리자 로그인할 경우
-> id = admin(가입한 id), role = ROLE_ADMIN

만약 로그인 안 한 사용자(비회원)가 접근하면
-> id = anonymousUser , role = ROLE_ANONYMOUS


### 세션 설정
사용자가 로그인을 진행한 뒤, 사용자 정보는 `SecurityContextHolder`에 의해서 서버 세션에 관리된다.

**세션 소멸 시간 설정**
세션 타임아웃 설정을 통해 로그인 이후 세션이 유지되고 소멸하는 시간을 설정할 수 있다.
세션 소멸 시점은 서버에 마지막 특정 요청을 수행한 뒤 설정한 시간 만큼 유지된다. (기본 시간 1800초)

application.properties에서
`server.servlet.session.timeout=1800` : 1800초
`server.servlet.session.timeout=90m` : 90분


**다중 로그인 설정**
[Authentication Persistence and Session Management 스프링 공식 문서 바로가기](https://docs.spring.io/spring-security/reference/servlet/authentication/session-management.html)
동일한 아이디로 다중 로그인을 진행할 경우에 대한 설정 방법은 세션 통제를 통해 진행한다.

SecurityConfig.java
```
...
http.sessionManagement((auth) -> auth  
                .maximumSessions(5)  
                .maxSessionsPreventsLogin(true));
...
```

sessionManagement() 메소드를 통한 설정을 진행한다.

maximumSession(정수) : 하나의 아이디에 대한 다중 로그인 허용 개수
maxSessionPreventsLogin(불린) : 다중 로그인 개수를 초과하였을 경우 처리 방법
- true : 초과시 새로운 로그인 차단
- false : 초과시 기존 세션 하나 삭제

**세션 고정 보호**

![](https://cafeptthumb-phinf.pstatic.net/MjAyNTA3MzBfNzAg/MDAxNzUzODIyNDkxNDAx.7Wa3kJRkR0OfmNLKzDLyFyGU7_B01yjsrO9KQywYbmkg.9bP2o9pF_NoOIwxd22046HxY2p045HZ3Wfum8Yt2UZ8g.PNG/image-39342e82-86de-4896-965f-884aadf780da.png?type=w1600)

위 그림처럼 해커가 서버에 접근해서 세션 쿠키를 획득하면 Admin 권한을 가지고 서버에 다시 접근할 수 있으므로 조치가 필요하다.
세션 고정 공격을 보호하기 위한 로그인 성공시 세션 설정 방법은 sessionManagement() 메소드의 sessionFixation() 메소드를 통해서 설정할 수 있다.

```
...
http.sessionManagement((auth) -> auth.sessionFixation().changeSessionId());
...
```

- `sessionManagement().sessionFixation().none()` : 로그인 시 세션 정보 변경 안함
- `sessionManagement().sessionFixation().newSession()` : 로그인 시 세션 새로 생성
- `sessionManagement().sessionFixation().changeSessionId()` : 로그인 시 동일한 세션에 대한 id 변경


### CSRF
**CSRF란?**
CSRF(Cross-Site Request Forgery)는 요청을 위조하여 사용자가 원하지 않아도 서버측으로 특정 요청을 강제로 보내는 방식이다. (회원 정보 변경, 게시글 CRUD를 사용자 모르게 요청)

**개발 환경에서 csrf disable()**
개발 환경에서는 Security Config 클래스를 통해 CSRF 설정을 disable 설정하였다. 배포 환경에서는 CSRF 공격 방지를 위해 csrf disable 설정을 제거하고 추가적인 설정을 진행해야 한다.

`http.csrf((auth) -> auth.disable());`

**배포 환경에서 진행 사항**
SecurityConfig 클래스에서 csrf.disable() 설정을 진행하지 않으면 자동으로 enable 설정이 진행된다. 
enable 설정시 스프링 시큐리티는 CsrfFilter를 통해 POST, PUT, DELETE 요청에 대해서 토큰 검증을 진행한다.

Security Config 클래스 설정
: csrf.disable() 구문 삭제

POST 요청에서 프론트 설정
: hidden 속성으로 csrf 토큰 추가

login.mustache
```
<form action="/loginProc" method="post" name="loginForm">  
    <input id="username" type="text" name="username" placeholder="id"/>  
    <input id="password" type="password" name="password" placeholder="password"/>  
    <input type="hidden" name="_csrf" value="{{_csrf.token}}"/>  
    <input type="submit" value="로그인"/>  
</form>
```

Ajax 요청시
HTML head 구획에 아래 요소 추가
```
<meta name="_csrf" content="{{_csrf.token}}"/> 
<meta name="_csrf_header" content="{{_csrf.headerName}}"/>
```

Ajax 요청시 위의 content 값을 가져온 후 함께 요청
XMLHttpRequest 요청시 setRequestHeader를 통해 `_csrf`, `_csrf_header` Key에 대한 토큰 값 넣어 요청

GET 방식 로그아웃을 진행할 경우 설정 방법
csrf 설정시 POST 요청으로 로그아웃을 진행해야 하지만 아래 방식을 통해 GET 방식으로 진행할 수 있다.

SecurityConfig 클래스 로그아웃 설정
```
...
http.logout((auth) -> auth.logoutUrl("/logout")  
                .logoutSuccessUrl("/"));
...
```
로그아웃 URL : /logout
로그아웃 성공시 연결 URL : /

LogoutController 생성
```
@Controller  
public class LogoutController {  
    @GetMapping("/logout")  
    public String logout(HttpServletRequest request, HttpServletResponse response) throws Exception {  
  
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();  
        if (authentication != null) {  
            new SecurityContextLogoutHandler().logout(request, response, authentication);  
        }  
  
        return "redirect:/";  
    }  
}
```
Logout을 GET 방식으로 요청할 것이므로 GET 매핑을 해준다.
SecurityContextHolder에 있는 사용자의 인가 정보를 LogoutHandler의 logout 메서드로 로그아웃한다.

mustache에서 csrf 토큰 변수 오류 발생시(csrf 토큰 속성(`_csrf`)을 못 찾을 때) :
아래 구문을 변수 설정 파일에 추가해준다.
application.properties
`spring.mustache.servlet.expose-request-attributes=true`

**API 서버의 경우 csrf.disable() ?**
앱에서 사용하는 API 서버의 경우 보통 세션을 STATELESS로 관리하기 때문에 스프링 시큐리티 csrf enable 설정을 진행하지 않아도 된다.
JWT를 사용해서 토큰을 관리하는 경우도 동일.


### InMemory 방식 유저 정보 저장
**소수의 유저를 저장할 좋은 방법**
토이 프로젝트를 진행하는 경우 또는 시큐리티 로그인 환경이 필요하지만 소수의 회원 정보만 가지며 데이터베이스라는 자원을 투자하기 힘든 경우에는 회원가입 없는 InMemory 방식으로 유저를 저장하면 된다.

이 방식은 InMemoryUserDetailsManager 클래스를 통해 유저를 등록하면 된다.

**InMemoryUserDetailsManager**
[https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/in-memory.html](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/in-memory.html)
SecurityConfig에 추가 - filterChain 말고 밖에
```
@Bean  
public UserDetailsService userDetailsService() {  
  
    UserDetails user1 = User.builder()  
            .username("admintest")  
            .password(bCryptPasswordEncoder().encode("admin1"))  
            .roles("ADMIN")  
            .build();  
  
    UserDetails user2 = User.builder()  
            .username("usertest")  
            .password(bCryptPasswordEncoder().encode("user1"))  
            .roles("USER")  
            .build();  
  
    return new InMemoryUserDetailsManager(user1, user2);  
}
```
ID: admintest, PW: admin1(암호화됨), 권한: ADMIN
ID: usertest, PW: user1(암호화됨), 권한: USER

DB 연동이 안 되어 있을때만 사용된다.
이미 사용자 DB가 있고 연동되어있으면 적용 안 됨

###### InMemoryUserDetailsManager 관련
스프링 시큐리티는 내부적으로 `UserDetailsService` 타입의 Bean을 단 하나만 사용합니다.

- 이미 다른 곳에서 `@Service` 또는 `@Bean` 으로
    `public class CustomUserDetailsService implements UserDetailsService {     ... }`
    이런 구현체를 등록해서 DB에서 사용자 정보를 읽어오고 있다면,  
    Spring Security는 **그 구현체를 우선적으로 사용**합니다.
    
- 반면,  
    DB 연동 없이,  
    오직 메모리 상의 사용자 정보만 쓰고 싶다면 `InMemoryUserDetailsManager`가 사용됩니다.

 ⚙️ 동작 우선순위
스프링 부트가 구동될 때,  
Security 설정 중 다음 순서로 `UserDetailsService`를 찾습니다:
1. **직접 등록한 Bean** (`@Bean` or `@Service` 로 등록된 `UserDetailsService`)
2. 없으면 **자동 구성된 InMemoryUserDetailsManager** (기본 계정 `user` + 랜덤 비밀번호)
3. 두 개가 다 있으면, **직접 등록한 Bean**이 우선합니다.



