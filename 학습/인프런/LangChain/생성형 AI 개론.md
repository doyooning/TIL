#LangChain 
#LLM 
#AI 

# 생성형 AI란?
학습 데이터의 분포를 모델링하여 텍스트, 이미지, 코드, 음성 등 **새로운 콘텐츠를 생성**하는 인공지능

**LLM(Large Language Model, 대규모 언어 모델)** 은 대규모 텍스트 코퍼스로 학습된 다음 토큰 예측 기반 모델로, ChatGPT, Claude, Gemini 등이 대표적 사례

### 모델 분류

| 구분 | 설명 | 대표 예시 |
|---|---|---|
| 판별 모델(Discriminative) | 입력에 대한 **분류 또는 예측** 수행 | 스팸 분류, 이미지 인식 |
| 생성 모델(Generative) | 데이터 분포를 학습해 **새로운 샘플 생성** | GPT, DALL·E, Stable Diffusion |

### LLM의 핵심 원리: 다음 토큰 예측
LLM은 주어진 문맥 다음에 등장할 확률이 가장 높은 **토큰(token)** 을 반복적으로 예측

번역, 요약, 추론, 코드 생성 등 복잡한 능력은 이 단순한 예측 과정의 반복에서 **창발(emergent)** 적으로 나타남

LLM은 매 시점마다 **현재까지의 문맥에서 다음 토큰(≈단어 조각)이 무엇일지** 확률 분포를 계산하고, 그중 하나를 선택

### 토큰(token)
영어 "I love AI"는 일반적으로 `["I", " love", " AI"]`로 분할되어 **3토큰**으로 처리

한국어 "안녕하세요"는 토크나이저에 따라 4~7토큰이 소비
동일 문장 기준으로 영어 대비 토큰 수가 더 많아 비용 측면에서 불리

### 할루시네이션(hallucination)
LLM이 사실과 다른 내용을 그럴듯하게 생성하는 현상
다음 토큰 예측 메커니즘은 사실 검증을 수행하지 않고 학습된 분포에서 가장 자연스러운 토큰을 선택하기 때문에, 정확성이 보장되지 않는 정보가 출력될 수 있음

이를 보완하기 위한 대표적인 기법이 외부 지식을 검색해 응답에 활용하는 **RAG(Retrieval-Augmented Generation, 검색 증강 생성)**


# 시작하기
### 클라이언트 설정
```python
from openai import OpenAI

client = OpenAI()  # OPENAI_API_KEY 자동 감지
MODEL = "gpt-4.1-mini"

response = client.chat.completions.create(
    model=MODEL,
    messages=[
        {"role": "system", "content": "당신은 친절한 한국어 AI 어시스턴트입니다."},
        {"role": "user", "content": "LLM이 무엇인지 한 문장으로 설명해주세요."},
    ],
)

print(f"[{MODEL}]")
print(response.choices[0].message.content)
# [gpt-4.1-mini]
# LLM은 대규모 언어 모델(Large Language Model)로, ...
```

### 메시지 역할(role)
- **system**: 모델의 페르소나와 행동 규칙을 정의
  (예: "당신은 금융 분야 전문가입니다.")
- **user**: 사용자의 질문 또는 지시 사항
- **assistant**: 이전 턴에서 모델이 생성한 응답
  멀티턴 대화 이력을 유지할 때 사용

# 구조화 출력 (JSON)
OpenAI 호환 API에서는 
`response_format={"type": "json_object"}` 
또는 JSON Schema 기반 옵션을 통해 구조화 출력을 강제할 수 있음

```python
r = client.chat.completions.create(
    model=MODEL,
    messages=[
        {
            "role": "system",
            "content": "사용자가 입력한 문장에서 의도를 분류해 JSON으로 응답하세요."
                       '필드: {"intent": "질문|요청|불만|칭찬", "sentiment": "긍정|중립|부정", "keywords": [최대 3개]}',
        },
        {
            "role": "user",
            "content": "...",
        },
    ],
    response_format={"type": "json_object"},
)

import json
data = json.loads(r.choices[0].message.content)
print(json.dumps(data, ensure_ascii=False, indent=2))
```

# 멀티턴 대화
이전 응답을 메시지 리스트에 누적하면 모델이 **대화 맥락을 유지**한 상태로 다음 응답을 생성

LangChain의 `Memory`와 LangGraph의 `Checkpointer`로 구현


