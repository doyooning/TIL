#Java 
#Spring

---
# Apache Tomcat
오픈 소스 진영의 대표적인 WAS
웹 컴포넌트를 운용한다.

서블릿, 페이지, EL(Expression Language), 웹소켓, 어노테이션 등을 관리
JVM 구동 하에 WAS 실행 구조 

Servlet = Java Web Component
Java Server Page(JSP) -> Jakarta Pages로 이름을 바꿨음

POJO = Plain Old Java Object = 순수 자바 객체
IoC, AOP 등으로 POJO 프로그래밍을 지향

(Apache Project에 참여해서 순수 개발만 해보는 것도 개발자 성장에는 좋을듯 ?)

# Spring
인터페이스의 모듈화 -> 프레임워크

==모듈화:== 인테페이스의캡슐화를통해모듈화를강화하여설계와구현변경에따른영향을최소화하여소프트웨어품질을향상시킨다.
Spring은 주요 모듈들을 통해 기능을 확장, 추가하여 사용한다.

### Spring Framework
1. 단순화된 단위 테스팅
	- 의존성 주입(DI)
2. 복잡한 코드 감소
3. 아키텍처의 유연성(모듈 방식)
4. 변화하는 시대를 선도
	- Spring Boot
		: 개발 환경 설정 간소화, WAS를 내장


# Spring 환경설정 및 세팅

### ! Jakarta와 Tomcat 버전 호환
- Jakarta EE 9 이상은 **Tomcat 10 이상**과 호환된다. 
    (Tomcat 9는 `javax.*`, Tomcat 10은 `jakarta.*` 네임스페이스)
- IntelliJ에서 Jakarta EE 10/11 선택 시, Tomcat 9는 자동 인식이 안 됨
    
즉,
- Jakarta EE 8 → Tomcat 9
- Jakarta EE 9~11 → Tomcat 10 

### ! Intellij 환경 설정
==Tomcat 설치 확인하고 새 프로젝트 생성==

버전 확인 : Jakarta EE 8, Tomcat 9.xx

==생성 후 상단 Tomcat -> Edit Configurations==
	Deployment에서 기존 war 지움, 새로 Artifact 추가 : ~.war (exploded)
	이후 Application Context를 / 하나 빼고 다 지움
	=> 주소 부분이 깨끗해짐

==한글이 깨질 수 있으므로 UTF-8 설정==
File -> Help -> Edit custom VM options
	idea64.exe.vmoptions 파일 열림, 내용 입력 :
		- Xmx2048m
		- Dfile.encoding=UTF-8
		- Dconsole.encoding=UTF-8
	( -(dash) 포함)
	`- Xmx2048m` : 저장 메모리의 크기를 확장해줌(한글에 최적화)
	`-Dfile.encoding=UTF-8` : 파일의 인코딩을 UTF-8로 설정
	`-Dconsole.encoding=UTF-8` : 콘솔의 인코딩을 UTF-8로 설정

==다시 상단 Tomcat -> Edit Configurations 에서 서버 관련 설정==
	Server 메뉴에서 브라우저를 Chrome으로 변경
	VM options에 `-Dfile.encoding=UTF-8` 입력
	On 'update' action과 On frame deactivation 모두 `Update classes and resources` 로 변경

==Spring framework 라이브러리 추가==

build.gradle
```
dependencies {  
    compileOnly('javax.servlet:javax.servlet-api:4.0.1')  
    testImplementation("org.junit.jupiter:junit-jupiter-api:${junitVersion}")  
    testRuntimeOnly("org.junit.jupiter:junit-jupiter-engine:${junitVersion}")  
    implementation 'org.springframework:spring-core:5.3.27'  
    implementation 'org.springframework:spring-context:5.3.27'  
    implementation 'org.springframework:spring-webmvc:5.3.27'  
    testImplementation 'org.springframework:spring-test:5.3.27'  
  
    compileOnly 'org.projectlombok:lombok:1.18.30'  
    annotationProcessor 'org.projectlombok:lombok:1.18.30'  
    testCompileOnly 'org.projectlombok:lombok:1.18.30'  
    testAnnotationProcessor 'org.projectlombok:lombok:1.18.30'  
    implementation 'org.apache.logging.log4j:log4j-api:2.22.1'  
    implementation 'org.apache.logging.log4j:log4j-core:2.22.1'  
    implementation 'org.apache.logging.log4j:log4j-slf4j-impl:2.22.1'  
    // https://mvnrepository.com/artifact/jstl/jstl  
    implementation 'jstl:jstl:1.2'  
}
```
의존성 추가
spring-core, spring-context, spring-webmvc, spring-test
lombok, log4j, jstl

index.jsp
 %@ : 서버에서 실행하는 태그임을 알려줌(서버 디렉티브)
  `<%= "Hello World!" %>` : print 함수와 동일

# 서블릿(Servlet)
==서버==쪽에서 실행되면서 클라이언트의 요청에 따라 동적으로 서비스를 제공하는 Java 클래스

일반 자바 프로그램과는 다르게 독자적으로 실행X
ex) 톰캣, 웹스피어, 웹로직, 제우스, 제이보스
서블릿은 WAS에 의해 실행됨
(WAS = Web Application Server) - WAS는 웹서버의 기능과 웹컨테이너의 기능 모두 가능

서버에서 실행되다가 웹 브라우저에서 요청을 하면 해당 기능을 수행한 후 웹 브라우저에 결과를 전송

서블릿도 Java 클래스이므로, 실행하면 초기화 과정과 메모리에 인스턴스를 생성하여 서비스를 수행한 후
다시 소멸하는 과정을 거침
이런 단계를 거칠 때마다 서블릿 클래스의 메서드가 호출되어 초기화, DB연동, 마무리 작업을 수행함
각 과정에서 호출되는 메서드들이 서블릿 생명주기 메서드임

`init()` : 서블릿 요청시 처음 한 번만 호출됨, 서블릿 생성시 초기화 작업 수행
작업 수행 : 
1. `doGet()` : 서블릿 요청시 매번 호출
2. `doPost()` : 실제로 클라이언트가 요청하는 작업을 수행함
`destroy()` : 서버 종료시 호출됨

##### **서블릿의 상속/구현 관계**
서블릿의 최상위 클래스 : GenericServlet
(GenericServlet은 java.lang.Object를 상속받음 - Java에서 최상위 슈퍼클래스)

웹 사이트에 알맞은 서블릿 구현을 도와주는 클래스 : HttpServlet
-> GenericServlet을 상속받음

GenericServlet은 Servlet, ServletConfig, Serializable을 구현

## HTML과 Servlet 연결 및 요청-응답
간단한 로그인 페이지 구현

login.html
```
<!DOCTYPE html>  
<html lang="ko">  
<head>  
    <meta charset="UTF-8">  
    <title>로그인 페이지</title>  
</head>  
<body>  
    <form action="login" method="get" enctype="UTF-8">  
        아이디 : <input type="text" name="user_id"><br>  
        비밀번호 : <input type="password" name="user_pwd"><br>  
        <input type="submit" value="로그인">&nbsp&nbsp<input type="reset" value="취소">  
    </form></body>  
</html>
```
form 속성 중요
`action="login"` : login으로 연결
`method="get"` : get 방식으로 요청 보냄

LoginServlet.java
```
// ... import 생략
@WebServlet("/login")  
public class LoginServlet extends HttpServlet {  
    @Override  
    public void init() throws ServletException {  
        System.out.println("LoginServlet init");  
    }  
  
    @Override  
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
	  // 내용 생략
    }  
  
    @Override  
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        request.setCharacterEncoding("UTF-8"); // 인코딩 먼저  
        String username = request.getParameter("user_id"); 
        // form > input에서 받았던 name 속성  
        String password = request.getParameter("user_pwd");  
  
        System.out.println(username);  
        System.out.println(password);  
  
        response.setContentType("text/html;charset=UTF-8");  
        response.setCharacterEncoding("UTF-8");  
  
        PrintWriter out = response.getWriter();  
        out.println("<html><body>");  
        out.println("<h1>" + username + "</h1>");  
        out.println("<h1>" + password + "</h1>");  
        out.println("</body></html>");  
    }  
}
```
get 방식의 요청을 받음 -> doGet() 호출

보통은 Servlet 내에 doHandle()과 같은 요청 핸들러 메서드를 만들고 요청 핸들러에서 한번에 처리한다.
doGet(), doPost()에 doHandle() 메서드를 호출해준다.

```
@Override  
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
    System.out.println("LoginServlet doPost");  
    doHandle(request, response);  
}

@Override  
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
    System.out.println("LoginServlet doGet");  
    doHandle(request, response);  
}

private void doHandle(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
	 // 내용 생략
}
```

#### **HTML+JS와 Servlet 연결**
당연하게도 HTML 내에 JavaScript를 실행시킬 수 있다.

login2.html
```
<!DOCTYPE html>  
<html lang="ko">  
<head>  
    <meta charset="UTF-8">  
    <title>로그인 페이지</title>  
    <script type="text/javascript">  
        function fn_validate() {  
            let frmLogin = document.frmLogin;  
            let user_id = frmLogin.user_id.value;  
            let user_pwd = frmLogin.user_pwd.value;  
  
            if ((user_id.length == 0 || user_id == "") || (user_pwd.length == 0) || (user_pwd == "")){  
                alert("아이디와 비밀번호 입력은 필수입니다.");  
            } else {  
                frmLogin.method = "post";  
                frmLogin.action = "login3";  
                frmLogin.submit();  
            }  
        }  
    </script>  
</head>  
<body>  
<form name="frmLogin" enctype="UTF-8">  
    아이디 : <input type="text" name="user_id"><br>  
    비밀번호 : <input type="password" name="user_pwd"><br>  
    <input type="hidden" name="user_address" value="서울시 강남구" />  
    <input type="button" onclick="fn_validate()" value="로그인">&nbsp&nbsp<input type="reset" value="취소">  
</form>  
</body>  
</html>
```

LoginServletJs.java
```
// ... import 생략
@WebServlet("/login3")  
public class LoginServletJs extends HttpServlet {  
	// ... doGet, doPost 생략
	
    private void doHandle(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {  
        request.setCharacterEncoding("UTF-8");  
        response.setContentType("text/html;charset=utf-8");  
        PrintWriter out = response.getWriter();  
  
        String user_id = request.getParameter("user_id");  
        String user_pwd = request.getParameter("user_pwd");  
        String user_address = request.getParameter("user_address");  
  
        String data = "<html>";  
        data += "<body>";  
        data += "<h1>" + user_id + "</h1>";  
        data += "<p>" + user_pwd + "</p>";  
        data += "<p>" + user_address + "</p>";  
        data += "</body>";  
        data += "</html>";  
        out.print(data);  
    }  
}
```
HTML요소들을 DOM 객체 안에서 찾게 되는데(document.~)
포함 관계를 잘 파악하고 지정해줘야 한다.

`let frmLogin = document.frmLogin;`
: form태그인 frmLogin(name 속성)은 바로 찾아지지만

`let user_id = frmLogin.user_id.value;`
: form태그 안의 input태그 user_id(name 속성)는 document에서 바로 안 찾아짐
-> 그냥 바로 위 부모노드인 frmLogin에서 찾으면 찾아짐


  