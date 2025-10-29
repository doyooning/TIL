#Java 
#Spring 

---
# Spring Web MVC (이어서)

### RedirectAttributes 와 리다이렉션
PRG패턴 구현에 필요한 RedirectAttributes 타입

스프링MVC의 경우 반환타입이 문자열이고 redirect: 로 시작하는 경우 리다이렉트 처리
RedirectAttributes는 리다이렉트에 필요한 쿼리 스트링을 구성하기 위한 객체
- addAttribute(키, 값) : 
- addFlashAttribute(키, 값) : 일회성 전달값


**컨트롤러 @GetMapping 메서드 리턴 타입**
- void
	주로 상황에 상관없이 동일한 화면 보여줄 때 사용
	컨트롤러의 @RequestMapping, @GetMapping 메서드에서 선언된 값을 그대로 뷰의 이름으로 사용
- String
	상황에 따라 다른 화면(결과값)을 보여줄 때 사용
	문자열의 경우 특별한 사용이 가능 ex) "redirect:/ex06" , "forward: "
	(forward: 브라우저의 URL은 고정되고 내부적으로 다른 URL로 처리하는 경우 - 잘 사용하지 X)



### Spring MVC 예외 처리
• @ControllerAdvice를 이용해서 처리 
• 예외에 따라서 @ExceptionHandler를 메서드에 활용

CommonExceptionAdvice로 예외 잡기
- AOP(Aspect Oriented Programming)의 포인트 컷 개념
	-> 프로그램이 상->하로 진행될 때 AOP는 횡으로 형성
	반드시 필요하진 않지만 품질 향상을 위해 구현하는 AOP 

CommonExceptionAdvice.java
```
// Spring Controller에서 발생하는 예외를 처리하는 일반적인 방식  
@ControllerAdvice  
@Log4j2  
public class CommonExceptionAdvice {  
    @ExceptionHandler(NumberFormatException.class)  
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    public String exceptNumber(NumberFormatException numberFormatException) {  
        log.error("NumberFormatException: " + numberFormatException.getMessage());  
        return "NumberFormatException !";  
    }  
  
    @ExceptionHandler(value = NoHandlerFoundException.class)  
    @ResponseStatus(HttpStatus.NOT_FOUND)  
    public String NotFound() {  
        return "custom404";  
    }  
}
```
NumberFormatException으로 인한 400 Bad Request -> exceptNumber 메서드 실행

NoHandlerFoundException으로 인한 404 Not Found -> custom404.jsp로 이동

web.xml의 servlet 태그 안에 추가
```
	<init-param>  
	    <param-name>throwExceptionIfNoHandlerFound</param-name>  
	    <param-value>true</param-value>  
	</init-param>  
	<load-on-startup>1</load-on-startup>
```

