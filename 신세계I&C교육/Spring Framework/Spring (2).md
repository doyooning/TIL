#Java 
#Spring 

---
# MVC 패턴으로 DB 정보 보여주기
PRG패턴으로 구성
PRG = POST, Redirect, GET

웹 페이지에서 사용자의 POST 요청을 받아 서버가 DB에 데이터를 저장하는 경우
사용자가 뒤로가기를 누르거나 새로고침을 수행할 때 중복 입력이 되는 문제가 발생
(뒤로가기, 새로고침을 하면 마지막 사용자의 요청을 서버가 받고 응답함)
-> 리디렉션으로 해결

클라: 폼 제출 POST요청 -> 서버: 응답으로 Redirect(HTTP 300) -> 클라: 폼 제출 화면 GET요청 -> 서버: 응답으로 맨 처음 페이지 주소 제공(HTTP 200)
이렇게 하면 폼 제출 화면의 GET요청이 가장 마지막 요청이므로 안전함

[PRG패턴 적용시 흐름도](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FTfNrO%2FbtrPgDbdDOA%2FAAAAAAAAAAAAAAAAAAAAAHyxk0Y5iovBE3KOnffDcB4PSB1CeW7KeaVJX50NIICP%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1761922799%26allow_ip%3D%26allow_referer%3D%26signature%3D3UK580DbE3vtxKp5gYTf%252FcnrCvY%253D)

Spring에서 구현:
`resp.sendRedirect("/calc/input");` : doPost 메서드에 구현, 리디렉트 주소 입력

# Connection Pool 구성, HikariCP
### JNDI (Java Naming and Directory Interface)
커넥션 풀 객체 구현 
=> javax.sql.DataSource 클래스 웹 애플리케이션은 커넥션 풀 객체에 접근 시 JNDI 를 이용한다.

- 웹브라우저에서 name/value 쌍으로 전송한 후 서블릿에서 getParameter(name) 로 값을 가져올때
- 해시맵이나 해시테이블에 키/값으로 저장한 후 키를 이용하여 값을 가져올때
- 웹브라우저에서 도메인 네임으로 DNS 서버에 요청할 경우 도메인 네임에 대한 IP주소를 가져올때

HikariCP 설정 
Connection Pool ⇒ HikariCP(https://github.com/brettwooldridge/HikariCP) 이용한다.

Connection Pool이 잘 구동되는지 Test
```
@Test  
public void testHikariCP() throws Exception {  
    HikariConfig config = new HikariConfig();  
    config.setDriverClassName("com.mysql.cj.jdbc.Driver");  
    config.setJdbcUrl("jdbc:mysql://localhost:3306/member?serverTimezone=Asia/Seoul&charEncoding=UTF-8");  
    config.setUsername("root");  
    config.setPassword("mysql1234");  
    
    HikariDataSource ds = new HikariDataSource(config);  
    Connection conn = ds.getConnection();  
  
    Assertions.assertNotNull(conn);  
    conn.close();  
}
```
드라이버 관련 설정이 잘 작동하는지를 테스트(Not Null인지)

