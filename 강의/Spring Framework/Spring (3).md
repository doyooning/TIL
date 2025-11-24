#Java 
#Spring 

---
# 객체 맵핑(Mapping)
### ModelMapper
객체 -> 객체 변환을 쉽게 할 수 있도록 도와주는 라이브러리

MapperUtil.java 생성
```
public enum MapperUtil {  
    INSTANCE;  
  
    private ModelMapper modelMapper;  
    MapperUtil() {  
        modelMapper = new ModelMapper();  
        modelMapper.getConfiguration().setFieldMatchingEnabled(true)  
                .setFieldAccessLevel(org.modelmapper.config.Configuration.AccessLevel.PRIVATE)  
        .setMatchingStrategy(MatchingStrategies.STRICT);  
    }  
    public ModelMapper get() {  
        return modelMapper;  
    }  
}
```
ModelMapper에 관한 정보 저장하는 MapperUtil.java
다른 클래스에서 호출해서 사용

TodoService.java
```
...
TodoService() {  
    this.dao = new TodoDAO();  
    this.modelMapper = MapperUtil.INSTANCE.get();  
}
...
public void register(TodoDTO todoDTO) throws Exception {  
    TodoVO todoVO = modelMapper.map(todoDTO, TodoVO.class);  
    log.info(todoVO);  
    dao.insert(todoVO);  
}
```
`TodoVO todoVO = modelMapper.map(todoDTO, TodoVO.class);` : 
DTO를 VO로 변환(TodoVO.class : TodoVO의 클래스 자체라고 생각하면 됨(Reflection))

여러 객체를 한 번에 변환도 가능
```
public List<TodoDTO> listAll() throws Exception {  
    List<TodoVO> voList = dao.selectAll();  
    log.info("voList-----------------");  
    log.info(voList);  
    List<TodoDTO> dtoList = voList.stream().map(  
            vo -> modelMapper.map(vo, TodoDTO.class)  
    ).collect(Collectors.toList());  
  
    return dtoList;  
}
```
voList는 TodoVO 객체들이 담긴 List

`voList.stream().map(vo -> modelMapper.map(vo, TodoDTO.class)).collect(Collectors.toList())` 
: List에 담긴 TodoVO들을 stream().map()을 사용, vo -> TodoDTO로 객체 맵핑
변환된 객체들을 List로 만들어서 dtoList로 받음

# 로그(log) 기능
### Log4j2
서버 로그를 찍어주는 오픈 소스 라이브러리

보안 레벨 설정
개발에 필요한 로그와 실제 운영시 필요한 로그
- 로그 레벨(Log level)
	FATAL > ERROR > WARN > INFO > DEBUG > TRACE
	일반적인 로그 레벨은 INFO, 하위로 갈 수록 출력 가능한 로그가 많음
- 어펜더(Appender)
	로그를 어떤 방식으로 기록하고 콘솔창에 출력할 것인지(파일로 출력할 지) 결정

Lombok의 경우 @Log4j2 제공하고 있음
-> 소스코드에서 간단히 로그 적용 가능

resources/log4j2.xml 생성, 내용은 log4j2 공식 문서 참조
```
<?xml version="1.0" encoding="UTF-8"?>  
<Configuration status="WARN">  
    <Appenders>        
	    <Console name="CONSOLE" target="SYSTEM_OUT">  
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>  
        </Console>  
    </Appenders>    
    <Loggers>        
	    <Root level="INFO">  
            <AppenderRef ref="CONSOLE"/>  
        </Root>  
    </Loggers>
</Configuration>
```

`<PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/> `
-> 패턴 레이아웃 설정
출력 결과: 
`11:15:47.068 [http-nio-8080-exec-6] INFO  com.ssg.webmvc.todo.TodoListController - /todo/list doGet 호출`

Loggers에는 로그에 대한 정보
-> 루트 레벨 : 현재 INFO


