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
(Mustache는 뷰 템플릿 역할)

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


