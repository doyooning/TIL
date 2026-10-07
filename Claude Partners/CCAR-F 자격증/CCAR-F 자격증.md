#MOC
#CCAR-F
#Claude

---

# CCAR-F 자격증 MOC

Anthropic **Claude Certified Architect – Foundations (CCAR-F)** 시험 대비 노트 허브입니다.

> ⚠️ **약칭 주의**: Anthropic 자격증은 4종입니다 — **CCA-F**(Claude Certified **Associate**, Foundations, $99) · **CCDV-F**(Developer, $125) · **CCAR-F**(Architect Foundations, $125) · **CCAR-P**(Architect Professional, $175).
> 2차 자료에서 CCA-F와 CCAR-F를 같은 것으로 쓰는 경우가 많지만, Partner Academy 카탈로그상 **별개 시험**입니다. CCA-F는 컨설턴트·세일즈·딜리버리 리드 대상이며 CPN 티어 자격 요건에 포함되지 않습니다.
공식 문서(<https://platform.claude.com/docs/ko/home>, <https://code.claude.com/docs>)를 기준으로 도메인별로 정리했습니다.

> 새 노트를 추가할 때마다 이 MOC에도 링크를 추가해주세요.

## 시험 개요

| 항목 | 내용 |
| --- | --- |
| 정식 명칭 | Claude Certified Architect, Foundations |
| 대상 | 프로덕션 수준 Claude 기반 애플리케이션을 설계·구축하는 솔루션 아키텍트 |
| 형식 | 시나리오 기반 객관식 / 복수 응답 (60문항) |
| 시간 | 120분 |
| 합격선 | 1000점 만점 환산 점수 **720점** (범위 100~1000) |
| 플랫폼 | Anthropic 공식 교육 플랫폼(Skilljar) |
| 응시 자격 | Claude Partner Network 파트너사 소속자 (개인 응시 불가) |

## 출제 도메인과 비중

| 도메인 | 주제 | 비중 | 노트 |
| --- | --- | --- | --- |
| D1 | 에이전틱 아키텍처 & 오케스트레이션 | **27%** | [[01 에이전트 아키텍처와 오케스트레이션]] |
| D2 | 도구 설계 & MCP 통합 | **18%** | [[02 도구 설계와 MCP 통합]] |
| D3 | Claude Code 설정 & 워크플로우 | **20%** | [[03 Claude Code 설정과 권한]] · [[04 Claude Code 확장 - 메모리 스킬 훅 서브에이전트]] |
| D4 | 프롬프트 엔지니어링 & 구조화된 출력 | **20%** | [[05 프롬프트 엔지니어링과 구조화된 출력]] |
| D5 | 컨텍스트 관리 & 신뢰성 | **15%** | [[06 컨텍스트 관리와 신뢰성]] |

## 노트 목록

- [[01 에이전트 아키텍처와 오케스트레이션]] — 에이전트 루프, 워크플로 vs 에이전트, 서브에이전트 오케스트레이션, Agent SDK
- [[02 도구 설계와 MCP 통합]] — 도구 정의 스키마, tool_choice, MCP 전송 방식/스코프/인증
- [[03 Claude Code 설정과 권한]] — settings.json 계층, 권한 규칙/모드, 샌드박스
- [[04 Claude Code 확장 - 메모리 스킬 훅 서브에이전트]] — CLAUDE.md, 스킬, 훅, 서브에이전트, 플러그인
- [[05 프롬프트 엔지니어링과 구조화된 출력]] — 프롬프팅 기법, structured outputs, strict tool use
- [[06 컨텍스트 관리와 신뢰성]] — 컨텍스트 윈도우, 압축, 컨텍스트 편집, 프롬프트 캐싱
- [[07 CLI 슬래시 명령 치트시트]] — claude CLI 플래그, 슬래시 명령, claude mcp 하위 명령
- [[08 빈출 포인트 요약]] — 도메인별 핵심 암기 포인트와 헷갈리는 비교표

## 학습 순서 제안

1. [[08 빈출 포인트 요약]]으로 전체 지형 파악
2. 비중이 큰 [[01 에이전트 아키텍처와 오케스트레이션]] → [[03 Claude Code 설정과 권한]] / [[05 프롬프트 엔지니어링과 구조화된 출력]]
3. [[02 도구 설계와 MCP 통합]] → [[06 컨텍스트 관리와 신뢰성]]
4. 실습: 로컬에서 `.claude/settings.json`, `.mcp.json`, 커스텀 스킬/훅/서브에이전트를 직접 만들어 보기
5. [[07 CLI 슬래시 명령 치트시트]]로 마무리 암기

## 공식 Prep Course (Anthropic Partner Academy)

`/page/claude-certified-architect-foundations-prep-courses`에 나열된 코스 = 사실상의 출제 범위 신호:

| 코스 | 내용 |
| --- | --- |
| **AI Fluency: Framework & Foundations** | AI 시스템과 효과적·효율적·윤리적·안전하게 협업하기 |
| **Building with the Claude API** | API 접근·프롬프트 eval·프롬프트 엔지니어링·도구 사용·**RAG와 에이전틱 검색**·Claude 기능(사고/비전/PDF/**Citations**/캐싱/코드 실행)·MCP·에이전트와 워크플로 |
| **Claude on Google Cloud** | Vertex AI에서의 Claude |
| **Claude with Amazon Bedrock** | Bedrock에서의 Claude |
| **Claude Code in Action** | 장시간 무감독 세션 운영: steer(plan mode·압축 유도·rewind) / configure(CLAUDE.md·skills·권한 모드·훅) / automate(routines·headless·GitHub Actions) / verify & share(검증·플러그인) |
| **Introduction to Model Context Protocol** | Python SDK로 MCP 서버·클라이언트 구축, server inspector, tools/resources/prompts |
| **Claude 101** | 일상 업무에서의 Claude |

> **Bedrock과 Google Cloud 코스가 prep에 포함**되어 있다 = 클라우드 플랫폼 배포와 비교가 출제 범위다. 현재 노트 01~08에는 이 부분이 얇으므로 보강이 필요하다.
> 플랫폼 비교표는 CCDV-F 노트 [[13 프로덕션 엔지니어링]]에 정리해두었다.

## 관련 노트

- [[CCDV-F 자격증]] — 개발자 트랙(CCDV-F). 도구·MCP, 에이전트, 프롬프트 엔지니어링, Claude Code 부분이 겹치므로 교차 학습 권장
- [[11 공식 커리큘럼 매핑]] — Partner Academy 공식 커리큘럼 전체 매핑(양쪽 시험 공통)
- [[12 RAG와 검색]] — CCAR-F prep의 "Building with the Claude API"에도 포함된 영역
- [[13 프로덕션 엔지니어링]] — 플랫폼 선택·데이터 경계

## 참고 링크

- Claude Platform 문서(한국어): <https://platform.claude.com/docs/ko/home>
- Claude Code 문서: <https://code.claude.com/docs>
- 문서 전체 색인(LLM용): <https://code.claude.com/docs/llms.txt>
- Anthropic Academy: <https://academy.claude.com/courses>
