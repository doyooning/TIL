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

### Controller 구현 및 View 구현

1. @RestController 클래스 전체에 @ResponseBody가 자동 적용됨 → 리턴 값이 바로 JSON
    
2. @RequestMapping("/api/board") 컨트롤러의 기본 경로. GET /api/board, GET /api/board/{bno}, POST /api/board
    
3. @RequestBody BoardVO request
    
4. Axios가 보낸 JSON → Jackson이 자동으로 BoardVO에 매핑.
    예: { "title": "제목", "content": "내용", "writer": "신세계" }
5. @ResponseEntity< T >
	HTTP 상태코드 + 바디를 함께 제어(객체 + 상태코드 + 헤더) - ResponseEntity.ok(vo) → 200 OK + JSON - ResponseEntity.status(HttpStatus.CREATED).body(saved) → 201 Created + JSON - ResponseEntity.notFound().build() → 404 - ResponseEntity.status(HttpStatus.BAD_REQUEST).build() → 400
	예) `return ResponseEntity.ok(boardService.getList());`
	
	언제 사용하는 것이 좋은가? 
	-  등록 후 201 Created 응답을 주고 싶을 때 
	-  실패 시 400, 404, 500 등 명확한 상태코드를 제어하고 싶을 때 
	-  헤더(Location, Set-Cookie, Content-Disposition 등)를 커스터마이징할 때

```java
@PostMapping
public ResponseEntity<BoardVO> create(@RequestBody BoardVO vo) {
	if (vo.getTitle() == null) {
		return ResponseEntity.badRequest().build(); // 400 Bad Request
	}
	boardService.register(vo);
	return ResponseEntity.status(201).body(vo);
}
```








