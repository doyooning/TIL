# Extensibility 요약 (Claude Partner Badge: Claude Code)

총 8개 레슨: ① MCP 서버 ② MCP 허용 목록 거버넌스 ③ MCP 터널(원격 릴레이) ④ Skills 개념 ⑤ Skills 작성법 ⑥ Hooks ⑦ 서브 에이전트 ⑧ 플러그인 마켓플레이스와 버전 관리

이 코스의 산출물: **Activation plugin** (Skill, hook, MCP 설정, 서브 에이전트, 슬래시 명령을 하나로 묶어 Day 30에 고객 CoE에 전달)

---

## Lesson 1. MCP 서버: 티켓, 오류 로그, 내부 시스템

> **MCP는 Claude Code를 고객의 시스템에 연결한다.** 모델은 그대로이고, 닿을 수 있는 범위가 달라진다.

### MCP가 추가하는 것

MCP(Model Context Protocol)는 Claude를 외부 시스템에 연결하는 개방형 표준이다.

| 요소 | 설명 |
|---|---|
| **Tools** | 외부 시스템에서 수행하는 행동(티켓 생성, DB 조회, 파이프라인 실행). 어떤 도구를 부를지는 모델이 결정 |
| **Resources** | 외부 시스템에서 읽는 데이터(열린 티켓, 최근 오류 로그, API 스키마). 필요할 때만 컨텍스트에 로드 |
| **Prompts** | 서버가 제공하는 재사용 템플릿(예: 코딩 전에 API 문서를 로드하는 표준 서두) |
| **Scoping** | 프로젝트(저장소 하나), 사용자(개발자 한 명), 조직(전체 개발자) 레벨 |

### 설치

```bash
claude mcp add linear                                                     # 티켓 맥락
claude mcp add --transport stdio playwright -- npx @playwright/mcp@latest  # 브라우저 자동화·E2E 테스트
/mcp                                                                      # 연결된 서버와 노출된 도구 확인
```

| 설정 위치 | 범위 |
|---|---|
| `.mcp.json` (프로젝트 루트, `--scope project`) | 해당 프로젝트만 |
| `~/.claude.json` + `--scope user` | 그 개발자의 모든 프로젝트 |
| `~/.claude.json` (기본 local 스코프) | 현재 프로젝트에 키가 걸려 다른 곳에는 안 나타남 |
| `managed-settings.json` | 조직 전체 (Lesson 2) |

### 거의 모든 활성화에서 등장하는 3가지 유형

1. **티켓 시스템** (Jira, Linear, GitHub Issues) – 수정 전에 티켓 맥락을 가져온다. "Ticket to PR" 패턴의 출발점
2. **오류 로그·관측성** (OTel, Datadog, Sentry) – 디버깅할 때 오류 트레이스를 직접 읽는다. "운영 오류 발생 → 수정 착수" 간격을 줄인다
3. **내부 시스템** (사내 API, DB, 서비스 카탈로그, Confluence) – 커스텀 MCP 서버나 MCP 터널(Lesson 3)이 필요하다

### 고객에게 설명하는 법

- MCP는 데이터 통합 프로젝트가 아니라 **컨텍스트 증폭기**다. 티켓을 열고, 읽고, 에디터로 옮기는 단계를 없애 개발자가 이미 맥락을 가진 Claude와 시작하게 한다.
- **보안 관점:** OAuth 범위 서버는 사용자의 권한을 그대로 물려받는다(사용자가 가진 것 이상은 안 보임). 하지만 정적 API 키, 서비스 계정, `headersHelper` 자격증명을 쓰는 서버는 사용자 권한과 다른 접근을 줄 수 있다. **배포 전에 인증 방식과 노출 도구를 반드시 확인할 것.**

### 시나리오

"노트북에서 외부 네트워크 호출은 금지예요. 그래도 내부 Jira에는 연결하고 싶어요."
→ **로컬 stdio MCP 서버**. 개발자 PC의 하위 프로세스로 실행되어 노트북이 이미 가진 사내망/VPN 경로를 그대로 쓴다. 외부 호출도, 새 방화벽 예외도 필요 없다. (클라우드 프록시는 결국 외부 호출이고, 수동 복붙은 해결 가능한 문제를 장애물로 취급하는 것)

---

## Lesson 2. 승인 서버 거버넌스와 MCP 허용 목록

> **MCP 거버넌스는 신뢰의 문제가 아니라 설정의 문제다.**

### 심층 방어 5계층 중 Layer 02

| 계층 | 내용 | 소유 |
|---|---|---|
| 01 아이덴티티·환경 | SSO, SCIM, RBAC, 네트워크 접근 통제 | 고객 |
| **02 플랫폼·런타임 ← MCP 허용 목록** | 실행 환경, 샌드박스, egress, 커넥터 동의 | 공동 (플랫폼은 Anthropic, 정책은 고객) |
| 03 런타임 안전 | 분류기, 표면별 시스템 프롬프트, 사용 정책 | Anthropic |
| 04 모델 안전 | Constitutional AI, RLHF, 학습된 거부 | Anthropic |
| 05 관측·감사 | 감사 로그, Compliance API, OTel, 분석 | 전 계층 횡단 |

→ 고객에게 "통제되지 않는 커넥터 접근을 믿어 달라"는 게 아니라, 개발자가 무엇을 설치하든 독립적으로 동작하는 4개 계층 사이에서 고객의 정책 레버가 어디 있는지 보여주는 것이다.

### 두 개의 통제 평면

| | 사용자 설치 | 조직 배포 |
|---|---|---|
| 방법 | `~/.claude.json`이나 프로젝트 `.mcp.json`에 직접 추가 | **배포:** `managed-mcp.json`이 모든 사용자에게 고정 서버를 자동 적용<br>**필터링:** `managed-settings.json`의 `allowedMcpServers`로 직접 추가 가능한 서버를 제한 |
| 통제 | 완전히 막으려면 `managed-settings.json`에서 override 정책을 deny로 | `allowManagedMcpServersOnly: true`로 필터를 **강제** |

→ 두 통제는 별개이므로 **Day 0에 둘 다 설정**한다.

### Day 0 설정 (SSO/SCIM, 비용 보고와 함께 최소 필수 조건)

```jsonc
// managed-settings.json — allowedMcpServers 항목은 문자열이 아니라 객체여야 함
{
  "allowManagedMcpServersOnly": true,   // 미승인 설치 차단
  "allowedMcpServers": [
    { "serverUrl": "https://api.github.com/*" },
    { "serverUrl": "https://linear.app/mcp/*" },
    { "serverCommand": ["npx", "-y", "@playwright/mcp@latest"] },
    { "serverUrl": "https://*.client-internal.com/*" }
  ]
}
```

**`allowManagedMcpServersOnly: true`가 CISO의 "개발자가 아무거나 설치할 수 있나요?"에 대한 답이다.** 이 플래그가 없으면 목록은 권고일 뿐 강제되지 않는다.

### 신규 서버 요청 프로세스

1. **킥오프 전:** 팀이 이미 쓰는 MCP 서버를 목록화 → 별도 평가 없이 승인 목록에 바로 올린다
2. **Day 0:** 보안팀과 초기 허용 목록 로드, egress 도메인 확인, 사용자 설치 정책 설정. 목록을 48시간 전에 공유해서 회의는 검토가 아닌 확인 자리가 되게(1시간 이내)
3. **매주:** CoE 정기 회의에서 커넥터 승인. CoE 리드가 결정하고 보안팀이 비표준 egress 도메인을 검토. **같은 주 안에 처리**

### O/X

- 기본 설정에서는 org 허용 목록과 무관하게 로컬 `.mcp.json`으로 무엇이든 설치 가능 → **O.** 플래그가 있어야 강제된다
- egress 도메인 허용 목록은 미승인 서버가 설치돼도 외부 호출을 막을 수 있다 → **O.** 독립된 계층이다
- 허용 목록은 4주차에 설정해도 된다 → **X.** Day 0 필수 조건. 습관이 굳은 뒤에 거버넌스를 덧입히기는 더 어렵다
- OAuth 범위 MCP 연결은 개발자가 이미 접근 가능한 리소스만 접근한다 → **O**

### 시나리오 (금융 서비스 30일 활성화)

| 상황 | 정답 |
|---|---|
| 킥오프 전 개발자가 매일 Confluence MCP를 쓴다고 함 | **킥오프 전 MCP 인벤토리에 추가** → Day 0 목록에 바로 반영 |
| Day 0, 보안팀이 Playwright 평가에 2주 더 필요. 내일 온보딩 | **승인된 서버로 예정대로 시작하고 Playwright는 보류로 기록**, 승인 후 CoE 안건으로. Day 0에는 '완전한' 목록이 아니라 '안전한' 목록이 필요하다 |
| 2주차, 금요일 마감에 Figma MCP 필요. CoE는 목요일 | **목요일 CoE 안건으로 올리고, 승인되면 금요일에 사용** (먼저 로컬 설치하거나 CoE를 우회하지 말 것) |

---

## Lesson 3. MCP 터널(원격 MCP 릴레이): 로컬 개발 연결

> **허용 목록은 '무엇이 승인되는가'를, 릴레이는 '무엇에 실제로 닿을 수 있는가'를 다룬다.**

### 문제

승인된 내부 시스템이 사내망 안에서만 접속을 받는데, Claude Code는 거기에 닿지 못하는 개발자 노트북에서 돈다. 릴레이가 없으면 API 응답을 수동으로 복붙해야 한다.

### 언제 쓰나

| 직접 MCP 연결 | 원격 MCP 릴레이 |
|---|---|
| 개발자 환경에서 대상에 바로 닿을 때 (GitHub·Linear·Slack 같은 SaaS, VPN으로 접근 가능한 시스템) | 노트북이 넘을 수 없는 네트워크 경계 뒤에 있을 때 (특정 서브넷만 허용하는 내부 API, 에어갭 환경, 노트북용 방화벽 규칙을 안 열어주는 시스템) |

→ 릴레이는 지연과 장애 지점을 추가하므로 **직접 연결이 불가능할 때만** 쓴다.

### 구성

사내망 안에 MCP 서버를 두고 HTTP 또는 SSE 엔드포인트로 노출하면, 개발자는 VPN을 통해 접속한다.

```json
{
  "mcpServers": {
    "internal-api": { "type": "sse", "url": "https://mcp.client-internal.com/sse" }
  }
}
```

```bash
claude mcp add --transport sse internal-api https://mcp.client-internal.com/sse
```

- **플랫폼팀에 전할 핵심:** MCP 서버는 **MCP 프로토콜 트래픽(도구 호출과 응답)만** 전달한다. 내부 환경에 대한 일반 네트워크 접근을 주지 않는다. 보안 경계는 MCP 서버에 있다.
- **인증:** OAuth(지원 시), 정적 헤더(API 키·Bearer 토큰), 또는 `headersHelper`(사내 SSO, Kerberos, 단기 토큰 등 커스텀 인증).

### 시나리오

| 질문 | 정답 |
|---|---|
| 릴레이가 필요한 상황은? | **사내망에서만 접근 가능한 내부 Confluence.** GitHub는 SaaS라 직접 연결, 전사 배포는 `managed-mcp.json`의 일(도달성 문제가 아니라 배포 문제) |
| 보안 책임자: "터널로 어떤 트래픽이 흐르는지 몰라서 허용 못 합니다" | **터널이 무엇을 운반하는지 설명.** MCP 도구 호출과 응답만 오가며, VPN이 아니고 일반 네트워크 경로를 열지 않는다 |

---

## Lesson 4. Skills: 재사용 가능한 작업 전문성 패키징

> **고객의 표준은 아무도 읽지 않는 문서 속에 있다.** Skill은 그것을 Claude가 자동으로 적용하는 것으로 바꾼다.

### Skill이란

지시문, 스크립트, 템플릿이 담긴 폴더. **현재 작업과 관련 있을 때만 로드**되므로, 필요 없는 세션에는 오버헤드가 없다.

| 종류 | 내용 |
|---|---|
| **Capability Skills** | 특수 지시 없이는 잘 못하는 일: 고객 형식의 Office 파일, 복잡한 PDF 레이아웃, 독자적인 출력 템플릿 |
| **Knowledge Skills** | 조직 고유 워크플로우·표준: 코딩 규칙, 컴플라이언스 문구, PR 설명 요건, 사후 분석 형식, 브랜드 가이드 |

### 구조

```
api-docs-skill/
├── SKILL.md              ← 필수 (YAML frontmatter + 본문)
├── endpoint-template.yaml ← 필요할 때 로드
├── example-spec.yaml      ← 필요할 때 로드
└── apply_template.py      ← 필요하면 실행
```

```markdown
---
name: API Documentation Standards
description: Apply client's internal REST API documentation format
  when writing or reviewing any endpoint specification
---
## Overview
## Format Requirements
## Example
Read ./endpoint-template.yaml for the full schema
```

### 점진적 공개 (Progressive Disclosure)

| 단계 | Claude가 보는 것 | 결과 |
|---|---|---|
| 1. Scan | 이름 + description만 | 이 작업과 관련 있나? |
| 2. Load | SKILL.md 본문 전체 | 지시와 예시를 읽음 |
| 3. Fetch | 연결된 파일(필요 시) | 템플릿, 스펙, 스크립트 |

- 그래서 **description이 가장 중요한 한 줄**이다. 정확하지 않으면 로드돼야 할 때 안 되거나, 안 돼야 할 때 로드된다.
- Skill 10~20개를 설치해도 오버헤드는 개수가 아니라 관련성에 비례한다.
- `/skill-name`으로 직접 호출하거나 지시에서 명시적으로 언급해 강제로 로드할 수도 있다.

### Skill 기회 3가지 패턴

| 패턴 | 신호 | 예시 (영국 금융사, 50명) |
|---|---|---|
| **1. 경험으로 다듬어진 표준** | 반복 작업 + 확립된 품질 기준 + 결과가 들쭉날쭉 | 사후 보고서의 심각도 분류가 매번 다름 → 심각도 기준표와 필수 항목을 Skill로 |
| **2. 자료에 품질이 좌우됨** | 자료는 있는데 Claude에게 안정적으로 전달되지 않음 | 90페이지 API 보안 체크리스트를 매번 복붙 → 참조 파일로 번들 |
| **3. 능력 격차** | Claude가 모르는 형식이라 일관되게 틀림 | ISO 20022 XML 필드명을 지어냄 → 스키마, 필수 필드, 완전한 예시를 제공 |

**언제 만들고 언제 설치하나:** 킥오프 전에 인벤토리, 2주차에 심화 작업, 4주차에 팀 플러그인으로 패키징. 고객이나 Anthropic 마켓플레이스에 이미 있으면 새로 만들지 말고 먼저 확인할 것.

### O/X

- 설치된 모든 Skill의 SKILL.md 전체가 매 작업마다 로드된다 → **X**
- Skill에 Claude가 실행하는 스크립트를 포함할 수 있다 → **O**
- description은 장식이라 로드 여부에 영향이 없다 → **X.** description이 곧 트리거다

### 시나리오: 패턴 식별

| 상황 | 정답 |
|---|---|
| 개발자 3명이 60페이지 서비스 카탈로그를 각자 잘라서 붙여 넣고, 버전이 갈라지는 중 | **패턴 2** (자료는 있는데 안정적으로 전달되지 않음) |
| 같은 사고인데 엔지니어마다 P2, P4로 심각도가 다름. 런북에 기준표 있음 | **패턴 1** (기준표를 참조 파일로 두는 게 아니라 **지시로 인코딩**해야 일관성이 생김) |
| ISO 20022 XML이 모든 스키마 검증에서 실패 | **패턴 3** (일관되게 틀림 = 능력 격차. 200페이지 스펙을 참조로 두는 것만으로는 부족) |

---

## Lesson 5. 효과적인 Skill 작성

> **Skill은 만들었는데 Claude가 한 번도 로드하지 않는다** → 지시는 괜찮고 description이 문제다.

### description 작성 3원칙

| 모호함 | 정확함 |
|---|---|
| `Apply PR format` | `Apply the client's internal PR description format when creating or reviewing pull request descriptions. Includes required fields: summary, testing plan, reviewer checklist, and linked ticket.` |

1. **트리거 조건을 명시:** "PR 설명을 작성하거나 검토할 때"
2. **결과물·맥락을 명시:** "PR 설명 형식"
3. **핵심 자료를 언급:** "필수 항목: 요약, 테스트 계획, 리뷰어 체크리스트" → 비슷한 Skill이 여럿일 때 구분된다

### 오래가는 Skill의 원칙

- **하나에 집중:** Skill 하나에 워크플로우 하나. 코딩 표준, PR 형식, 보안 리뷰를 한 파일에 넣지 말 것
- **단순하게 시작:** 핵심부터, 실제 사용에서 드러난 빈틈으로 확장
- **예시 사용:** 규칙 세 문단보다 완성된 예시 하나가 낫다
- **점진적 테스트:** 변경할 때마다 테스트. 조용히 실패하는 Skill은 없는 것보다 나쁘다
- **버전 관리:** 고객 표준이 바뀌면 Skill도 바뀐다. 코드처럼 다룰 것

### 5단계 작성 예시 (TypeScript API 문서 Skill)

1. **트리거 정의:** "TypeScript 함수 시그니처, API 엔드포인트, 서비스 인터페이스 문서를 작성·검토할 때"
2. **description 작성:** 트리거 + JSDoc 형식, 필수 필드, 예시 패턴
3. **SKILL.md 핵심 작성:** 형식 명시, 완성 예시 하나, 필수 필드(`@param`, `@returns`, `@throws`, `@example`), 참조 파일은 링크
4. **참조 파일 번들:** `spec-template.yaml`
5. **테스트:** "이 엔드포인트 문서화해줘" → 로드 안 되면 description 수정, 로드됐는데 결과가 틀리면 지시 수정

**지름길:** "[작업]용 Skill을 만들어줘. 예시 5개는 이거야." → Claude가 폴더 구조, SKILL.md, 예시 파일을 생성한다. Knowledge Skill은 여기서 시작하는 게 빠르다.

### 시나리오

| 상황 | 정답 |
|---|---|
| "이 엔드포인트 문서화해줘"에 트리거될 description은? | **"Apply the client's TypeScript documentation standard when writing or reviewing function signatures, API endpoints, or service interfaces. Includes JSDoc format, required fields, and example patterns."** (트리거 + 형식 + 구분 표지. "Help with API documentation"은 너무 모호하고, 구분 표지가 없으면 비슷한 Skill끼리 구별이 안 됨) |
| 사후 분석 Skill이 경미한 사고에도 'lessons learned' 섹션을 넣음 (P1/P2에만 필요) | **SKILL.md에 "P1, P2에만 lessons learned 섹션 포함" 규칙을 명시.** 예시를 더 넣어도 Claude는 예시에서 규칙을 추론하지 않으므로 제약을 직접 써야 한다 |

---

## Lesson 6. Hooks: pre-task, post-edit, on-failure, on-completion

> **Hook은 Claude에 대한 지시가 아니라 시스템 레벨 트리거다.** Claude가 무엇을 하든 정해진 시점에 실행된다.

Jira pre-commit 게이트나 ServiceNow 변경 승인과 같은 강제 로직을, 공유 CI가 아닌 개발자 세션에서 로컬로 실행하는 것이다.

### 4가지 용도

- **알림:** 작업 완료 시 Slack 메시지, 입력 대기 시 소리
- **자동 포맷:** `.ts` 편집마다 Prettier, `.go`마다 gofmt
- **로깅:** 모든 도구 호출을 타임스탬프와 함께 기록 (컴플라이언스, 사고 검토)
- **강제:** PreToolUse에서 조건 확인 후 exit 2로 차단, PostToolUse에서 사후 대응

### 주요 라이프사이클 이벤트 5가지 (전체 20개 이상)

| 이벤트 | 시점 | 주 용도 |
|---|---|---|
| **PreToolUse** | 도구 실행 전 | **차단형 강제** |
| **PostToolUse** | 도구 실행 후 | 로깅, 자동 포맷, 후속 트리거 |
| **Notification** | 알림 발생 시 | Slack, 데스크톱 소리, 웹훅 |
| **Stop** | 메인 에이전트 작업 완료 시 | 요약 로깅, 정리 |
| **SubagentStop** | 서브 에이전트 완료 시 | 서브 에이전트 결과 처리 |

기타 고급 이벤트: SessionStart, PermissionRequest/PermissionDenied, FileChanged, ConfigChange 등.

> **가장 중요한 구분:** 행동 전에 막으려면 **PreToolUse**, 사후에 대응하려면 **PostToolUse**.

### 설정 예시 (`/hooks` 명령 또는 플러그인의 `hooks/hooks.json`)

```json
// 예시 1: 모든 Bash 호출 로깅 (PostToolUse)
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.command' | xargs -I {} sh -c 'echo \"[$(date)] Bash: {}\" >> ~/claude-bash-log.txt'"
      }]
    }]
  }
}
```

```json
// 예시 2: TypeScript 편집 후 자동 포맷 (PostToolUse)
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit",
      "hooks": [{
        "type": "command",
        "command": "jq -r '.tool_input.file_path' | { read file; [[ \"$file\" == *.ts ]] && npx prettier --write \"$file\"; }"
      }]
    }]
  }
}
```

- 도구 입력은 **stdin으로 JSON**이 들어오므로 `jq`로 파싱한다. `$TOOL_INPUT`, `$EDITED_FILE` 같은 변수는 **존재하지 않는다.**

### 종료 코드

| 코드 | 동작 |
|---|---|
| **0** | 성공, 계속 진행 |
| **1** | 실패지만 비차단. 오류가 Claude 컨텍스트에 표시되고 실행은 계속 |
| **2** | **도구 실행 차단.** PreToolUse hook이 exit 2를 반환할 때만 실제로 막힌다 |

⚠️ **보안 주의:** Hook은 개발자 세션과 **같은 권한**으로 셸 명령을 실행한다. 배포 전에 스크립트를 검토할 것. 손상된 hook은 내부 확산(lateral movement) 경로가 된다. 팀 플러그인에 들어가는 hook은 CoE가 소유하고 버전 관리한다.

### 도입 시점

- **1주차 Day 3~5:** 위험 낮고 효과 큰 두 가지부터. ① 입력 대기 알림 hook ② 주 언어 자동 포맷 hook
- **4주차 패키징:** 보안 로깅 hook, 컴플라이언스 hook을 팀 플러그인에 포함 → 거버넌스 하에서 운영된다는 증거

### O/X

- Edit에 대한 PostToolUse hook으로 검증 실패 시 저장을 막을 수 있다 → **X.** 이미 편집된 후다. PreToolUse를 써야 한다
- Hook 스크립트는 개발자 세션과 같은 권한으로 실행된다 → **O**
- 설정 파일 편집 전 컴플라이언스 검사는 Stop hook으로 → **X.** Stop은 세션이 끝날 때 실행된다. 파일 경로 matcher를 건 PreToolUse를 써야 한다

### 시나리오

| 상황 | 정답 |
|---|---|
| 세션에 티켓 번호가 없으면 `/config` 디렉터리 편집을 막고 싶음 | **Edit에 대한 PreToolUse.** 대상이 `/config`이고 티켓이 없으면 exit 2로 차단 (PostToolUse는 이미 바뀐 뒤, Stop은 감사일 뿐 강제가 아님) |
| 보안 감사 서브 에이전트가 끝나는 순간 Slack 알림 | **SubagentStop** (Stop은 메인 작업 종료라 늦고, Bash에 대한 PostToolUse는 메시지가 수십 개) |

---

## Lesson 7. 서브 에이전트와 병렬 오케스트레이션

> **작업이 컨텍스트 하나에 담기엔 너무 크다면, 서브 에이전트로 나눈다.**

### 서브 에이전트 vs 병렬 Claude 인스턴스

| | 서브 에이전트 | 병렬 Claude 인스턴스 |
|---|---|---|
| 정체 | 세션 안에서 생성되는 전문 미니 에이전트 | 별도 터미널의 독립 Claude Code |
| 컨텍스트 | 격리된 하위 컨텍스트, **결과를 메인 에이전트에 반환** | 인스턴스별 완전히 독립 |
| 용도 | 메인 컨텍스트를 오염시키지 않고 전문 작업 위임 | 각각 별도 PR로 나갈 수 있는 작업, 몇 시간짜리 독립 작업 |
| 조정 | 자동 (메인이 결과를 기다림) | 수동 (터미널 간 관리, TMUX 등) |

> 핵심 구분: **서브 에이전트는 결과를 돌려주고, 병렬 인스턴스는 각자 작업을 출시한다.** 이걸 헷갈리는 것이 가장 흔한 설계 오류다. 비유하면 메인 에이전트가 PM, 서브 에이전트가 워크스트림이고 산출물만 위로 올라온다.

### 구조: agents 디렉터리의 스펙 파일

```markdown
# agents/security-auditor.md
name: security-auditor
description: Scans for SQL injection, XSS, CSRF, and OWASP Top 10
  vulnerabilities in modified code. Returns a structured findings report.
tools: Read, Grep, Bash
```

- **description:** 메인 에이전트가 어떤 서브 에이전트를 부를지 결정하는 기준. Skill처럼 트리거 조건을 명시한다.
- **tools:** 필요한 도구만. 감사 에이전트에게 Edit, Write는 필요 없다.
- ※ 강의 예시는 `agents/`로 표기하지만(플러그인 구조 기준), 일반 프로젝트에서는 보통 `.claude/agents/`에 둔다.

### 대표 패턴: 3-에이전트 보안 감사 (고객에게 가장 먼저 제안할 것)

1. **메인 에이전트** – 기능 작성. 리뷰 준비가 되면 감사 서브 에이전트 호출
2. **감사 서브 에이전트** – OWASP Top 10 스캔. 격리된 컨텍스트에서 받은 코드만 보고, 추론 과정이 아니라 **구조화된 결과 보고서**만 반환
3. **수정 서브 에이전트** – 보고서를 받아 취약점을 수정하고 결과를 반환
4. **메인 에이전트** – 메인 컨텍스트는 끝까지 깨끗하게 유지되고, 감사 세부 내용은 들어오지 않는다

→ AI가 생성한 코드에 보안 거버넌스를 보여줘야 하는 고객에게 적합하다. 각 서브 에이전트의 작업과 출력을 기록할 수 있어 감사 추적이 남고, 개발 흐름도 느려지지 않는다.

### 컨텍스트 격리와 실패 처리

- **서브 에이전트가 보는 것:** 자기 시스템 프롬프트(스펙 파일), 받은 작업, 자기 도구 결과
- **보지 못하는 것:** 메인의 대화 기록, 메인 세션의 MCP 연결(명시적으로 넘기지 않는 한), 작업 설명에 없는 파일 상태 → **작업을 자기 완결적으로 설계**할 것
- **네트워크·자격증명:** 같은 세션 안에서 컨텍스트 창만 분리된 것이며, 프로세스나 네트워크 수준의 격리가 아니다. 민감한 작업을 맡기기 전에 네트워크 정책, 프록시, 자격증명이 예상대로 적용되는지 확인할 것
- **실패:** 서브 에이전트가 실패하면 메인은 오류 결과(예: `{ status: 'error', message: 'Parse failed: ...' }`)를 받는다. **자동 재시도는 없다.** 재시도, 건너뛰기, 에스컬레이션 같은 처리를 메인 에이전트 지시에 미리 써둬야 한다. 처리하지 않으면 파이프라인이 멈춘다. 실패한 감사 결과가 수정 에이전트로 넘어가는 것이 가장 흔한 실패 유형이므로 넘기기 전에 검증할 것.

### 시나리오

| 상황 | 정답 |
|---|---|
| IT: "프록시 정책 때문에 서브 에이전트 생성이 막힐 겁니다" | **무엇이 막히는지 IT에 먼저 물어본다.** 서브 에이전트는 세션 안에서 생성되며 별도 아웃바운드 연결을 만들지 않으므로 제한이 해당하지 않을 수 있다 |
| 감사 서브 에이전트가 minified 파일 파싱 실패, lint와 test는 정상 완료 | **오류를 기록하고 해당 파일을 수동 검토로 표시한 뒤, lint와 test 결과로 진행** (전체 중단이나 자동 재시도는 오답) |

---

## Lesson 8. 플러그인 마켓플레이스와 의존성 버전 관리

> **Day 30. 당신이 떠난 뒤에도 무엇이 남는가?**

### 플러그인 구조

플러그인은 `plugin.json` 매니페스트와 Claude Code 확장 전체를 담은 폴더다. 명령 하나로 Skill, hook, MCP 설정, 슬래시 명령, 서브 에이전트 스펙을 한꺼번에 설치한다.

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json      ← 필수: 매니페스트
├── commands/            ← 슬래시 명령
├── agents/              ← 서브 에이전트 스펙
├── skills/              ← Skills
│   └── coding-standards/SKILL.md
├── hooks/
│   └── hooks.json
└── .mcp.json            ← MCP 서버 정의
```

```json
{
  "name": "finserv-activation",
  "version": "1.0.0",
  "description": "Claude Code activation plugin for FinServ Corp engineering team"
}
```

```bash
/plugin install @org/plugin-name
/plugin browse
/plugin update @org/plugin-name
```

설치 후에는 각 구성요소가 제대로 로드되는지 확인할 것 (MCP 인증 문제, hook 권한 오류, 플러그인 충돌로 일부가 동작하지 않을 수 있음).

### 번들 적합성 기준

| 구성요소 | 이런 경우 번들 |
|---|---|
| Skill | 최소 2명 이상이 사용하고 활성화 기간 중 실전에서 검증됨 |
| MCP 설정 | 고객 보안팀 승인 + 안정적이고 테스트된 연결 |
| Hook | 동작·테스트 완료, 한 사람이 아니라 팀 전체에 가치가 있음 |
| 서브 에이전트 | 실제 시나리오에서 최소 1회 검증 |
| 슬래시 명령 | 팀이 보존하고 싶은 자주 쓰는 프롬프트 패턴 (일회성 실험 제외) |

**Day 27 전에 CoE와 나눌 대화:** 플러그인이 엔게이지먼트 후 어디에 있을지(CoE가 소유하는 엔터프라이즈 마켓플레이스나 공유 레지스트리). 마켓플레이스가 없으면 구축에 시간이 걸리므로 **3주차 CoE 회의에서** 꺼낼 것.

### 배포 범위 2가지

| 팀 플러그인 | 엔터프라이즈 마켓플레이스 |
|---|---|
| 팀 고유 Skill·hook, 팀이 소유 | 승인된 MCP 설정, 보안 hook, 컴플라이언스 Skill. CoE가 검증·게시 |
| CoE 릴리스 주기 없이 팀이 반복 개선 | 승인 워크플로우를 거치는 엄격한 릴리스 주기 |

→ 하나의 거대 플러그인은 둘을 묶어버려 작은 변경도 CoE 전체 릴리스를 거쳐야 한다. **소유자와 릴리스 주기를 분리**한다.

**Day 1 거버넌스 피치:** 관리 콘솔에서 승인된 플러그인을 RBAC·SCIM 그룹에 연결 → ① 플러그인 생성·승인 ② 마켓플레이스 게시 ③ SCIM 그룹에 할당. 그러면 해당 그룹에 프로비저닝된 개발자는 **첫날부터 승인된 설정을 자동으로 받는다.** 도구 접근과 통제된 설정이 함께 도착한다. 정책 문서를 배포하고 지키길 바라는 방식이 아니다.

### 버전 관리 (Day 30에 CoE와 확립할 3가지)

1. **MCP 서버 버전 고정** – 업스트림에서 도구 입력 스키마가 바뀌면 워크플로우가 깨질 수 있다. 매니페스트에 고정하고 CoE가 테스트 후 의도적으로 올린다
2. **플러그인 자체 버전 관리** – `/plugin update`로 업데이트하며, 버전 번호로 어떤 설정을 쓰는지 지원팀이 바로 알 수 있다
3. **Day 30 패키지 문서화** – 구성 내용, 번들된 MCP 서버 버전, 소유자를 기록한 인수인계 문서. 없으면 아무도 유지 관리 못 하는 블랙박스가 된다

### O/X

- 팀 플러그인과 CoE 플러그인은 함께 배포되므로 릴리스 주기를 공유해야 한다 → **X.** 소유자와 주기가 별개다
- 보안 검토 안 된 MCP 설정도 "experimental" 라벨을 붙이면 번들 가능 → **X.** 라벨이 보안 검토를 대신하지 않는다. 개발자는 번들된 것을 안전하다고 여긴다

### 시나리오: Day 28 패키징 검토

구성요소: 보안 감사 서브 에이전트(3스프린트 검증), API 문서 Skill(10명 중 8명 사용), ISO 20022 Skill(2명, 테스트 중), 로깅 hook(테스트 완료), Jira MCP(지난주 IT 승인)
→ **보안 감사 에이전트, API 문서 Skill, 로깅 hook, Jira MCP를 번들.** ISO 20022 Skill은 기준을 충족할 때까지 개발 브랜치에 둔다.

### 플러그인 만들기

1. `<client-name>-activation/` 생성: `.claude-plugin/plugin.json`, `skills/`, `hooks/`, `agents/`, `.mcp.json`
2. 앞 레슨 산출물 복사: Lesson 5 SKILL.md → `skills/<name>/`, Lesson 6 hooks.json → `hooks/`, Lesson 7 서브 에이전트 스펙 → `agents/`, Lesson 1 `.mcp.json`
3. `plugin.json` 작성: 이름, 버전, 설명(필수 3개 필드) + MCP 버전 고정
4. `/plugin install ./<client-name>-activation/` 후 Skill 로드, hook 실행, MCP 연결 확인

---

## 전체 정리

- **MCP:** Tools, Resources, Prompts를 추가하고 `.mcp.json`(프로젝트) 또는 `~/.claude.json`(사용자)으로 범위를 정한다. 외부 호출이 막히면 로컬 stdio. 인증 방식을 반드시 확인한다.
- **거버넌스:** `allowManagedMcpServersOnly: true` + `allowedMcpServers` + egress 통제를 Day 0에 설정한다. 신규 요청은 주간 CoE에서 같은 주 안에 처리한다.
- **릴레이:** 네트워크 경계 뒤 시스템에만 쓴다. MCP 트래픽만 운반하며 일반 네트워크 경로가 아니다.
- **Skills:** 점진적 공개 구조이므로 description이 트리거다. 3가지 패턴(표준 일관성 / 자료 전달 / 능력 격차)으로 기회를 찾는다. 하나에 집중하고, 예시를 넣고, 버전 관리한다.
- **Hooks:** 막으려면 PreToolUse + exit 2, 대응은 PostToolUse. 입력은 stdin JSON. 세션 권한으로 실행되므로 CoE가 소유한다.
- **서브 에이전트:** 결과를 반환하고 컨텍스트가 격리된다. 실패 처리는 메인 지시에 직접 쓴다. 3-에이전트 보안 감사 패턴을 먼저 제안한다.
- **플러그인:** 적합성 기준을 통과한 것만 번들한다. 팀과 CoE는 분리하고, SCIM 그룹으로 자동 배포하며, 버전을 고정하고 문서화한다.

다음 코스: **Security and Governance** (인증, 접근 관리, 감사 로그, 데이터 레지던시, 컴플라이언스)
