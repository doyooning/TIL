#AI 
#자동화
#n8n

---
# Claude Code x n8n 자동화 구조
![[Drawing 2026-06-02 10.50.44.excalidraw.png]]
워크플로우 JSON 핵심 구조
nodes: 노드 배열(이름, 타입, 설정)
connections: 노드 간 연결 라우팅
settings: 워크플로우 메타데이터

# Docker로 n8n 설치하기
Claude Code에 입력:
```
"Docker로 n8n 서버를 로컬에 설치해줘"
```

Claude Code가 생성하는 명령어:
```bash
docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n n8nio/n8n
```

`-d` → 백그라운드 실행 (터미널 닫아도 서버 유지) 
`-p` 5678:5678 → 내 컴퓨터 5678 포트 ↔ n8n 5678 포트 연결 
`-v` n8n_data:... → 데이터 영속 저장 (컨테이너 삭제해도 유지)

준비물: docker-compose
# n8n 웹 UI 접속 + 초기 설정
`localhost:5678`에서 바로 확인
로그인창 로그인
API Key 발급

# n8n MCP 서버 연결

### MCP란?
Model Context Protocol

==MCP 연결 전vs후==
MCP 연결 전: 
파일 읽기/쓰기 + 터미널 실행만 가능
n8n 서버에는 접근 불가

MCP 연결 후: 
n8n 워크플로우 조회·생성·수정·삭제 가능 
21개 n8n 제어 도구 자동 활성화!

### MCP 설정 — settings.json
.claude/settings.json에 추가

**방법 1 — Claude Code에게 맡기기 (추천)**
> "n8n MCP 서버를 연결해줘. 내 n8n 서버 주소는 http://localhost:5678이고, API 키는 .env 파일에 있어." 

**방법 2 — 직접 설정 (.claude/settings.json)**
```json
{
	"mcpServers": { 
		"n8n": {
			"type": "url",
			"url": "https://n8n-mcp.example.com/mcp",
			"headers": {
				"x-n8n-api-key": "YOUR_API_KEY",
				"x-n8n-host-url": "http://localhost:5678"
			}
		} 
	}
}
x-n8n-api-key → 이전 클립에서 발급한 API 키로 교체 
x-n8n-host-url → 내 n8n 서버 주소 (로컬 또는 Cloud)
```

### 연결 확인
health_check로 MCP 연결 상태 검증

Claude Code에 입력: 
`"n8n 서버 상태 확인해줘"`

###### ✅정상 연결 시 응답
`n8n_health_check → "n8n is running"` 
(MCP  연결 성공!)

###### ❌에러 발생 시 확인 사항
- API 키 앞뒤 공백 확인 (복사 시 실수 주의)
- Docker Desktop에서 n8n 컨테이너 "Running" 상태 확인
- 설정 저장 후 Claude Code 재시작 필요

### 주의사항
#### settings.json 관련
(글로벌) ~/.claude/settings.json
이 컴퓨터의 모든 프로젝트에 공통으로 적용되는 사용자 전역 설정

(로컬) .claude/settings.local.json
같은 프로젝트여도 "내 컴퓨터에서만" 적용되는 로컬 전용 설정

#### 변경사항 적용이 안돼요
자동 업데이트가 안되면 재시작해도 변경 안 될 가능성 높음
/doctor로 Auto-update: enable인지 확인
그 다음 수동 업데이트 진행(관리자 권한 터미널 권장)
```bash
npm install -g @anthropic-ai/claude-code@latest
```

근데 터미널에서 Claude 세션 다 종료해도 Claude Desktop 사용중이면
Claude 서비스가 실행중이라 수동 업데이트가 안 됨(EPERM)
-> 모든 서비스 종료 후 다시 시도, 버전 확인(`claude --version`) 및 /doctor 확인

그래도 계속 install failed가 기록되어 있다면 다음 명령어를 실행
```bash
Remove-Item "$env:USERPROFILE\.claude\.last-update-result.json"
```

그 다음 터미널에서 claude 재시작

#### /mcp 에서 아무것도 안떠요
mcp서버 연결 방법이 이전과 조금 달라진 듯
해당 프로젝트에서 claude가 mcp 읽을 때 .mcp.json에서 읽음
파일을 만들도록 지시하면 됨

결과물
```bash
{
  "mcpServers": {
    "n8n": {
      "command": "npx",
      "args": ["-y", "n8n-mcp"],
      "env": {
        "N8N_API_URL": "http://localhost:5678/api/v1",
        "N8N_API_KEY": "내 API 키"
      }
    }
  }
}
```

#### 워크플로우 만드는데 SSRF 문제가 생겨요
mcp 활용해서 만들면 localhost로 연결해서 SSRF 문제 생기는 듯

1. SSRF를 우회한다
2. 안되면 n8n REST API를 직접 사용해서 만든다

==Claude 가라사대...==
MCP가 하는 일은 결국 이 API를 내부적으로 대신 호출해주는 래퍼입니다. 
이번엔 SSRF 문제로 MCP가 그 API 호출 자체를 막아서, 우리가 직접 호출한 거고요.

MCP가 제대로 동작했다면:
Claude Code → mcp__n8n__n8n_create_workflow(…) → n8n-mcp 서버 → REST API → n8n

이번에 실제로 한 것:
Claude Code → Node.js HTTP 요청 → n8n REST API → n8n

다음 세션에서는 패치한 ssrf-protection.js가 로드되므로 MCP 도구가 제대로 동작할 것입니다. 
그때 mcp__n8n__n8n_create_workflow로 만들어보면 MCP 경험을 제대로 확인할 수 있습니다.

