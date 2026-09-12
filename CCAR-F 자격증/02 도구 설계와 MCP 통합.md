#CCAR-F
#Claude
#MCP
#Tool

---

# D2. 도구 설계 & MCP 통합 (18%)

## 1. 도구 사용(Tool Use)의 기본 구조

`tool use` = function calling. Claude가 요청과 **도구 설명**을 근거로 호출 시점을 판단하고 구조화된 호출을 반환한다.

### 클라이언트 도구 vs 서버 도구

| 구분 | 실행 위치 | 예시 | 앱이 할 일 |
| --- | --- | --- | --- |
| **클라이언트 도구** | 내 애플리케이션 | 사용자 정의 도구, `bash`, `text_editor`, `computer`, `browser`, `memory` | `tool_use` 파싱 → 실행 → `tool_result` 반환 |
| **서버 도구** | Anthropic 인프라 | `web_search`, `web_fetch`, `code_execution`, `advisor`, `tool_search`, MCP 커넥터 | 없음(결과가 바로 옴) |

### 왕복 흐름

1. 요청에 `tools` 배열 포함
2. 응답: `stop_reason: "tool_use"` + `tool_use` 블록(`id`, `name`, `input`)
3. 앱이 도구 실행
4. 다음 요청에 `assistant` 메시지(원본 content) + `user` 메시지의 `tool_result` 블록(`tool_use_id`, `content`) 추가
5. Claude가 최종 답변 생성

> SDK의 **Tool Runner**(`client.beta.messages.tool_runner`)를 쓰면 이 왕복을 직접 작성하지 않아도 된다.

## 2. 도구 정의 스키마

| 필드 | 설명 |
| --- | --- |
| `name` | `^[a-zA-Z0-9_-]{1,64}$` |
| `description` | 도구가 하는 일 / 쓸 때와 쓰지 말아야 할 때 / 각 파라미터 의미 / 반환하지 않는 것. **최소 3~4문장** |
| `input_schema` | JSON Schema 객체 |
| `input_examples` | (선택) 스키마 검증되는 예시 입력 배열 |

선택 속성: `cache_control`, `strict`, `defer_loading`, `allowed_callers`

```json
{
  "name": "get_stock_price",
  "description": "Retrieves the current stock price for a given ticker symbol. The ticker must be a valid symbol for a publicly traded company on a major US exchange like NYSE or NASDAQ. Returns the latest trade price in USD. Use it when the user asks about the current or most recent price of a specific stock. It will not provide any other information about the stock or company.",
  "input_schema": {
    "type": "object",
    "properties": {
      "ticker": { "type": "string", "description": "The stock ticker symbol, e.g. AAPL for Apple Inc." }
    },
    "required": ["ticker"]
  }
}
```

### 도구 설계 모범 사례 (시험 빈출)

1. **아주 상세한 설명** — 도구 성능에 가장 큰 영향을 주는 단일 요인
2. **설명 우선, 복잡한 입력에는 `input_examples`** — 중첩 객체·형식 민감 파라미터에 유용. 간단한 예시 20~50토큰, 복잡한 예시 100~200토큰
3. **관련 작업을 더 적은 도구로 통합** — `create_pr` / `review_pr` / `merge_pr` 대신 `action` 파라미터를 가진 단일 도구. 도구 수가 적고 유능할수록 선택 모호성이 준다
4. **의미 있는 네임스페이스** — `github_list_prs`, `slack_send_message`. tool search 사용 시 특히 중요
5. **응답은 신호가 높은 정보만** — 불투명한 내부 참조 대신 안정적 식별자(slug, UUID), 다음 단계 추론에 필요한 필드만. 비대한 응답은 컨텍스트 낭비

## 3. tool_choice

| 값 | 동작 |
| --- | --- |
| `auto` | Claude가 호출 여부 결정 (**`tools` 제공 시 기본값**) |
| `any` | 반드시 도구 중 하나를 사용 (특정 도구는 강제하지 않음) |
| `tool` | 특정 도구를 강제 (`{"type":"tool","name":"get_weather"}`) |
| `none` | 도구 사용 금지 (**`tools` 미제공 시 기본값**) |

주의사항:
- `any` / `tool`은 어시스턴트 메시지를 프리필하므로 **`tool_use` 앞에 자연어 설명이 나오지 않는다**
- `tool_choice` 변경은 **프롬프트 캐시의 메시지 블록을 무효화**한다(도구·시스템 캐시는 유지)
- 수동 extended thinking(`thinking: {type:"enabled"}`)과 `any`/`tool`은 **비호환(오류)**. `auto`/`none`만 가능. 적응형 사고 모델은 강제 도구 사용 지원
- `disable_parallel_tool_use: true` → 턴당 최대 1개 도구 호출

### 스키마 준수 보장
- 도구 정의에 `strict: true` → 도구 입력이 스키마를 엄격히 따름
- `tool_choice: any` + `strict: true` 조합 = "반드시 도구를 호출 + 입력이 스키마 준수" 둘 다 보장

## 4. 토큰 비용

도구 사용 시 추가 토큰이 발생하는 지점:
- `tools` 파라미터(이름·설명·스키마)
- `tool_use` 콘텐츠 블록
- `tool_result` 콘텐츠 블록
- **자동 삽입되는 도구 사용 시스템 프롬프트** (모델별 상이, 예: Opus 5 = `auto`/`none` 286토큰, `any`/`tool` 406토큰 / Sonnet 5 = 354 / 474)

서버 도구는 토큰 외 **사용량 기반 추가 과금**(예: 웹 검색 건당).

## 5. 도구가 많을 때: Tool Search

- `ToolSearch` 도구로 **필요할 때만 스키마를 로드**(deferred loading)
- Claude Code / Agent SDK에서 MCP 도구 스키마는 **기본적으로 deferred**. 이름만 컨텍스트에 올라가고 필요 시 검색해서 로드
- `ENABLE_TOOL_SEARCH=auto` → 컨텍스트 윈도우의 10% 내에 들어가면 선로딩, `false` → 전부 선로딩

---

# MCP (Model Context Protocol)

AI 도구를 외부 데이터 소스에 연결하는 **개방형 표준**. Claude Code가 Google Drive 문서 읽기, Jira 티켓 갱신, Slack 데이터 조회, 사내 커스텀 도구 사용 등을 할 수 있게 한다.

## 6. 전송 방식(Transport)

| 방식 | 설정 | 비고 |
| --- | --- | --- |
| **HTTP** | `claude mcp add --transport http <name> <url>` | **권장** |
| SSE | `--transport sse` | 지양(레거시) |
| stdio | `--transport stdio <name> -- <command> [args]` | 로컬 프로세스. `--`로 Claude 옵션과 서버 명령 분리 |
| WebSocket | `.mcp.json` 또는 `claude mcp add-json`에서만 (`"type": "ws"`) | |

```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/ --header "Authorization: Bearer YOUR_TOKEN"
```

```bash
claude mcp add --env AIRTABLE_API_KEY=KEY --transport stdio airtable -- npx -y airtable-mcp-server
```

### 주요 옵션

| 옵션 | 축약 | 설명 |
| --- | --- | --- |
| `--transport` | `-t` | `http` / `sse` / `stdio` / `ws` |
| `--header` | `-H` | HTTP 헤더(복수 가능) |
| `--env` | `-e` | 환경변수(복수 가능) |
| `--scope` | `-s` | `local`(기본) / `project` / `user` |
| `--client-id`, `--client-secret`, `--callback-port` | | OAuth 사전 등록 자격증명 |

## 7. 스코프(Scope) — 시험 빈출

| 스코프 | 저장 위치 | 적용 범위 | 팀 공유 |
| --- | --- | --- | --- |
| **local** (기본) | `~/.claude.json` | 현재 프로젝트만 | X |
| **project** | `.mcp.json` (프로젝트 루트) | 현재 프로젝트 | **O (버전 관리)** |
| **user** | `~/.claude.json` | 모든 프로젝트 | X |

**우선순위(높음 → 낮음)**: Managed MCP(조직) → Local → Project → User → 플러그인 제공 서버 → claude.ai 커넥터.
같은 이름이면 **우선순위가 높은 정의가 통째로 사용**되고 필드 병합은 하지 않는다.

### `.mcp.json` 형식

```json
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": { "Authorization": "Bearer ${API_KEY}" },
      "timeout": 600000
    },
    "db-server": {
      "command": "${CLAUDE_PROJECT_DIR}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": { "DB_URL": "${DB_URL}" }
    }
  }
}
```

### 환경변수 확장
- `${VAR}` — 환경변수 사용
- `${VAR:-default}` — 미설정 시 기본값
- 미설정 변수는 문자 그대로 남고 경고 표시
- 사용 가능 위치: `command`, `args`, `env` (stdio) / `url`, `headers`, `headersHelper` (HTTP·SSE·WS)
- 플레이스홀더: `${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_PLUGIN_DATA}`, `${CLAUDE_PROJECT_DIR}`

## 8. 인증

| 방식 | 설명 |
| --- | --- |
| **OAuth 2.0** (권장) | Dynamic Client Registration이면 자동. `/mcp`에서 브라우저 로그인, 또는 `claude mcp login <name>` / `logout` |
| 정적 헤더 | `--header "Authorization: Bearer ..."` — 단, 자격증명을 `.mcp.json`에 커밋하지 말 것 |
| **`headersHelper`** | 스크립트가 JSON 헤더를 stdout으로 출력. Kerberos·단기 토큰·사내 SSO용. `CLAUDE_CODE_MCP_SERVER_NAME`/`_URL` 환경변수 제공 |

OAuth 토큰은 시스템 키체인(macOS) 또는 자격증명 파일에 저장된다.

## 9. 리소스와 프롬프트

| 기능 | 문법 | 예 |
| --- | --- | --- |
| MCP **리소스** 참조 | `@[server:resource-type:path]` | `@[github:repository:owner/repo]` |
| MCP **프롬프트** 실행 | `/<server-name>:<prompt-name>` | `/github:pr-review` |
| 상태 확인 | `/mcp` | 연결 상태·인증·리소스/프롬프트 목록 |

### 서버 상태 표시
`✔ Connected` / `! Needs authentication` / `✘ Failed to connect` / `⏸ Pending approval`(프로젝트 서버, 신뢰 필요) / `⊘ Disabled for this project` / `cached 2h ago`

## 10. 출력 제한과 타임아웃

| 항목 | 값 |
| --- | --- |
| 출력 경고 임계값 | 10,000 토큰 |
| 기본 출력 제한 | 25,000 토큰 (초과 시 파일로 저장하고 경로 표시) |
| 전역 제한 변경 | `MAX_MCP_OUTPUT_TOKENS` |
| 도구별 제한(서버 개발자) | `_meta`의 `anthropic/maxResultSizeChars` |
| 서버 시작 타임아웃 | `MCP_TIMEOUT` (ms) |
| 도구 유휴 타임아웃 | `CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT` (기본 5분) |
| 자동 백그라운드 전환 | `CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS` |
| 서버별 타임아웃 | `.mcp.json`의 `timeout` (ms) |

## 11. 보안 (시험 포인트)

- **신뢰(Trust)**: 프로젝트 `.mcp.json` 서버 로드, 로컬 스코프 `headersHelper` 실행, 프로젝트 훅 실행은 **워크스페이스 신뢰 수락 후**에만 이뤄진다
- **프롬프트 인젝션**: 외부 콘텐츠를 가져오는 서버는 인젝션 위험에 노출된다. 알려진 서버만 추가하고 소스를 검토할 것
- **자격증명**: `~/.claude.json`은 `.gitignore`, `.mcp.json`에는 URL/헤더 형태만 두고 값은 환경변수나 `headersHelper`로
- **권한 규칙**: MCP 도구는 `mcp__<server>` / `mcp__<server>__*` / `mcp__<server>__<tool>` 로 allow/ask/deny 지정
  - `deny`에 `mcp__*` → 모든 MCP 도구 차단
  - allow 규칙에서 서버 세그먼트에는 와일드카드 불가(`mcp__puppeteer__*`는 OK, 서버명 자리에 `*`는 불가)
  - 설정 파일에서 괄호가 붙은 `mcp__` 규칙은 무시된다. MCP 도구의 파라미터 매칭은 `--disallowedTools`로만
- `requiresUserInteraction` 표시된 MCP 도구와 조직이 `ask`로 지정한 커넥터 도구는 **어떤 권한 모드에서도 자동 승인되지 않는다**

## 12. 서버 관리 명령

```bash
claude mcp list
claude mcp get notion
claude mcp remove notion
claude mcp login sentry
claude mcp logout sentry
claude mcp reset-project-choices
```

## 13. MCP 커넥터 (Messages API)

Claude Code가 아닌 **API에서 직접** 원격 MCP 서버에 연결하는 방법. 별도의 MCP 클라이언트 구현 없이 Messages API에서 원격 서버를 붙일 수 있다. 자체 클라이언트를 만들려면 MCP 공식 가이드의 "Build a client"를 참조.

## 연결 노트

- [[01 에이전트 아키텍처와 오케스트레이션]]
- [[03 Claude Code 설정과 권한]]
- [[06 컨텍스트 관리와 신뢰성]]
