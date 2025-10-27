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

ApplicationContext
: 스프링이 빈들을 관리하는 공간
• root-context.xml을 읽어서 해당 클래스들을 인스턴스화하여 ApplicationContext 내부에서 관리

