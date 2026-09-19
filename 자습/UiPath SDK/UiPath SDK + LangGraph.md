
---
# 개발환경 구성
Python 3.11 ~ 3.13 지원(공식)
uv 사용 권장
사용자 환경: Windows

### 가상환경 설정

```powershell
uv venv --python 3.13

.venv\Scripts\activate

uv pip install uipath-langchain
```

### UiPath CLI 설치
공식 설치 스크립트:

```powershell
irm https://download.uipath.com/uipath-cli/install.ps1 | iex
```

Node.js 22+, UiPath CLI, coding-agent skills, .NET SDK 8, Python 등을 함께 준비

```powershell
uip --version
uip --help
```

설치 결과 확인

### Coded Agent Tool 설치

```powershell
uip tools install @uipath/codedagent-tool
```

Result가 Success이면 정상 처리 완료

```powershell
uip codedagent --help
```

설치 결과 확인

### UiPath 구성
UiPath 계정에 로그인

```powershell
uip login
```

브라우저가 열리면 Automation Cloud에 로그인하고 tenant를 선택

```powershell
uip login status
```

상태 확인(Result: Success)

**현재 설정을 그대로 유지**

```powershell
uip codedagent setup --force
```

### 스캐폴드 생성

```powershell
uip codedagent new OrderAgent
```

대충 다음 파일들을 만들어줌

```
OrderAgent/
│
├── main.py
├── pyproject.toml
├── langgraph.json
├── uipath.json
├── entry-points.json
├── bindings.json
│
├── AGENTS.md
├── CLAUDE.md
│
└── .agent/
```

개발 서버 추가

```powershell
uv add uipath-dev --dev
uv sync

uip codedagent init
```

### 디렉토리 구분 예시

```
OrderAgent/
│
├── main.py
│
├── pyproject.toml
│
├── langgraph.json
├── uipath.json
├── entry-points.json
├── bindings.json
│
├── agent/
│   ├── __init__.py
│   │
│   ├── graph.py
│   ├── state.py
│   │
│   └── nodes/
│       ├── __init__.py
│       ├── analyze.py
│       ├── execute.py
│       └── respond.py
│
├── tools/
│   ├── __init__.py
│   └── uipath_tools.py
│
├── prompts/
│   └── system.py
│
└── tests/
    └── test_agent.py
```

Clean Architecture로 설계

```
main.py
   │
   └── Agent Entry Point
             │
             ▼
       agent/graph.py
             │
       LangGraph 구성
             │
     ┌───────┼────────┐
     ▼       ▼        ▼
 analyze   execute   respond
     │       │
     │       ▼
     │    tools/
     │       │
     │       ▼
     │   UiPath SDK
     │
     ▼
   Claude
```

### Agent State 생성
`agent/state.py`
```python
from typing import TypedDict

class AgentState(TypedDict, total=False):
    request: str

    intent: str
    order_id: str

    tool_result: dict

    response: str
```

State는 Agent Workflow를 돌아다니는 데이터 저장 가방같은 느낌

### 프로젝트 로직 설계
요청 분석 등 핵심 로직
LangGraph 구성

### Tip
**UiPath Process를 Tool처럼 사용할 수 있음**

예를 들어, 주문 조회가 필요한 경우 Agent는
```
Claude
 │
 │ "주문 조회 필요"
 ▼
LangGraph
 │
 ▼
get_order_status()
 │
 ▼
UiPath Process
 │
 ▼
ERP
```
이런 식으로 작업하게 됨

`tools/uipath_tools.py`
```python
async def get_order_status(order_id: str) -> dict:

    # UiPath SDK
    # ↓
    # GetOrderStatus Process 실행
    # ↓
    # 결과 대기

    return {
        "order_id": order_id,
        "status": "shipping"
    }
```

`agent/nodes/execute.py`
```python
from tools.uipath_tools import get_order_status

async def execute_request(state: AgentState):

    if state["intent"] == "order_status":

        result = await get_order_status(
            state["order_id"]
        )

        return {
            "tool_result": result
        }

    return {}
```
이런 식으로 노드에서 활용

**Adapter를 활용할 것**
나중에는 Tool이 엄청 많아질 수 있음
Agent는 실행 함수만 알면 되고, 그 안에서의 실제 구현(UiPath Process, REST API, SAP API 등)은 Agent가 몰라도 됨

### main.py
최대한 얇게 구성
`main.py`는 Agent의 Entry Point 역할
```python
from agent.graph import graph

agent = graph
```
이 정도만 구성함

실제 UiPath Coded Agent의 entry point와 schema는 scaffold 버전과 SDK 버전에 맞춰 생성 권장
`uip codedagent new`가 framework에 맞는 scaffold를 생성하고
`uip codedagent init`이 entry-point schema를 생성하는 것이 현재 공식 흐름임

### 로컬 테스트
로컬 개발 서버 지원
```powershell
uip codedagent dev
```
localhost:8080으로 사용중
Automation(가장 왼쪽)으로 선택하고 테스트 진행 결과 확인 가능

### 패키징
```python
uip codedagent pack
```
버전 업데이트하는 작업
Coded Agent는 UiPath에서 배포 가능한 artifact로 패키징됨 
UiPath에서는 Coded Agent를 일반 UiPath automation과 마찬가지로 `.nupkg` 패키지 형태로 Orchestrator에 배포

### Orchestrator 배포
개인 워크스페이스에 배포한다면
```powershell
uip codedagent publish --my-workspace
```
이후 브라우저 창에서 환경변수 등 설정

