#Java 
#Spring 

---
### 관점 지향 프로그래밍(Aspect-Oriented Programming, AOP)
- 객체 지향 프로그래밍 패러다임을 보완하는 기술로,
    메소드나 객체의 기능을 
    핵심 관심사(Core Concern)와 공통 관심사(Cross-cutting Concern)로 나누어 프로그래밍하는 것 . 
    “핵심 관심사”는 각 객체가 가져야 할 본래의 기능이며,
    “공통 관심사”는 여러 객체에서 공통적으로 사용되는 코드를 의미한다.

- 여러 개의 클래스에서 반복해서 사용하는 코드가 있다면 해당 코드를 모듈화 하여 공통 관심사로 분리한다.
- 분리한 공통 관심사를 Aspect로 정의하고 Aspect를 적용할 메소드나 클래스에 Advice를 적용하여 공통 관심사와 핵심 관심사를 분리할 수 있다.
- 이렇게 AOP에서는 공통 관심사를 별도의 모듈로 분리하여 관리하며, 이를 통해 코드의 재사용성과 유지 보수성을 높일 수 있다.

### Spring AOP
Spring AOP는 스프링 프레임워크에서 제공하는 기능 중 하나로 관점 지향 프로그래밍을 지원하는 기술

로깅, 보안, 트랜잭션 관리 등과 같은 공통적인 관심사를 모듈화하여 코드 중복을 줄이고 
유지 보수성을 향상하는데 도움을 준다.
![[Pasted image 20251104094153.png]]
클래스 A에서는 주황, 파랑, 빨간색 블록으로 구성이 되어 있고 
클래스 B에서는 빨강, 주황 블록으로 구성되어 있으며 
클래스 C에서는 주황, 파랑 블록으로 구성되어 있습니다.

해당 색은 클래스 A, B, C에서 동일하게 사용되는 코드를 의미합니다.

예를 들어서 클래스 A에서 주황색 블록을 수정을 하게 되면 클래스 B, C에서도 수정을 해야 합니다. 
이렇게 되면 유지보수 차원에서 모든 코드를 수정해야 하니 불편한 점이 있습니다. 
그래서 Aspect X에서는 공통 관심사인 주황색 블록을 묶어서 모듈화를 시켜서 코드의 재사용성과 유지 보수성을 강화하였습니다.

이렇듯, 관점 지향 프로그래밍에서는 소스코드에서 반복적으로 사용하는 코드를 하나로 묶어서 모듈화하여 재사용성과 유지 보수성을 높일 수 있는 강점을 가지고 있습니다.

![[Pasted image 20251104095155.png]]
![[Pasted image 20251104095308.png]]
`Trasnaction Advisor` 다음에 `Custom Advisor`가 위치하면, `Custom Advisor`또한 `Transaction`에 포함되는것입니다.중첩 AOP 사용시, `Advisor`가 수행되는 시점은 굉장히 중요합니다.

수행 순서에 따라 결과 데이터가 다를 수도 있고, 트랜잭션 포함여부가 달라 질 수 있습니다.

###### Spring AOP 이해하기

**주요 용어**
- Aspect : 공통적인 기능을 모듈화한 것을 의미
- Target : Aspect가 적용될 대상을 의미하며 메서드, 클래스 등이 이에 해당
- Join point : Aspect가 적용될 수 있는 시점을 의미하며 메서드 실행 전, 후 등이 될 수 있음
- Advice : Aspect의 기능을 정의한 것으로 메서드의 실행 전, 후, 예외처리 발생 시 실행되는 코드
- Point cut : Advice를 적용할 메서드의 범위를 지정하는 것을 의미

**주요 어노테이션**
- @Aspect : 해당 클래스를 Aspect로 사용
- @Before : 대상 메서드가 실행되기 전에 Advice를 실행
- @AfterReturning : 대상 메서드가 정상적으로 실행되고 반환된 후에 Advice를 실행
- @AfterThrowing : 대상 메서드에서 예외가 발생했을 때 Advice를 실행
- @After : 대상 메서드가 실행된 후에 Advice를 실행
- @Around : 대상 메서드 실행 전, 후 또는 예외 발생 시에 Advice를 실행

### AOP 실습

build.gradle 추가
```groovy
implementation 'org.springframework:spring-aop:5.3.27' 
implementation 'org.aspectj:aspectjrt:1.9.8' 
implementation 'org.aspectj:aspectjweaver:1.9.8'
```

root-context.xml proxy , aop 설정
```xml
<context:component-scan base-package="com.ssg.springboard.aop"/> <aop:aspectj-autoproxy/>
```

aop > LogAdvice.java
```java
@Aspect  
@Component  
@Log4j2  
public class LogAdvice {  
    @Before("execution(* com.ssg.springboard.service.BoardService.getList(..))")  
    public void logBefore(){  
        log.info("------------before-----------------------");  
        log.info("------------before-----------------------");  
        log.info("------------before-----------------------");  
    }  
  
    @Around("execution(* com.ssg.springboard.service.*Service.*(..))")  
    public Object logTime(ProceedingJoinPoint proceedingJoinPoint) throws Throwable{  
  
        long start = System.currentTimeMillis();  
  
        Object result = proceedingJoinPoint.proceed();  //메소드를 실행하는 코드  
  
        long end = System.currentTimeMillis();  
  
        long gap = end - start;  
  
        log.info("-------------------------");  
        log.info(proceedingJoinPoint.getTarget());  
        log.info(proceedingJoinPoint.getSignature());  
        log.info("TIME: " + gap);  
  
        if(gap > 100){  
            log.warn("-----------------WARN-------------");  
        }  
  
        return result;  
    }  
}
```

### Transaction 실습

build.gradle
```groovy
implementation 'org.springframework:spring-tx:5.3.27'
```

root-context.xml
```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
</bean>
<aop:aspectj-autoproxy/>
<tx:annotation-driven/>
```

테스트 테이블 추가
```sql
create table transaction_test1 ( 
      tno1 int auto_increment primary key, col varchar(500)  
      ); 
create table transaction_test2 ( 
      tno2 int auto_increment primary key, col varchar(500)  
      ); 
```

TimeMapper 인터페이스 테스트 기능 추가
```java
@Insert("insert into transaction_test1 (col) values (#{str})")
int insert1(String str);

@Insert("insert into transaction_test2 (col) values (#{str})")
int insert2(String str);
```

ReplyService.java 적용 (@Trancational 어노테이션 확인)
```java
@Transactional
@RequiredArgsConstructor
@Log4j2
@Service
public class ReplyService {

  private final ReplyMapper replyMapper;
  private final TimeMapper timeMapper;
  
  public void insertTwo(String str) {
  
    timeMapper.insert1(str);
    timeMapper.insert2(str);
  
  }
```

ReplyController.java 추가
```java
@GetMapping("/txtest")
public String[] get(String str) {
   replySerivce.insertTwo(str);
   return new String[] {"A","B","C"};
}
```

POSTMAN 에서 확인
`http://localhost:8080/time/txtest?str=1234567890...(50자)`
-> 두 테이블에 모두 반영 O

`http://localhost:8080/time/txtest?str=1234567890...(>50자)`
-> 두 테이블에 모두 반영 X (트랜젝션 해제하면 test1에만 insert됨)

Error 확인 ⇒ sample2 테이블에 col 컬럼의 속성이 50자를 넘을 수 없다.
```sql
select * from transaction_test1;
select * from transaction_test2; 
```

Trancational 을 적용해 놓았기 때문에 두 기능이 모두 rollback (취소) 두 테이블중 하나라도 컬럼속성에 맞지 않으면 모두 적용되지 않습니다.



