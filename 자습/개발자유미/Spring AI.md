#Java 
#Spring 

---
### 개요
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

---
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

---
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


**멀티턴 : Service단 Chat 메소드 수정**

- domain > openai > repository > ChatRepository.java
`public interface ChatRepository extends JpaRepository<ChatEntity, Long> { }`

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

---
### 히스토리 구현 JPA

**JPA 의존성 추가 및 설정**

**- build.gradle**

implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
기존에 MySQL 드라이버와 DB연결은 진행했기 때문에 JPA만 추가

**application.properties : ddl 설정 추가**

```
spring.jpa.hibernate.ddl-auto=update 
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl 
spring.jpa.show-sql=true
```
dld-auto=update 적용되었는지 확인

### Entity, Repository 생성

- domain > openai > entity > ChatEntity.java, 
- domain > openai > repository > ChatRepository.java

### Service단 채팅 내역 추가

- domain > openai > service > OpenAIService.java : 의존성 추가
- domain > openai > service > OpenAIService.java : 챗 스트림 메소드

- domain > openai > service > OpenAIService.java : 의존성 추가

**조회 API 작성**

저장한 전체 대화 내역을 조회하는 API

- domain > openai > repository > ChatRepository.java : 커스텀 메소드 추가
- `List<ChatEntity> findByUserIdOrderByCreatedAtAsc(String userId);`

domain > openai > service > ChatService.java

```java
@Service 
public class ChatService { 
	private final ChatRepository chatRepository; 
	public ChatService(ChatRepository chatRepository) { 
		this.chatRepository = chatRepository; 
	} 
	@Transactional(readOnly = true) 
	public List<ChatEntity> readAllChats(String userId) 
	{ return chatRepository.findByUserIdOrderByCreatedAtAsc(userId); } 
}
```

api > ChatController.java

`private final ChatService chatService;` 추가
```java
@ResponseBody 
@PostMapping("/chat/history/{userid}") 
public List<ChatEntity> getChatHistory(@PathVariable("userid") String userId) { 
	return chatService.readAllChats(userId); 
}
```

---
### ChatClient DSL Wrapper

**- 기존 사용 : OpenAI 기준**
`private final OpenAiChatModel openAiChatModel;`

ChatClient를 활용하면 스프링 AI의 다양한 기능들을 함께 활용 가능
openAiChatModel을 ChatClient에 Wrapping해서 사용

**ChatClient**
`ChatClient chatClient = ChatClient.create(openAiChatModel);`

**Bean 등록 예시**

```java
@Configuration 
public class ChatClientConfig {
	 
	@Bean 
	public ChatClient openAiChatClient(OpenAiChatModel chatModel) { 
		return ChatClient.create(chatModel); 
	} 
	
	@Bean 
	public ChatClient anthropicChatClient(AnthropicChatModel chatModel) { 
		return ChatClient.create(chatModel); 
	} 
}
```

**Service 챗 스트림 메소드 수정**

domain > openai > service > OpenAIService.java
기존 openAiChatModel을 리턴했던 부분을 chatClient를 사용해서 바꿔줌

```java
// 요청 및 응답  
return chatClient.prompt(prompt)  // 프롬프트 메기고
        .stream()  // 스트림 사용해서
        .content()  
        .map(token -> {  
            responseBuffer.append(token);  // responseBuffer : StringBuilder
            return token;  // 버퍼에 담긴 응답 메시지 토큰들을 매핑
        })  
        .doOnComplete(() -> {  
            // chatMemory 저장  
            chatMemory.add(userId, new AssistantMessage(responseBuffer.toString()));  
            chatMemoryRepository.saveAll(userId, chatMemory.get(userId));  
  
            // 전체 대화 저장용  
            ChatEntity chatAssistantEntity = new ChatEntity();  
            chatAssistantEntity.setUserId(userId);  
            chatAssistantEntity.setType(MessageType.ASSISTANT);  
            chatAssistantEntity.setContent(responseBuffer.toString());  
  
            chatRepository.saveAll(List.of(chatUserEntity, chatAssistantEntity));  
        });
```


**ChatClient를 사용하는 이유**

ChatClient로 감싼 객체를 통해 아래 기능들을 쉽게 추가 가능
- ObservationRegistry : 로깅
- tools : LLM에게 사용할 툴을 붙여줌
- advisors : RAG
- entity : 응답 데이터 객체 파싱 (call 메소드만)
- 추상화 : 모델 변경되어도 동일한 메소드

즉, 기존의 OpenAiChatModel 객체만으로는 가질 수 없는 여러 기능을 붙일 수 있음

---
**LLM 응답 구조화**

LLM 응답을 자연어스러움에서 구조화하여 자바스러움의 객체로 응답을 받는 방법

**응답을 객체로 담기 : 예상 시나리오**

LLM에게 데이터를 만들라고 요청하는 경우가 많음

정말 간단하게 특정 국가명을 보내면 국가에 속한 도시 N개를 데이터로 받는 로직
DTO에 데이터가 담도록 구성하고 싶다면 Structured Output을 활용하면 됨

담을 DTO : 
```
public record CityResponseDTO(List<String> city) { }
```

###### (추가) ​Record란?

record는 “불변 데이터 객체”를 간단하게 만들고 싶을 때 쓰는 문법
equals/hashCode/toString/getter/생성자까지 자동 생성

1) **DTO, VO 같은 단순 데이터 전달용 객체**

예: API 응답, 설정 값, DB 조회 결과 등  
필드만 있고 로직은 거의 없는 객체

2) **불변 객체를 만들고 싶을 때**

record는 기본적으로 모든 필드가 `final`이고, 생성 후 값이 바뀌지 않음
따라서 스레드 안전성도 자연스럽게 확보됨

3) **데이터 중심의 구조를 표현할 때**

예: 좌표, Range, Pair, Key-Value 등  
값 자체가 객체의 정체성인 경우 record가 더 자연스러움

4) **Lombok을 쓰기 싫거나 줄이고 싶을 때**

`@Getter`, `@RequiredArgsConstructor`, `@EqualsAndHashCode` 같은 걸  
record 하나로 대체 가능


**Output 컨버터 등록**

문장 형태로 받은 데이터를 DTO로 맞춰 넣기 위한 컨버터가 요구됨
컨버터는 다양하게 제공됨

**AbstractConversionServiceOutputConverter<>**

추상 클래스, 상속해서 커스텀 컨버터 생성

**AbstractMessageOutputConverter< T >**

메시지 기반 변환 추상 클래스, 상속해서 커스텀 컨버터 생성

**BeanOutputConverter< T >**

DTO/Record/Bean 클래스 기반 자동 JSON 컨버터

**MapOutputConverter**

Map<String, Object> 기반 자동 컨버터

**ListOutputConverter**

List< T > 기반 자동 컨버터

**적용**

domain > openai > service > OpenAIService.java
 원래 컨버터 등록을 해야하지만 (DTO라 Bean 컨버터) 자동 타입 추론이 되기 때문에 명시하지 않아도 사용 가능 - 현재 미사용중인 InMemory 방식에 두현
 
```java
public CityResponseDTO generate(String text) { 
	ChatClient chatClient = ChatClient.create(openAiChatModel); // 메시지 
	SystemMessage systemMessage = new SystemMessage(""); 
	UserMessage userMessage = new UserMessage(text); 
	AssistantMessage assistantMessage = new AssistantMessage(""); // 옵션 
	OpenAiChatOptions options = OpenAiChatOptions.builder() 
			.model("gpt-4.1-mini") 
			.temperature(0.7) 
			.build(); 
			// 프롬프트 Prompt prompt = new Prompt(List.of(systemMessage, userMessage, assistantMessage), options); 
			// 요청 및 응답 return chatClient.prompt(prompt) 
			.call() 
			.entity(CityResponseDTO.class); }
```

원리 자체는 프롬프트 보낼 때, 내 포맷대로 만들어 주세요 → 이후 포맷으로 파싱
현재 1.0.0이 나온지 얼마되지 않아 공식 문서 코드도 이상함(...)

---
### Agent와 Tools
​
에이전트란 목표가 주어지면, 그것을 이행하기 위해 LLM이 스스로 판단하여 행동을 수행하는 방식

LLM이 문제를 해결하는 과정에서 외부의 도움이 필요하다고 판단하면, 필요한 도구(Tool)를 사용해 작업을 수행

**- 툴 샘플**

- 검색 툴
- DB 조회 툴
- 계산 툴
- 코드 실행 툴
- 시각화 툴
- 등등

**Tool 사용 동작 원리**

![[Pasted image 20251207144830.png]]
1. 사용자의 프롬프트와 툴 목록을 함께 보내줌
2. LLM API가 사용자의 질문과 툴 목록을 확인 후, 특정 툴을 사용하겠다고 콜백을 줌
3. 툴을 활용해서 데이터를 처리하거나 가져옴 (스프링 AI 어플리케이션에서 툴 실행)
4. 툴 처리 완료
5. 툴 실행 결과를 다시 LLM API에 제공
6. LLM API 최종 응답

**툴 등록**

```java
public class ChatTools {  
    @Tool(description = "User personal information : name, age, address, phone, etc")  
    public UserResponseDTO getUserInfoTool() {  
        return new UserResponseDTO("홍길동", 20L, "서울특별시 강남구 강남대로 1", "010-0000-0000", "00000");  
    }  
}
```


**ChatClient 호출시 등록**

```java
return chatClient.prompt(prompt) 
		.tools(new ChatTools()) 
		.stream() 
		.content() 
		// 이하 생략
```

---
### RAG : Retrieval Augmented Generation

**RAG**
LLM에게 우리 도메인의 지식을 부여하기 위해 “사용자의 프롬포트”에 유사한 문서 N개(문서는 특정 도메인 데이터)를 뽑아 프롬프트에 붙여 보내는 기법

**스프링 AI : Advisor**

스프링 AI의 ChatClient advisors() 메소드는 RAG를 쉽게 통합할 수 있는 기능을 제공
이 advisors() 메소드에는 VectorStore라는 객체를 넣어주어야 하는데 관련된 아래 의존성이 요구됨

build.gradle > dependencies
`implementation 'org.springframework.ai:spring-ai-advisors-vector-store'`

위 의존성을 통해 advisor API를 추가하면 통합할 수 있는 어댑터는 활성화 되지만, 결국 RAG를 위한 DB 필요
-> ElasticSearch 활용...?

