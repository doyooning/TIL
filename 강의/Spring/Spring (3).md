#Java 
#Spring 

---
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