#CCDV-F
#Claude
#Tool
#MCP

---

# D8. 도구와 MCP (10.6%)

> 세부: 도구 구현 4.4% / MCP 서버 개발 2.1% / 에이전틱 커스터마이징 4.1%

## 1. 클라이언트 도구 vs 서버 도구

| 구분 | 실행 위치 | 예시 | 앱이 할 일 |
| --- | --- | --- | --- |
| **클라이언트 도구** | 내 애플리케이션 | 사용자 정의 도구, `bash`, `text_editor`, `memory`, `computer`, `browser` | `tool_use` 파싱 → 실행 → `tool_result` 반환 |
| **서버 도구** | Anthropic 인프라 | `web_search`, `web_fetch`, `code_execution`, `advisor`, `tool_search`, MCP 커넥터 | 없음(결과가 바로 옴) |

### 왕복 흐름
1. 요청에 `tools` 배열 포함
2. 응답: `stop_reason: "tool_use"` + `tool_use` 블록(`id`, `name`, `input`)
3. 앱이 도구 실행
4. 다음 요청에 assistant 메시지(원본 content) + user 메시지의 `tool_result` 블록(`tool_use_id`, `content`) 추가
5. Claude가 최종 답변 생성

> **Tool Runner**(`client.beta.messages.tool_runner`)를 쓰면 이 루프를 SDK가 대신 돌린다.

### 주요 서버 도구
| 도구 | 비용 |
| --- | --- |
| `web_search` | 검색 **1,000회당 $10** + 토큰 |
| `web_fetch` | **추가 요금 없음** (토큰만). `max_content_tokens`로 상한 설정 |
| `code_execution` | 컨테이너-시간당 $0.05, 월 1,550시간 무료. **웹 검색/가져오기와 함께 쓰면 무료** |
| `advisor` | 빠른 executor 모델이 생성 중 고지능 advisor에게 자문 |
| `tool_search` | 수천 개 도구를 필요할 때만 로드 |

## 2. 도구 정의

| 필드 | 설명 |
| --- | --- |
| `name` | `^[a-zA-Z0-9_-]{1,64}$` |
| `description` | 무엇을 하는지 / 언제 쓰고 언제 안 쓰는지 / 각 파라미터 의미 / 반환하지 않는 것. **최소 3~4문장** |
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
2. **복잡한 입력에는 `input_examples`** — 중첩 객체·형식 민감 파라미터에 유용. 간단한 예시 20~50토큰, 복잡한 예시 100~200토큰
3. **관련 작업을 더 적은 도구로 통합** — `create_pr`/`review_pr`/`merge_pr` 대신 `action` 파라미터를 가진 단일 도구
4. **의미 있는 네임스페이스** — `github_list_prs`, `slack_send_message`. tool search 사용 시 특히 중요
5. **응답은 신호가 높은 정보만** — 안정적 식별자(slug, UUID)와 다음 단계 추론에 필요한 필드만. 비대한 응답은 컨텍스트 낭비

### 파라미터가 누락되면
Opus 계열은 누락을 인지하고 되묻는 경향이 강하고, **Sonnet/Haiku는 합리적인 값을 추론해 채울 수 있다**. 보장되는 동작이 아니다.

## 3. tool_choice

| 값 | 동작 |
| --- | --- |
| `auto` | Claude가 호출 여부 결정 (**`tools` 제공 시 기본값**) |
| `any` | 반드시 도구 중 하나 사용 |
| `tool` | 특정 도구 강제 (`{"type":"tool","name":"..."}`) |
| `none` | 도구 사용 금지 (**`tools` 미제공 시 기본값**) |

- `any`/`tool`은 어시스턴트 메시지를 프리필하므로 **`tool_use` 앞에 자연어 설명이 나오지 않는다**
- `tool_choice` 변경은 **메시지 캐시를 무효화**한다(도구·시스템 캐시는 유지)
- 수동 extended thinking과 `any`/`tool`은 **비호환(오류)**
- **Opus 5.5, Fable 5.1, Mythos 5.1은 강제 도구 사용을 지원하지 않는다** → `auto` + `strict: true` 또는 구조화된 출력 사용
- `disable_parallel_tool_use: true` → 턴당 최대 1개 도구 호출

## 4. 병렬 도구 호출

최신 모델은 독립적인 도구 호출을 **병렬로 실행**한다(여러 파일 동시 읽기, 투기적 검색, bash 병렬 실행).
- 성공률을 ~100%로 올리려면 `<use_parallel_tool_calls>` 프롬프트 사용 → [[05 프롬프트와 컨텍스트 엔지니어링]]
- 줄이려면 "순차 실행" 지시
- 긴 에이전트 루프(Fable 5.1)에서는 **각 도구 결과 라운드 후 턴 범위 시스템 메시지**로 지시를 보낸다

## 5. 토큰 비용

추가 토큰 발생 지점: `tools` 파라미터, `tool_use` 블록, `tool_result` 블록, **자동 주입되는 도구 사용 시스템 프롬프트**.

| 모델 | `auto`/`none` | `any`/`tool` |
| --- | --- | --- |
| Opus 5.5 | 286 | (강제 미지원) |
| Opus 5 | 286 | 406 |
| Sonnet 5 | 354 | 474 |
| Haiku 4.5 | 496 | 588 |

도구별 추가 토큰: bash 도구 244~325, text editor 700, computer toolset 약 4,500, browser toolset 약 6,600.

## 6. 도구가 많을 때: Tool Search

`ToolSearch` 도구로 **필요할 때만 스키마를 로드**한다(deferred loading). Claude Code / Agent SDK에서 MCP 도구 스키마는 **기본이 deferred** — 이름만 컨텍스트에 올라간다.
`ENABLE_TOOL_SEARCH=auto` → 컨텍스트의 10% 내에 들어가면 선로딩, `false` → 전부 선로딩.

---

# MCP (Model Context Protocol)

AI 도구를 외부 데이터 소스에 연결하는 **개방형 표준**.

## 7. MCP 커넥터 (Messages API)

별도의 MCP 클라이언트 구현 없이 **Messages API에서 직접 원격 MCP 서버에 연결**한다. 배치 요청에서도 동작한다. 단, **토큰 카운팅 엔드포인트는 MCP 커넥터를 지원하지 않는다**(400).

## 8. Claude Code에서의 MCP

### 전송 방식
| 방식 | 설정 | 비고 |
| --- | --- | --- |
| **HTTP** | `claude mcp add --transport http <name> <url>` | **권장** |
| SSE | `--transport sse` | 지양(레거시) |
| stdio | `--transport stdio <name> -- <command> [args]` | 로컬 프로세스. `--`로 분리 |
| WebSocket | `.mcp.json` / `add-json`에서만 (`"type": "ws"`) | |

### 스코프와 우선순위
| 스코프 | 저장 위치 | 범위 | 팀 공유 |
| --- | --- | --- | --- |
| **local** (기본) | `~/.claude.json` | 현재 프로젝트 | X |
| **project** | `.mcp.json` (프로젝트 루트) | 현재 프로젝트 | **O (커밋)** |
| **user** | `~/.claude.json` | 모든 프로젝트 | X |

**우선순위**: Managed MCP → Local → Project → User → 플러그인 → claude.ai 커넥터. 같은 이름이면 **필드 병합 없이 통째로 교체**.

### 환경변수 확장
`${VAR}`, `${VAR:-default}`, `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_PLUGIN_DATA}`
사용 위치: `command`/`args`/`env`(stdio), `url`/`headers`/`headersHelper`(HTTP·SSE·WS)

### 인증
| 방식 | 설명 |
| --- | --- |
| **OAuth 2.0** (권장) | DCR이면 자동. `/mcp`에서 브라우저 로그인 또는 `claude mcp login <name>` |
| 정적 헤더 | `--header "Authorization: Bearer ..."` — 자격증명을 `.mcp.json`에 커밋 금지 |
| **`headersHelper`** | 스크립트가 JSON 헤더를 stdout으로 출력. Kerberos·단기 토큰·사내 SSO용 |

### 출력 제한
| 항목 | 값 |
| --- | --- |
| 경고 임계값 | 10,000 토큰 |
| 기본 제한 | **25,000 토큰** (초과 시 파일로 저장) |
| 전역 변경 | `MAX_MCP_OUTPUT_TOKENS` |
| 도구별 제한(서버 개발자) | `_meta`의 `anthropic/maxResultSizeChars` |

### 리소스·프롬프트
| 기능 | 문법 |
| --- | --- |
| 리소스 참조 | `@[server:resource-type:path]` |
| 프롬프트 실행 | `/<server-name>:<prompt-name>` |
| 상태 확인 | `/mcp` |

## 9. MCP 서버 개발 (2.1%)

MCP 서버가 클라이언트에 노출하는 세 가지 기본 요소:

| 프리미티브 | 설명 | Claude Code에서의 사용 |
| --- | --- | --- |
| **Tools** | 모델이 호출하는 함수 | `mcp__<server>__<tool>` |
| **Resources** | 읽을 수 있는 데이터(파일·URI) | `@[server:type:path]` 멘션 |
| **Prompts** | 프리셋 프롬프트 템플릿 | `/<server>:<prompt>` 슬래시 명령 |

서버 개발 시 고려할 점:
- 도구 정의는 **클라이언트 도구와 같은 설계 원칙**을 따른다(상세한 설명, 통합, 네임스페이스, 신호 높은 응답)
- 읽기 전용 도구에는 `readOnlyHint` 어노테이션을 달아 **병렬 실행**을 허용한다
- 큰 응답은 `_meta`로 결과 크기 상한을 선언한다
- 사용자 상호작용이 꼭 필요한 도구는 `requiresUserInteraction`으로 표시 → 어떤 권한 모드에서도 자동 승인되지 않는다
- 전송 방식은 HTTP를 우선하고, 로컬 프로세스만 stdio를 쓴다

## 10. 에이전틱 커스터마이징 (4.1%)

Claude Code / Agent SDK에서 도구 표면을 조정하는 수단:

| 수단 | 용도 |
| --- | --- |
| `allowed_tools` / `disallowed_tools` | 도구 허용·차단 |
| 권한 규칙 (`mcp__<server>__<tool>`) | allow/ask/deny. **평가 순서는 deny → ask → allow** |
| 서브에이전트의 `tools` / `disallowedTools` | 위임 시 도구 제한 |
| `PreToolUse` 훅 | 런타임 검증·차단(exit 2) |
| 스킬의 `allowed-tools` | 그 턴에만 사전 승인 |
| MCP 서버 스코프 | 팀 공유 여부 결정 |

- `deny`에 `mcp__*` → 모든 MCP 도구 차단
- allow 규칙의 **서버 세그먼트에는 와일드카드 불가**(`mcp__puppeteer__*`는 OK)
- 설정 파일에서 **괄호가 붙은 `mcp__` 규칙은 무시**된다 → 파라미터 매칭은 `--disallowedTools`로만

## 연결 노트

- [[04 에이전트와 워크플로]]
- [[07 보안과 안전]] — MCP 신뢰·프롬프트 인젝션
- [[02 도구 설계와 MCP 통합]] (CCAR-F 노트, 더 상세)
