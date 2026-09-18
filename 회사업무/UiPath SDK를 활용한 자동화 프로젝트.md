
---
# 프로젝트 구조
```
사용자 요청 / 업무 이벤트
          ↓
┌─────────────────────────┐
│    Coded Agent          │
│    Python               │
│                         │
│  LLM → 판단 → Tool 선택 │
│       ↘       ↙         │
│       상태/로직 관리     │
└────────────┬────────────┘
             │
       UiPath SDK
             │
 ┌───────────┼────────────┐
 ↓           ↓            ↓
RPA Process  Queue       Asset
 ↓                        ↓
ERP/웹      업무처리     API Key

             +
             ↓
       Action Center
       사람 승인/검토
```

# 지원되는 개발 방식
| 방식                | 용도                         |
| ----------------- | -------------------------- |
| **Python SDK**    | 직접 Python으로 에이전트 로직 구현     |
| **LangGraph**     | 상태 기반/복잡한 Agent Workflow   |
| **LlamaIndex**    | RAG/문서 검색 중심 Agent         |
| **OpenAI Agents** | OpenAI Agents SDK 기반 Agent |
| **MCP SDK**       | MCP Server 구축              |

# 특징

RPA 프로젝트를 일반적인 Python 프로젝트처럼 구성 가능
```
customer-service-agent/
│
├── main.py
├── agent/
│   ├── graph.py
│   ├── state.py
│   └── nodes/
│       ├── classify.py
│       ├── reason.py
│       └── execute.py
│
├── tools/
│   ├── erp.py
│   ├── email.py
│   └── uipath.py
│
└── pyproject.toml
```

# 예시
```
                    [업무 요청]
                        ↓
                 Coded Agent
                        ↓
            "무슨 업무인지 판단"
              ↙        ↓       ↘
          송장업무    문의업무    ERP업무
             ↓          ↓         ↓
       UiPath RPA    LLM 답변   UiPath RPA
             ↓                    ↓
       InvoiceRegister       CreateERPRecord
```

RPA들이 Agent의 Tool로써 작동함

**Agent**
```
요청 이해
↓
판단
↓
어떤 Tool을 사용할지 결정
↓
결과 해석
↓
다음 행동 결정
```

**RPA**
```
ERP 로그인
버튼 클릭
데이터 입력
파일 다운로드
Excel 조작
메일 전송
```

# HITL 설정
예를 들어 Agent가 판단했는데
```
환불 금액: 3,200,000원
```

이라면 바로 실행하지 않고
```
Agent
 ↓
환불 필요 판단
 ↓
금액 > 1,000,000
 ↓
Action Center
 ↓
담당자 승인
 ↓
승인
 ↓
Refund RPA 실행
```

HITL 구조 설계 가능

UiPath Coded Agent에서는 **interrupt point를 정의하여 실행을 중지하고 사람에게 입력/승인을 요청하는 Human-in-the-loop 구조**를 지원

# Context Grounding
```
사용자:
"출장 숙박비 18만원인데 처리 가능해?"
        ↓
Coded Agent
        ↓
Context Grounding
        ↓
출장규정 검색
        ↓
"국내 출장 숙박비 한도 150,000원"
        ↓
Agent 판단
        ↓
규정 초과 → 승인 요청
```

UiPath가 공식 예제로도 **Context Grounding index를 조회하여 회사 정책을 확인하는 Coded Agent** 패턴을 제시하고 있음

# Orchestrator
Agent 운영 플랫폼 역할 수행

UiPath에서는 Agent를 패키징해서 **Orchestrator에 `.nupkg` 형태로 배포 가능**
```
Python Agent
     ↓
uip codedagent pack
     ↓
.nupkg
     ↓
Orchestrator
     ↓
Process
```

일반 UiPath Process처럼 **schedule / trigger / monitor / governance**를 적용할 수 있음

# Claude Code / Codex 연동
개발 경험의 변화:
```
Claude Code / Codex

"ERP 주문 취소를 처리하는
LangGraph 기반 UiPath Coded Agent 만들어줘.

주문 취소 RPA Process를 Tool로 사용하고
100만원 이상이면 Action Center 승인을 받아."
```

↓
```
Python 프로젝트 생성
↓
Agent 코드 작성
↓
UiPath 연결
↓
테스트
↓
패키징
↓
Orchestrator Publish
```
형태가 됨


공식 문서: [UiPath Coded Agents 개요](https://docs.uipath.com/agents/automation-suite/2.2510/user-guide/about-coded-agents?utm_source=chatgpt.com) / [UiPath Agents SDK](https://docs.uipath.com/sdk/other/latest/developer-guide/using-agents-sdks?utm_source=chatgpt.com) / [Coding Agents로 Coded Agent 구축하기](https://docs.uipath.com/agents/automation-cloud/latest/user-guide/building-coded-agents-with-coding-agents?utm_source=chatgpt.com)