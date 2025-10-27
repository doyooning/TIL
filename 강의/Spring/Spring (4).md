#Java 
#Spring 

---
# Spring Web MVC 환경설정
기본 xml 구조 설정

WEB-INF 디렉토리
: Tomcat이 관리하는 디렉토리로 외부에 드러나지 않음
 -> 디렉토리 만들고(spring) root-context, servlet-context.xml 추가
	New -> XML Configuration File -> Spring Config 선택

root-context.xml
```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
</beans>
```
여기서 Bean 등록 처리, xml들을 다시 web.xml에 등록해주면 끝


### IoC(Inversion of Control)
= 제어의 역전

필요로 하는 여러 가지 기능들을 미리 만들어놓고 사용자가 요구할 때 딱딱 제공해줌
즉 사용자가 이것저것 만들어서 쓰는 게 아니라 스프링이 미리 만들어놓은 기능들을 사용자는 가져다 씀
(실질적으로 제어하는 사람은 스프링)

### 웹 프로젝트 설정

ApplicationContext
: 스프링이 빈들을 관리하는 공간
• root-context.xml을 읽어서 해당 클래스들을 인스턴스화하여 ApplicationContext 내부에서 관리

==@Autowired==
@Autowired 가 있는 필드의 경우 해당 타입의 객체가 스프링의 컨텍스트 내 존재한다면 실행시 주입된다.

• 패키지를 지정해서 해당 패키지 내 클래스의 인스턴스들을 스프링의 빈으로 등록하기 위해서 사용
• 특정 어노테이션을 이용해서 스프링의 빈으로 관리될 객체를 표시

==@Controller== 
: MVC 컨트롤러를 위한 어노테이션 

==@Service==
: 서비스 계층의 객체를 위한 어노테이션 

==@Repository==
: DAO와 같은 객체를 위한 어노테이션 

==@Component== 
: 일반 객체나 유틸리티 객체를 위한 어노테이션

생성자 주입 방식
1. 주입받는 타입을 final로 선언하고 생성자를 통해서 의존성 주입
2. lombok의 @RequiredArgsConstructor 를 통해서 생성자 자동생성

	@Repository는 어노테이션이 붙은 빈이 2개 이상이 될 때 에러 발생
	때문에 @Primary나 @qualifier("~") 적용

==@Qualifier==
: 이름을 지정하여 특정한 이름의 객체를 주입받는 방식 
Lombok과 @Qualifier 를 같이 이용하기 위해 src/main/java 폴더에 lombok.config  파일생성

lombok.config
`lombok.copyableAnnotations += org.springframework.beans.factory.annotation.Qualifier`

web.xml
WEB-INF폴더에 web.xml listener, context-param 추가
```
<context-param>
	 <param-name>contextConfigLocation</param-name>
	 <param-value>/WEB-INF/root-context.xml</param-value>
<context-param>
<listener>
	 <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
<listener>
```

톰캣을 실행하면 스프링과 관련된 로그가 기록되면서 실행 확인

### DataSource 구성하기

root-content.xml -> HikariCP 설정
```
<bean id="hikariconfig" class="com.zaxxer.hikari.HikariConfig">  
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"></property>  
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/sqldb?serverTimezone=Asia/Seoul"></property>  
    <property name="username" value="root"></property>  
    <property name="password" value="mysql1234"></property>  
    <property name="dataSourceProperties">  
        <props>            <prop key="cachePrepStmts">true</prop>  
            <prop key="prepStmtCacheSize">250</prop>  
            <prop key="prepStmtCacheSqlLimit">2048</prop>  
        </props>  
    </property>  
</bean>  
<bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource" destroy-method="close">  
    <constructor-arg ref="hikariconfig"/>  
</bean>
```

활용:
```
@Autowired  
private DataSource dataSource;
```

커넥션만 얻어오면 되고 자동으로 알아서 close 처리 해 준다.
`Connection conn = dataSource.getConnection();`


### JDBC Template
