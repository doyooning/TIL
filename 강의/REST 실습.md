#Java 
#Spring 

---
### REST 방식 구현

```
@RestController  
@RequestMapping("/replies")  
public class ReplyController {  
    @GetMapping("/sample")  
    public Map<String, String> sample() {  
  
        return Map.of("value1", "AAA", "value2", "BBB");  
    }  
}
```
컨트롤러를 @RestController로 선언해주고
Get 방식 -> Map을 출력

개발자 도구에서 Content-Type에서 application/json임을 확인
Collection과 같은 객체들은 json으로 변환됨

문자열은 text/html로 취급됨
json으로 변환해주려면 `produces = MediaType.APPLICATION_JSON_VALUE` 추가

```
@RestController  
@RequestMapping("/replies")  
public class ReplyController {  
    @GetMapping(value = "/sample", produces = MediaType.APPLICATION_JSON_VALUE)  
    public String sample() {  
  
        return "HELLO";  
    }  
}
```

