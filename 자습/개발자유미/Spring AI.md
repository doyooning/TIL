#Java 
#Spring 

---
**Spring AI**

: LLM API를 활용한 여러 웹 시스템 (챗봇, 멀티모달, RAG)을 쉽게 구축하도록 도와주는 스프링 기반의 모듈

현재 AI 진영에서는 LangChain/LangGraph와 같은 파이썬 모듈의 편의성으로 자바/스프링 기반의 구현이 줄어들고 있음

자바/스프링 또한 API 호출을 하는 기초 코드 부터 작성해 일련의 과정들을 수행할 수 있지만, 프레임워크 대비 많이 번거로운데 이 문제를 해결하기 위해 2024년 스프링은 스프링 AI라는 모듈을 출시하게 됨

공식문서
[https://docs.spring.io/spring-ai/reference/index.html](https://docs.spring.io/spring-ai/reference/index.html)

AI와 연관된 도구들을 쉽게 통합하도록 하는 프레임워크
- LLM API 호출
	- 챗모델 : Anthropic, Azure, OpenAI, DeepSeek
	- 임베딩 모델
	- 이미지 모델
	- 오디오 모델
- 챗 메모리 : 채팅 히스토리 관리
- 툴 호출
- MCP
- RAG
- Vector DB
	- ES
	- Neo4j
	- PostgresML


**멀티턴 및 히스토리 (multi-turn)**

여러 대화간의 문맥을 고려해 앞선 내용에 이어 응답할 수 있는 멀티턴 기능,
동일한 페이지에 재접속 할 경우 그동안의 대화 내용을 보여줄 수 있는 히스토리 기능

참고자료
[https://www.skelterlabs.com/blog/multiturn-rag](https://www.skelterlabs.com/blog/multiturn-rag)

###### 멀티턴 : ChatMemoryRepository와 문제점
​
**- 멀티턴 구현 원리는?**

멀티턴 기능을 구현하기 위해선 결국 과거에 챗봇과 나눴던 QnA를 다시 프롬프트에 담아야 함 
-> 히스토리 저장도 필요

**- 스프링 AI에서 제공**

그 기능을 스프링 AI에서 제공 
과거 N개의 대화 데이터를 ChatMemoryRepository 인터페이스에 담을 수 있도록 제공함

ChatMemoryRepository, 구현체는 InMemory, JDBC를 기본 제공

**- 문제점**

과거 N개의 대화 이력을 ChatMemoryRepository를 통해 관리할 수 있으나, 멀티턴이라고 해서 과거 모든 데이터를 다룰 순 없음
-> ==max_prompt_size==, ==비용==, ==딜레이== 문제
이 문제로 대략 과거 몇 개 또는 요약 데이터를 활용해 멀티턴을 구현

이렇게 된다면 페이지 로딩을 위해 과거 모든 대화 히스토리를 불러와야하는 상황에 문제가 생김

**- 해결책**

2개의 저장소를 사용
: 전체 히스토리 저장용, 멀티턴용 ChatMemoryRepository

**멀티턴 : ChatMemoryRepository Bean 등록**

구현체 크게 2종류 (InMemory, JDBC) 
대화 내역 저장을 위한 각각의 Bean 등록 방법

(기본적으로 스프링 AI 의존성을 추가한다면 InMemory Bean이 자동 활성화, 수동 등록해도 무방)
-> InMemory 방식은 생략(비추)


**- JDBC 구현체 사용을 위한 의존성 추가 (+MySQL)**

```
implementation 'org.springframework.ai:spring-ai-starter-model-chat-memory-repository-jdbc' 
runtimeOnly 'com.mysql:mysql-connector-j'
```

application.properties 변수 추가

```
spring.datasource.url=jdbc:mysql://아이피:3306/디비?serverTimezone=UTC&useSSL=false spring.datasource.username=root 
spring.datasource.password=비밀번호 
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.ai.chat.memory.repository.jdbc.initialize-schema=never
```

`spring.ai.chat.memory.repository.jdbc.initialize-schema=never`
: 테이블 자동생성은 never로 설정하는 게 좋음(오류 잦음)

**- 테이블 생성**

테이블 생성 오류가 자주 발생하므로 직접 생성 권장
[https://github.com/spring-projects/spring-ai/tree/main/memory/repository/spring-ai-model-chat-memory-repository-jdbc/src/main/resources/org/springframework/ai/chat/memory/repository/jdbc](https://github.com/spring-projects/spring-ai/tree/main/memory/repository/spring-ai-model-chat-memory-repository-jdbc/src/main/resources/org/springframework/ai/chat/memory/repository/jdbc)

```sql
CREATE TABLE IF NOT EXISTS SPRING_AI_CHAT_MEMORY ( 
	`conversation_id` VARCHAR(36) NOT NULL, 
	`content` TEXT NOT NULL, 
	`type` ENUM('USER', 'ASSISTANT', 'SYSTEM', 'TOOL') NOT NULL, 
	`timestamp` TIMESTAMP NOT NULL, 
	INDEX `SPRING_AI_CHAT_MEMORY_CONVERSATION_ID_TIMESTAMP_IDX` (`conversation_id`, `timestamp`) 
);
```


```java
public Flux<String> generateStream(String text) {  
  
    // 유저&페이지별 ChatMemory를 관리하기 위한 key (우선은 명시적으로)  
    String userId = "dynii1923" + "_" + "3"; 
  
    ChatMemory chatMemory = MessageWindowChatMemory.builder()  
            .maxMessages(10)  
            .chatMemoryRepository(chatMemoryRepository)  
            .build();  
    chatMemory.add(userId, new UserMessage(text)); // 신규 메시지도 추가  
  
    // 옵션  
    OpenAiChatOptions options = OpenAiChatOptions.builder()  
            .model("gpt-4.1-mini")  
            .temperature(0.7)  
            .build();  
  
    // 프롬프트  
    Prompt prompt = new Prompt(chatMemory.get(userId), options);  
  
    // 응답 메시지를 저장할 임시 버퍼  
    StringBuilder responseBuffer = new StringBuilder();  
  
    // 요청 및 응답  
    return openAiChatModel.stream(prompt)  
            .mapNotNull(response -> {  
                String token = response.getResult().getOutput().getText();  
                responseBuffer.append(token);  
                return token;  
            })  
            .doOnComplete(() -> {  
  
                chatMemory.add(userId, new AssistantMessage(responseBuffer.toString()));  
                chatMemoryRepository.saveAll(userId, chatMemory.get(userId));  
            });  
}
```

