#DB 
#Redis

---
참고: [https://digitalbourgeois.tistory.com/751#google_vignette](https://digitalbourgeois.tistory.com/751#google_vignette) [평범한 직장인이 사는 세상:티스토리]

## RAG와 Redis Vector Library

### RAG(Retrieval Augmented Generation)란?

**RAG**
검색 기반 정보 검색(Retrieval)과 생성형 AI(Generation)를 결합한 기술
단순히 LLM(예: GPT-4)만 사용하는 것이 아니라, **벡터 검색**을 통해 관련 정보를 찾아 모델의 응답을 보강하는 방식

**RAG의 핵심 원리:**
1. 사용자의 질문을 벡터화(Embedding)
2. RedisVL에서 관련 문서를 검색
3. 검색된 문서를 바탕으로 OpenAI GPT 모델이 답변 생성


**RedisVL**
Redis에서 벡터 데이터 검색을 쉽게 구현할 수 있도록 도와주는 라이브러리
기존 Redis는 키-값(Key-Value) 저장소로 많이 사용되었지만, 
벡터 검색(Vector Search)을 통해 AI 및 추천 시스템에서도 강력한 성능을 발휘

**RedisVL의 주요 기능:**
- Hugging Face 모델을 활용한 벡터 임베딩 지원
- HNSW 기반의 빠른 벡터 검색
- OpenAI API와의 손쉬운 통합

