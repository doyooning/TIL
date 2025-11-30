#Spring 

---
Sionic AI 블로그 참조

### Spring AI란?
Spring 개발자들을 위한 LLM 통합 도구
OpenAI, Anthropic 등 다양한 벤더 모델들을 하나의 공통된 방식으로 다룰 수 있게 도와주는 프레임워크

###### 장점
1. 익숙한 방식으로 AI 활용
	Spring Boot 기반
2. 여러 모델을 같은 코드로 한 번에 관리
	다양한 모델을 일관되게 요청할 수 있음
3. 벡터 저장소를 기본으로 제공
	텍스트를 청크로 나누고, 벡터화해서 저장하는 등의 작업을 기본으로 구성
	추상화가 잘 되어 있어 유연한 설계 가능
4. 최신 기능들을 지원
	MCP, Functional Calling, Prompt Template 등 지원

### Spring AI의 추상화 방식과 구조
###### 프롬프트 관리 : Prompt, ChatOption 클래스

[Prompt](https://docs.spring.io/spring-ai/reference/api/prompt.html) 클래스는 Spring AI에서 모델에 보낼 메시지와 모델 파라미터 옵션 ChatOptions을 감싸는 역할
```java
public class Prompt implements ModelRequest<List<Message>> { 
	private final List<Message> messages; 
	private ChatOptions modelOptions; 
	
	@Override 
	public ChatOptions getOptions() {...} 
	
	@Override 
	public List<Message> getInstructions() {...} 
}
```
-> Prompt는 '어떤 메시지를 보낼지' 와 '어떤 옵션으로 보낼지' 를 함께 담고 있는 구조

[ChatOptions](https://docs.spring.io/spring-ai/reference/api/chatmodel.html#_chat_options) 는 LLM 호출 시 사용할 다양한 파라미터를 정의한 인터페이스 
대부분의 LLM에서 공통으로 사용될 수 있는 옵션들만 포함
```java
public interface ChatOptions extends ModelOptions { 
	String getModel(); 
	Float getFrequencyPenalty(); // frequencyPenalty 
	Integer getMaxTokens(); // maxTokens 
	Float getPresencePenalty(); // presencePenalty 
	List<String> getStopSequences(); // stopSequences 
	Float getTemperature(); // temperature 
	Integer getTopK(); // topK 
	Float getTopP(); // topP 
	ChatOptions copy(); 
}
```
ChatOptions가 제공하는 속성 (`maxTokens`, `temperature`, `stopSequences`)은 벤더 간 자동 변환됨
ex) OpenAI `stop` vs Anthropic `stop_sequences` 같은 차이를 Spring AI가 알아서 처리함

```kotlin
import org.springframework.ai.chat.prompt.ChatOptions 
val openAIChatOptions = ChatOptions.builder() 
		.model("gpt-3.5-turbo") 
		.temperature(0.7) 
		.stopSequences(listOf("\n")) // OpenAI의 'stop' 매개변수로 자동 변환 
		.build() 
val anthropicChatOptions = ChatOptions.builder() 
		.model("claude-3-7-sonnet-20250219") 
		.temperature(0.7) 
		.stopSequences(listOf("\n")) // Anthropic의 'stop_sequences' 매개변수로 자동 변환 
		.build()
```

#### Spring AI의 주요 추상화 계층 : ChatModel

Spring AI는 ChatModel이라는 핵심 컴포넌트를 기반으로 작동함
`ChatModel` 은 LLM과의 기본적인 상호작용을 담당하는 인터페이스

``` kotlin
public interface ChatModel extends Model<Prompt, ChatResponse> { 
	default String call(String message) {...} 
	
	@Override 
	ChatResponse call(Prompt prompt); 
}
```

`ChatModel` 인터페이스를 구현한 클래스 -> [ChatResponse](https://docs.spring.io/spring-ai/reference/1.0/api/chatmodel.html#ChatResponse)라는 공통된 응답 객체로 리턴 
이 안에는 모델의 출력 메시지, 사용된 프롬프트, 모델 파라미터, 응답 시간 등의 메타 정보 포함 
후처리나 로깅, 디버깅 시에도 유용하게 활용 가능

#### 내부 동작 순서

Spring AI의 ChatModel 인터페이스를 구현한 클래스는 내부적으로 다음과 같은 과정을 거침

1. 입력으로 받은 Prompt를 벤더의 API 형식에 맞게 변환
2. 변환된 메시지를 사용하여 벤더의 API를 호출
3. 벤더로부터 받은 응답을 ChatResponse 형식으로 변환하여 반환

#### Spring AI 구현 코드

Spring AI로 OpenAI와 Anthropic 모델을 호출할 수 있도록 설정, 모델별로 ChatService에서 호출하는 구조
```kotlin
@Service 
class ChatService( 
	private val openAiApi: OpenAiApi, 
	private val anthropicApi: AnthropicApi 
) {
	fun getOpenAiResponse(userInput: String, stop: List<String>, temperature: Double): ChatResponse { 
	
		// 메시지 구성 
		val messages = listOf( 
			SystemMessage("You are a helpful assistant."), 
			UserMessage(userInput) 
		) 
		
		// 챗 옵션 설정 
		val chatOptions = ChatOptions.builder() 
			.model("gpt-3.5-turbo") 
			.temperature(temperature) 
			.stopSequences(stop) 
			.build() 
			
		// 프롬프트 생성 
		val prompt = Prompt(messages, chatOptions) 
		
		// 챗 모델 생성 
		val chatModel = OpenAiChatModel.builder() 
			.openAiApi(openAiApi) 
			.build() 
			
		// 챗 모델 호출 
		return chatModel.call(prompt) 
	}
	
	fun getAnthropicResponse(userInput: String, stop: List<String>, temperature: Double): ChatResponse { 
		val messages = listOf( 
			SystemMessage("You are a helpful assistant."),
			UserMessage(userInput) 
		) 
		
		val chatOptions = ChatOptions.builder() 
			.model("claude-3-7-sonnet-20250219") 
			.temperature(temperature) 
			.stopSequences(stop) 
			.build() 
			
		val prompt = Prompt(messages, chatOptions) 
		val chatModel = AnthropicChatModel.builder() 
			.anthropicApi(anthropicApi) 
			.build() 
		
		return chatModel.call(prompt) 
	}
}

fun main() { 

	// API 키 설정 
	val openAiApiKey = System.getenv("OPENAI_API_KEY") 
	val anthropicApiKey = System.getenv("ANTHROPIC_API_KEY") 
	
	// API 클라이언트 생성 
	val openAiApi = OpenAiApi.builder() 
		.apiKey(openAiApiKey) 
		.build() 
	val anthropicApi = OpenAiApi.builder() 
		.apiKey(anthropicApiKey) 
		.build() 
		
	// ChatService 생성 
	val chatService = ChatService( 
		openAiApi = openAiApi, 
		anthropicApi = anthropicApi 
	) 
	
	// OpenAI 호출 
	val openAiResponse = chatService.getOpenAiResponse( 
		userInput = "Tell me a joke.", 
		stop = listOf("\n", "END"), 
		temperature = 0.5 
	) 
	println("OpenAI Response: $openAiResponse") 
	
	// Anthropic 호출 
	val anthropicResponse = chatService.getAnthropicResponse( 
		userInput = "Tell me a joke.", 
		stop = listOf("\n", "END"), 
		temperature = 0.5 
	)
	println("Anthropic Response: $anthropicResponse") 
}

```

OpenAI와 Anthropic 각각의 API 클라이언트를 주입받아 독립적으로 구성됨
사용자 입력(`userInput`)과 프롬프트 옵션(`stop`, `temperature`)으로 Prompt 객체를 만든 뒤 모델들을 호출
ChatResponse에는 모델의 응답 결과와 메타 정보가 담겨 반환됨

