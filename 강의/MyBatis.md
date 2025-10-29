#Java 
#DB

---
# MyBatis
객체 지향 언어인 자바의 관계형 데이터베이스 프로그래밍을 쉽게 도와 주는 개발 프레임 워크

JDBC를 통해 데이터베이스에 엑세스하는 작업을 캡슐화하고 일반 SQL 쿼리, 저장 프로 시저 및 고급 매핑을 지원하며 모든 JDBC 코드 및 매개 변수의 중복작업을 제거 합니다. 
MyBatis에서는 프로그램에 있는 SQL쿼리들을 한 구성파일에 구성하여 프로그램 코드와 SQL을 분리할 수 있는 장점을 가지고 있습니다.

### MyBatis 특징
복잡한 쿼리나 다이나믹한 쿼리에 강하다 - 반대로 비슷한 쿼리는 남발하게 되는 단점이 있다.
프로그램 코드와 SQL 쿼리의 분리로 코드의 간결성 및 유지보수성 향상
resultType, resultClass등 Vo를 사용하지 않고 조회결과를 사용자 정의 DTO, MAP 등으로 맵핑하여 사용 할 수 있다.
빠른 개발이 가능하여 생산성이 향상된다.

###### 1. build.gradle 추가
```
implementation 'org.springframework:spring-tx:5.3.27'  
implementation 'org.springframework:spring-jdbc:5.3.27'
implementation 'org.mybatis:mybatis:3.5.7'
implementation 'org.mybatis:mybatis-spring:2.0.6'
```

###### 2. root-context.xml 추가
```
<mybatis:scan base-package="com.ssg.springwebmvc.mapper"></mybatis:scan>
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">  
    <property name="dataSource" ref="dataSource"/>  
</bean>
```
SQLSessionFactory 설정 및 MyBatis가 스캔할 디렉토리 지정
(import : `xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"`)

###### 3. Mapper 인터페이스 작성(쿼리문)

간단한 쿼리문은 인터페이스에 직접 작성
```
// DB의 현재 시각을 문자열로 처리하는 메서드 getTimeNow()
public interface TimeMapper {  
    @Select("select now()")  
    String getTimeNow();  
}
```

복잡하거나 긴 쿼리문은 xml에서 작성
```
public interface TimeMapper2 {  
    String getNow();  
}
```

TimeMapper2.xml
```
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper  
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">  
<mapper namespace="com.ssg.springwebmvc.mapper.TimeMapper2">  
	<-- id와 사용함수명 일치 필요 -->
    <select id="getNow" resultType="string">  
        SELECT NOW()  
    </select>  
  
</mapper>
```

###### 4. root-context에 입력
```
<-- sqlSessionFactory 빈 내부에 입력 -->
	<property name="mapperLocations" value="classpath:/mappers/**/*.xml"/>
```
mapper 위치를 지정해줌
`classpath:/mappers/**/*.xml` : mappers 디렉토리 안의 모든 xml 확장자(중간에 디렉토리 있어도 됨)


### MyBatis Mapper 활용
resources에 mapper xml 생성

MemberMapper.xml
```
<mapper namespace="com.ssg.memberservicetest.mapper.MemberMapper">  
  
    <resultMap id="MemberResultMap" type="com.ssg.memberservicetest.dto.MemberDTO">  
        <id property="userId"    column="userId"/>  
        <result property="userPwd"   column="userPwd"/>  
        <result property="userName"  column="userName"/>  
        <result property="userEmail" column="userEmail"/>  
        <result property="joinDate"  column="joinDate" jdbcType="DATE" javaType="java.time.LocalDate"/>  
    </resultMap>  
  
    <insert id="insert" parameterType="com.ssg.memberservicetest.dto.MemberDTO">  
        INSERT INTO member (userId, userPwd, userName, userEmail, joinDate)  
        VALUES (#{userId}, #{userPwd}, #{userName}, #{userEmail},  
        #{joinDate, jdbcType=DATE, javaType=java.time.LocalDate})  
    </insert>  
  
     <select id="findAll" resultMap="MemberResultMap">  
        SELECT userId, userPwd, userName, userEmail, joinDate  
        FROM member  
        ORDER BY userId  
    </select>  
  
    <select id="findById" parameterType="string" resultMap="MemberResultMap">  
        SELECT userId, userPwd, userName, userEmail, joinDate  
        FROM member  
        WHERE userId = #{userId}  
    </select>  
</mapper>
```
mapper의 네임스페이스를 해당 xml 이름으로 지정, 쿼리 작성
resultMap은 이전에 사용했던 resultSet과 비슷한 역할 -> **쿼리 결과와 DB 테이블 맵핑**

- id : 쿼리를 사용할 메서드명과 같게 해 준다.
- parameterType : 
- resultMap : 



