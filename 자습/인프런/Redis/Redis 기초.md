#DB 
#Redis

---

## Redis

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
: 대충 logger가 없다는 내용




