#MOC
#CCDV-F
#Claude

---

# CCDV-F 자격증 MOC

Anthropic **Claude Certified Developer – Foundations (CCDV-F)** 시험 대비 노트 허브입니다.
Claude API와 개발 플랫폼으로 **프로덕션 수준의 애플리케이션·에이전트를 만들고 통합·배포하는 능력**을 검증하는 시험입니다.

> 새 노트를 추가할 때마다 이 MOC에도 링크를 추가해주세요.

## 시험 개요

| 항목 | 내용 |
| --- | --- |
| 정식 명칭 | Claude Certified Developer – Foundations |
| 시험 코드 | CCDV-F |
| 문항 수 | **53문항** (객관식 + 복수 응답, 문항마다 선택 개수 명시) |
| 시험 시간 | **120분** |
| 합격선 | 100~1,000 스케일 중 **720점** |
| 응시 비용 | **$125 USD** (회차당) |
| 응시 방법 | **Pearson VUE** 온라인 감독 또는 시험센터 |
| 자격 유효기간 | 12개월 |
| 결과 | 합격/불합격 + 스케일 점수 + 도메인별 정답률 |
| 재응시 | 실패 후 **14일 → 30일 → 90일** 대기, 12개월당 최대 **4회** |
| 권장 배경 | 엔지니어링 경력 1~5년, Claude/LLM 실무 6개월 이상, Python 또는 TypeScript, REST API·CLI 숙달 (권장일 뿐 필수 아님) |

> ⚠️ 도메인 비중·시험 형식은 공식 시험 가이드(파트너 전용)를 정리한 2차 출처 기준입니다. **개념 내용 자체는 전부 공식 문서**(platform.claude.com/docs, code.claude.com/docs)에서 확인했습니다.
> 공식 **Partner Academy prep course의 학습 목표**로 출제 범위를 교차 검증한 결과는 [[11 공식 커리큘럼 매핑]]에 정리했습니다.

> Anthropic 자격증은 4종입니다: **CCA-F**(Associate, $99) · **CCDV-F**(Developer, $125) · **CCAR-F**(Architect Foundations, $125) · **CCAR-P**(Architect Professional, $175).
> CCA-F는 Architect가 아니라 **Associate**입니다 — 2차 자료에서 자주 혼동됩니다.

## 출제 도메인과 비중

| # | 도메인 | 비중 | 세부 영역 | 노트 |
| --- | --- | --- | --- | --- |
| D2 | **Applications and Integration** | **33.1%** | 요구사항 이해 3.4 / 시스템 생명주기 2.8 / **Claude API 메커니즘 6.8** / SW 엔지니어링 기초 7.4 / **Claude 애플리케이션 설계 8.6** / 구성 관리 4.1 | [[02 Claude API 메커니즘]] · [[03 애플리케이션 설계와 통합]] |
| D5 | **Model Selection and Optimisation** | **16.8%** | LLM 기초 5.2 / 기술 기초 6.1 / 모델 선택·트레이드오프 2.7 / 비용·토큰 관리 2.8 | [[01 모델 선택과 최적화]] |
| D1 | **Agents and Workflows** | **14.7%** | 에이전트 아키텍처 4.5 / Claude로 에이전트 구축 5.3 / 에이전트 패턴·프레임워크 4.9 | [[04 에이전트와 워크플로]] |
| D6 | **Prompt and Context Engineering** | **11.0%** | 컨텍스트 엔지니어링 3.8 / 프롬프트 엔지니어링 4.6 / 출력 처리 2.6 | [[05 프롬프트와 컨텍스트 엔지니어링]] |
| D8 | **Tools and MCPs** | **10.6%** | 도구 구현 4.4 / MCP 서버 개발 2.1 / 에이전틱 커스터마이징 4.1 | [[06 도구와 MCP]] |
| D7 | **Security and Safety** | **8.1%** | AI 앱 보안 3.2 / 가드레일·안전 배포 2.3 / **Claude Hooks 1.0** / 식별·비밀·키 관리 1.6 | [[07 보안과 안전]] |
| D3 | **Claude Code** | **3.1%** | Claude Code 운영 3.1 | [[08 Claude Code 운영]] |
| D4 | **Eval, Testing, and Debugging** | **2.6%** | 디버깅·오류 처리 2.6 | [[09 평가 테스트 디버깅]] |

## 노트 목록

- [[00 시험 직전 최종 요약]] — **시험 전날 한 장 요약**: 문제 푸는 원칙, 도메인별 핵심, 숫자표, 자가 점검 14문항
- [[01 모델 선택과 최적화]] — LLM/토큰 기초, 모델 라인업·가격, 컨텍스트 윈도우, effort·thinking, 비용 최적화
- [[02 Claude API 메커니즘]] — Messages API, 콘텐츠 블록, stop_reason, 스트리밍 SSE, 오류·재시도, 속도 제한
- [[03 애플리케이션 설계와 통합]] — 비전·Files·PDF, 배치 처리, 구조화된 출력, SDK, 워크스페이스·키·환경 구성, 배포 플랫폼
- [[04 에이전트와 워크플로]] — 워크플로 vs 에이전트, 에이전트 루프, Agent SDK, Managed Agents, 서브에이전트
- [[05 프롬프트와 컨텍스트 엔지니어링]] — 프롬프팅 기법, 컨텍스트 편집·압축, 프롬프트 캐싱, 출력 처리
- [[06 도구와 MCP]] — 도구 정의·tool_choice·병렬 호출, 서버 도구, MCP 커넥터와 MCP 서버 개발
- [[07 보안과 안전]] — 프롬프트 인젝션·탈옥 방어, 가드레일, API 키·워크스페이스 격리, 훅
- [[08 Claude Code 운영]] — CLI, 설정, 권한, CLAUDE.md, 헤드리스 실행
- [[09 평가 테스트 디버깅]] — eval 설계, 채점 방식, 환각 감소, 디버깅 절차
- [[10 빈출 포인트 요약]] — 숫자 암기표, 헷갈리는 비교, 자가 점검 문항
- [[11 공식 커리큘럼 매핑]] — **Anthropic Partner Academy 공식 prep course 학습 목표 → 시험 도메인 매핑**
- [[12 RAG와 검색]] — 청킹·임베딩·BM25·하이브리드 파이프라인, Citations
- [[13 프로덕션 엔지니어링]] — 테스트 계층·추적, 재시도 vs 종료 오류, 예산 관리, 메모리 스코프, 플랫폼 선택, accelerator 패키징

## 학습 순서 제안

1. [[10 빈출 포인트 요약]]으로 전체 지형 훑기
2. **비중 1위** [[02 Claude API 메커니즘]] → [[03 애플리케이션 설계와 통합]] (합 33.1%)
3. [[01 모델 선택과 최적화]] (16.8%) → [[04 에이전트와 워크플로]] (14.7%)
4. [[05 프롬프트와 컨텍스트 엔지니어링]] → [[06 도구와 MCP]]
5. [[07 보안과 안전]] → [[08 Claude Code 운영]] → [[09 평가 테스트 디버깅]]
6. 실습: Messages API 왕복 → 스트리밍 → 도구 루프 → 배치 → 구조화된 출력 순으로 직접 코드 작성
7. 시험 직전: [[00 시험 직전 최종 요약]]만 훑고 자가 점검 문항 풀기

## 관련 노트

- [[CCAR-F 자격증]] — 아키텍트 트랙(CCA-F/CCAR-F). 에이전트 아키텍처·Claude Code 설정·MCP 부분이 겹치므로 교차 학습 권장

## 참고 링크

- Claude Platform 문서(한국어): <https://platform.claude.com/docs/ko/home>
- API 레퍼런스: <https://platform.claude.com/docs/ko/api/overview>
- Claude Code 문서: <https://code.claude.com/docs>
- Cookbook: <https://platform.claude.com/cookbook>
- Anthropic Academy: <https://academy.claude.com/courses>
- Pearson VUE(Anthropic): <https://www.pearsonvue.com/us/en/anthropic.html>
