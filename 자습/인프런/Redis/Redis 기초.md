#DB 
#Redis

---
### Lettuce 클라이언트
Redis의 Java 공식 라이브러리
Java 환경에서 Redis와 통신할 수 있는 비동기 클라이언트

Netty 기반, 고성능 작업 가능
Spring Framework의 Spring Data Redis 모듈에서 Lettuce 라이브러리 이용중

## Redis
Redis의 가장 큰 특징?
-> 빠른 속도, 사용 간단함, 자료형 풍부
메모리에 데이터 보관, 디스크 I/O 발생하지 않아 빠른 데이터 로드 가능
하지만 기본적으로 데이터를 영구보관하지 않기 때문에 관계형 DB와 함께 사용하는 것이 좋음

Redis를 docker에 설치

Redis Container 실행 - 컨테이너 ID 확인, 포트 6379

Redis Cli 실행
-> 터미널에서 입력:
`docker exec -it 컨테이너ID redis-cli`

127.0.0.1:6379 뜨면 성공

Connection Health 체크:
`ping` -> pong 뜨면 성공


Redis에서 제공하는 기초 명령어
보통 저장 명령어, 조회 명령어로 구분

ex) 저장 : set, 조회 : get, 멀티저장 : mset, 멀티조회 : mget
저장 데이터가 숫자일 때
증가시킬 때 : incr
감소시킬 때 : decr

간단하게 테스트
```java
public class RedisLettuceStringTest {  
    @Test  
    public void setGet() {  
        RedisURI redisURI = RedisURI.builder()  
                .withHost("localhost")  
                .withPort(6379)  
                .withDatabase(0) // 0 ~ 15  
                .build();  
        RedisClient redisClient = RedisClient.create(redisURI);  
        StatefulRedisConnection<String, String> connection = redisClient.connect();  
        RedisCommands<String, String> redisCommands = connection.sync();  
  
        String key = "lettuce:string";  
        String value = "hello";  
        redisCommands.set(key, value);  
  
        String get = redisCommands.get(key);  
        System.out.println("get = " + get);  
    }  
}
```
-> get = hello

Test는 성공하나, 오류 문구가 있음
: slf4j 로거에 구현체가 없어서 나오는 에러

로거로 logback-classic 사용

resources > logback.xml 만들어준다.
DEBUG -> INFO로 올림

TTL = Time To Live
따로 설정하지 않으면 -1
-1은 만료시간이 없기 때문에 Redis 서버가 살아있는 한 계속 유지됨

양수인 경우에는 데이터가 살아있는 시간을 초단위로 나타낸 것
-2는 키가 존재하지 않는다는 뜻

TTL 설정 및 변경
```java
Long ttl = redisCommands.ttl(key);  
System.out.println("ttl = " + ttl);  
  
redisCommands.expire(key, Duration.ofMinutes(1));  
System.out.println("ttl = " + redisCommands.ttl(key));

SetArgs setArgs = SetArgs.Builder  
//                .ex(90)  
		.keepttl(); // redis 6.0 부터 가능, ex와 keepttl 같이 쓰면 오류 발생

redisCommands.set(key, value + "_new", setArgs);  
System.out.println("ttl = " + redisCommands.ttl(key));
```

Redis 6.2부터 새로 추가된 명령어

`getdel()` : 조회와 동시에 데이터를 삭제
`getex()` : 조회를 하면서 TTL을 새로 설정하고 싶은 경우

```java
String getdel = redisCommands.getdel(key);  
System.out.println("getdel = " + getdel);  
System.out.println("ttl = " + redisCommands.ttl(key));  
  
GetExArgs getExArgs = GetExArgs.Builder.ex(120);  
String getex = redisCommands.getex(key, getExArgs);  
System.out.println("getex = " + getex);  
System.out.println("ttl = " + redisCommands.ttl(key));
```

Redis 사용이 완료되었으면 Connection 객체와 Redis Client 연결 해제
```java
connection.close();  
redisClient.shutdown();
```


