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
###### Prompt, ChatOption 클래스

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



