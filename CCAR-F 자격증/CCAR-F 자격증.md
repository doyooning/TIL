#MOC
#CCAR-F
#Claude

---

# CCAR-F 자격증 MOC

Anthropic **Claude Certified Architect - Foundations (CCA-F / CCAR-F)** 시험 대비 노트 허브입니다.
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

## 참고 링크

- Claude Platform 문서(한국어): <https://platform.claude.com/docs/ko/home>
- Claude Code 문서: <https://code.claude.com/docs>
- 문서 전체 색인(LLM용): <https://code.claude.com/docs/llms.txt>
- Anthropic Academy: <https://academy.claude.com/courses>
