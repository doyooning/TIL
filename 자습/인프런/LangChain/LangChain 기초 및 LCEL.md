#LangChain 
#LLM
#AI 

---
# Models
`init_chat_model("provider:model")` 한 함수로 모든 제공자의 모델을 통합 초기화 가능

가장 간단한 호출
```python
response = llm.invoke("LangChain v1.0 의 핵심 철학을 한 문장으로 설명하세요.")
print(response.content)
```

# Messages
대화의 기본 단위
메시지 타입이 `langchain.messages` 모듈에 통합되어 있음
`SystemMessage`, `HumanMessage`, `AIMessage`, `ToolMessage` 등의 클래스로 표현

```python
messages = [
    SystemMessage(content="당신은 사내 IT 도우미입니다. 간결하게 답하세요."),
    HumanMessage(content="VPN 연결이 자꾸 끊기는데 어떻게 해야 하나요?"),
]
reply = llm.invoke(messages)
print(reply.content)
print("\n메시지 타입:", type(reply).__name__)  # AIMessage
```

### AIMessage의 메타데이터
`AIMessage` 객체에는 응답 본문 외에도 토큰 사용량, 모델 식별자, 종료 사유 등의 메타데이터가 포함되어 있음

# Prompt Template
변수 기반 프롬프트
`ChatPromptTemplate`은 시스템 메시지와 사용자 메시지의 템플릿을 정의하고, 변수 치환을 통해 일관된 형태의 메시지 리스트를 생성

```python
prompt = ChatPromptTemplate.from_messages([
    ("system", "당신은 {role} 입니다. 친절하고 전문적으로 답변하세요."),
    ("human", "{question}"),
])
```

# Output Parser
응답 변환
`StrOutputParser`는 가장 단순한 형태의 파서로, `AIMessage` 객체에서 문자열 `content`만 추출

# LCEL
파이프 연산자 기반의 체인 구성
**LCEL(LangChain Expression Language)** 은 
`|` 파이프 연산자를 사용해 컴포넌트를 연결하고, 
체인을 **선언적으로 정의**하는 표현 방식

코드 표현
```
prompt | llm | parser
```

### 파이프 연산자의 이점
1. **가독성**: "프롬프트 → 모델 → 파서"의 처리 흐름이 코드에 그대로 드러남
2. **재사용성**: `prompt | llm` 부분을 변수로 분리하면 서로 다른 파서와 조합 가능
3. **공통 인터페이스**: 각 Runnable은 `.invoke()`, `.stream()`, `.batch()`를 자동 지원

### 스트리밍 호출
LLM은 토큰 단위로 응답을 생성
`.stream()`을 사용하면 토큰이 생성되는 즉시 클라이언트로 전달받을 수 있음

### 배치 호출
`.batch()`를 사용하면 여러 입력을 한 번의 호출 단위로 묶어 병렬 처리 가능
다수의 독립적인 입력을 처리할 때 전체 처리 시간을 단축할 수 있음

# Runnable 컴포넌트
LCEL 체인을 구성하는 모든 부품은 공통적으로 `Runnable` 인터페이스를 따름

|컴포넌트|역할|
|---|---|
|`RunnableSequence`|파이프라인 (`\|` 연산자로 자동 생성)|
|`RunnableParallel`|여러 체인을 병렬 실행|
|`RunnablePassthrough`|입력을 그대로 전달 (딕셔너리 필드 채울 때)|
|`RunnableLambda`|일반 파이썬 함수를 Runnable로 래핑|

### RunnablePassthrough + RunnableParallel
동일한 입력에 대해 **여러 관점의 응답을 동시에** 생성하는 패턴
각 분기 체인이 독립적으로 실행되어 직렬 호출 대비 응답 시간 단축 가능

```python
pro_prompt  = ChatPromptTemplate.from_messages([("system", "긍정적 관점으로 답하세요."), ("human", "{q}")])
con_prompt  = ChatPromptTemplate.from_messages([("system", "비판적 관점으로 답하세요."), ("human", "{q}")])

pros_chain = pro_prompt | llm | parser
cons_chain = con_prompt | llm | parser

debate = RunnableParallel(
    question=RunnablePassthrough(),   # 입력의 q를 그대로 통과
    pros=pros_chain,
    cons=cons_chain,
)

out = debate.invoke({"q": "재택근무를 전면 도입하는 것은 좋은가?"})
print("PROS:", out["pros"][:100], "...")
print("CONS:", out["cons"][:100], "...")
```

### RunnableLambda
Python 함수 삽입
체인 중간에 전처리 또는 후처리 단계를 추가할 때 사용

# 멀티턴 대화를 위한 메시지 누적
보다 복잡한 메모리 관리(체크포인팅, 세션 단위 상태 저장 등)는 LangGraph의 `MemorySaver` 체크포인터에서 확인

