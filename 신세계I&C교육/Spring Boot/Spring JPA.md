#Java 
#Spring 

---
### Spring Boot 프로젝트 생성
start.spring.io에서 생성


**개발 환경 설정**

1. 데이터베이스 설치
2. JDK 설치
3. 프로젝트 생성

- 스프링 부트 백엔드 프로젝트 생성
- Vite를 활용한 프론트엔드 프로젝트 생성

---
### 프로젝트 설계
#### **가장 먼저 해야할 일:**
패키지 추가, 유틸리티 추가(HTTP)

#### **도메인별 구현 순서 정하기**
ex) 상품 API -> 계정 API -> ...

#### **공통 기능 구현**
세션 기능 구현
```java
public class HttpUtils {  
  
    // 세션 입력  
    public static void setSession(HttpServletRequest request, String key, Object value) {  
        request.getSession().setAttribute(key, value);  
    }  
  
    // 세션 값 조회  
    public static Object getSession(HttpServletRequest request, String key) {  
        return request.getSession().getAttribute(key);  
    }  
  
    // 세션 삭제  
    public static void removeSession(HttpServletRequest request, String key) {  
        request.getSession().removeAttribute(key);  
    }  
}
```
request는 세션을 가지고 있다.
`request.getSession()` : request에 있는 세션 정보를 얻음
세션에서는 `setAttribute(key, value)`로 속성을 추가해줄 수 있고,
`getAttribute(key)`로 해당 key를 가진 value를 얻을 수 있다.

세션 삭제 로직은 로그아웃 구현에 필요


#### **도메인별 API 구현**
테이블 생성 -> 엔티티 작성 -> 리포지토리 작성 -> 서비스 구현 -> 컨트롤러 구현
(상식적으로 생각해봐도 이 순서가 맞다..)

테이블을 직접 쿼리로 Create해서 넣고 엔티티 만들어도 되고
엔티티를 제약 조건 다 반영해서 만들고 Hibernate 통해서 DDL-auto로 만들어도 된다.

리포지토리 작성
: 별거 없음
```java
public interface MemberRepository extends JpaRepository<Member, Integer> {  
    // 아이디와 패스워드로 회원 정보를 조회  
    Optional<Member> findByLoginIdAndLoginPw(String loginId, String loginPw);  
}
```
Member 찾는 기능에는 Optional 권장(null 처리)

서비스 구현
: MemberService(회원 관리 기본 기능 - 찾기, 저장) + AccountHelper(로그인 등 회원 관리 기능 구현 부분)
```java
@Service  
@RequiredArgsConstructor  
public class BaseMemberService implements MemberService {  
    private final MemberRepository memberRepository;  
  
    @Override  
    public void save(String name, String loginId, String loginPw) {  
        memberRepository.save(new Member(name, loginId, loginPw));  
    }  
  
    @Override  
    public Member find(String loginId, String loginPw) {  
        Optional<Member> member = memberRepository.findByLoginIdAndLoginPw(loginId, loginPw);  
        return member.orElse(null); // 회원 데이터가 있으면 해당 멤버를 리턴, 없으면 Null    }  
}
```

AccountHelper의 기능
```java
public interface AccountHelper {  
    // 회원가입  
    void join(AccountJoinRequests joinReq);  
  
    // 로그인  
    String login(AccountLoginRequests loginReq, HttpServletRequest request, HttpServletResponse response);  
  
    // 회원 아이디 조회  
    Integer getMemberId(HttpServletRequest request);  
  
    // 로그인 여부 확인  
    boolean isLoggedIn(HttpServletRequest request);  
  
    // 로그아웃  
    void logout(HttpServletRequest request, HttpServletResponse response);  
}
```

SessionAccountHelper <- AccountHelper 구현체
```java
...
@Override  
public String login(AccountLoginRequests loginReq, HttpServletRequest request, HttpServletResponse response) {  
    Member member = memberService.find(loginReq.getLoginId(), loginReq.getLoginPw());  
    // 회원 데이터가 없으면  
    if (member == null) {  
        return null;  
    }  
    HttpUtils.setSession(request, AccountConstants.MEMBER_ID_NAME, member.getId());  
    return member.getLoginId().toString();  
}
...
```
로그인 기능
`HttpUtils.setSession(request, AccountConstants.MEMBER_ID_NAME, member.getId());`
: 세션 설정 - `키` : MEMBER_ID_NAME(상수 선언됨), `값` : member객체에서 얻은 Id 

```java
// 계정 관련 상수 정의  
public class AccountConstants {  
    public static final String MEMBER_ID_NAME = "member_id";  
}
```
`MEMBER_ID_NAME` :세션에 들어갈 고정 키값을 "member_id"로 설정

컨트롤러 구현
RestController이므로 `ResponseEntity<>` 반환
DTO를 파라미터로 넘길 땐 `@RequestBody` 

```java
@PostMapping("/api/account/login")  
public ResponseEntity<?> login(HttpServletRequest request, HttpServletResponse response, @RequestBody AccountLoginRequests loginReq) {  
    if(loginReq.getLoginId() == null || loginReq.getLoginPw() == null) {  
        return new ResponseEntity<>(HttpStatus.BAD_REQUEST);  
    }  
  
    String output = accountHelper.login(loginReq, request, response);  
    if (output == null) {  
        return new ResponseEntity<>(HttpStatus.NOT_FOUND);  
    }  
    return new ResponseEntity<>(output, HttpStatus.OK);  
}
```

---
### 기능 테스트

1. 콘솔에서 확인:
```js
fetch("/v1/api/account/login", {
	headers : {"Content-type": "application/json;charset=utf-8"},
	method : "POST",
	body : JSON.stringify({
		loginId: "ssg1234@ssg.com",
		loginPw: "1q2w3e4r"
	})
}).then(res => console.log(res.status));
```
=> 200 OK를 확인하면 됨

2. POSTMAN 사용
`POST http://localhost:8080/v1/api/account/join` 요청 보내기
-> Body - raw - JSON 선택해서 JSON 형태로 요청을 보내야 함(RestController)
```json
{
    "loginId" : "ssg4255@ssg.com",
    "loginPw" : "4255",
    "name" : "ssg3"
}
```
회원가입 요청 DTO 속성에 맞춰 JSON 형태를 만들어서 요청 전송
=> 200 OK를 확인하면 됨


