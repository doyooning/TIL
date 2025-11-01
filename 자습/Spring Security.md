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

