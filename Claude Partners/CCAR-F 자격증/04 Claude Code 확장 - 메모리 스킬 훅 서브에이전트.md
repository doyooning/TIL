#CCAR-F
#Claude
#ClaudeCode

---

# D3-2. Claude Code 확장: 메모리 · 스킬 · 훅 · 서브에이전트

"같은 요구를 어떤 메커니즘으로 구현할 것인가"를 고르는 문제가 자주 나온다.

## 0. 메커니즘 선택 기준 (핵심 비교표)

| 메커니즘 | 로딩 시점 | 강제력 | 용도 |
| --- | --- | --- | --- |
| **CLAUDE.md** | 매 세션 시작 시 항상 | 없음(컨텍스트) | 규약, 빌드 명령, 아키텍처 사실 |
| **`.claude/rules/`** | 항상 또는 `paths` 매칭 시 | 없음 | 경로별 규칙, 대형 프로젝트 분할 |
| **Skill** | 호출될 때만 | 없음 | 반복 가능한 절차·워크플로 |
| **Subagent** | 위임될 때, 별도 컨텍스트 | 도구 제한 O | 대용량/격리 작업, 병렬 리서치 |
| **Hook** | 생명주기 이벤트마다 | **있음(셸 수준 강제)** | "항상 X 해라/하지 마라" |
| **권한 규칙** | 도구 호출마다 | **있음** | 접근 통제 |
| **auto memory** | 매 세션(인덱스) | 없음 | Claude가 스스로 축적하는 학습 |

> **"매 커밋 전에 반드시 ~해야 한다"** 같은 요구는 CLAUDE.md가 아니라 **훅**으로 구현해야 한다. CLAUDE.md는 강제 장치가 아니다.

---

## 1. CLAUDE.md (메모리)

### 위치와 로드 순서 (넓은 범위 → 좁은 범위)

| 스코프 | 위치 | 공유 대상 |
| --- | --- | --- |
| Managed policy | macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`<br>Linux/WSL `/etc/claude-code/CLAUDE.md`<br>Windows `C:\Program Files\ClaudeCode\CLAUDE.md` | 조직 전체 |
| User | `~/.claude/CLAUDE.md` | 나(모든 프로젝트) |
| Project | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md` | 팀(버전 관리) |
| Local | `./CLAUDE.local.md` | 나(이 프로젝트, gitignore) |

- 작업 디렉터리와 **그 위 모든 상위 디렉터리**의 CLAUDE.md / CLAUDE.local.md가 시작 시 로드된다
- **덮어쓰지 않고 전부 연결(concatenate)** 된다. 파일시스템 루트 → 작업 디렉터리 순서라서 **가까운 파일이 마지막에** 읽힌다
- 같은 디렉터리 안에서는 `CLAUDE.md` 다음에 `CLAUDE.local.md`
- **하위 디렉터리**의 CLAUDE.md는 시작 시가 아니라 Claude가 그 디렉터리 파일을 읽을 때 로드된다
- `--add-dir`로 추가한 디렉터리의 CLAUDE.md는 기본적으로 로드되지 않음 → `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1`

### import

```text
See @README for project overview and @package.json for npm commands.

# Additional Instructions
- git workflow @docs/git-instructions.md
- @~/.claude/my-project-instructions.md
```

- 상대 경로는 **import를 포함한 파일 기준**, 재귀 import는 **최대 4홉**
- 코드 스팬/코드 블록 안의 `@path`는 무시됨 (백틱으로 감싸면 리터럴)
- 프로젝트 메모리 파일이 **작업 디렉터리 밖 파일**을 import하면 최초 1회 승인 대화상자가 뜬다. 거절하면 영구 비활성
- import한 파일도 시작 시 컨텍스트에 들어가므로 **분할해도 토큰은 줄지 않는다**

### AGENTS.md
Claude Code는 `AGENTS.md`를 직접 읽지 않는다. `CLAUDE.md`에서 `@AGENTS.md`로 import하거나 심볼릭 링크를 걸어야 한다(Windows는 import 권장). `/init`은 Cursor·Copilot 규칙을 읽어 CLAUDE.md에 반영하고, `/import`는 다른 코딩 에이전트 설정(명령·서브에이전트·스킬·MCP 서버)을 가져온다.

### 작성 모범 사례
- **200줄 이하** 목표(4 MiB 초과 파일은 스킵). 길수록 준수율이 떨어진다
- 마크다운 헤더·불릿으로 구조화
- 검증 가능한 구체성: "2-space indentation" > "코드를 잘 포매팅해라"
- 모순되는 지시는 임의로 선택되므로 주기적으로 정리
- 블록 단위 HTML 주석(`<!-- ... -->`)은 컨텍스트 주입 전에 제거된다 → 사람용 메모 용도
- 모노레포에서 남의 팀 파일 제외: `claudeMdExcludes` (절대 경로 글롭, 레이어 간 배열 병합, **관리형 정책 CLAUDE.md는 제외 불가**)

### `.claude/rules/`

```text
your-project/
├── .claude/
│   ├── CLAUDE.md
│   └── rules/
│       ├── code-style.md
│       ├── testing.md
│       └── security.md
```

- `paths` 프론트매터가 **없는** 규칙: 시작 시 로드(우선순위는 `.claude/CLAUDE.md`와 동일)
- **경로 스코프 규칙**: 매칭 파일을 읽을 때만 로드 → 컨텍스트 절약

```markdown
---
paths:
  - "src/**/*.{ts,tsx}"
  - "tests/**/*.test.ts"
---

# API Development Rules
- 모든 엔드포인트는 입력 검증을 포함한다
```

- 글롭: `**/*.ts`, `src/**/*`, `*.md`, `src/components/*.tsx`. 중괄호 확장은 규칙당 1,000패턴/4MiB 예산
- `~/.claude/rules/`는 모든 프로젝트에 적용(사용자 규칙이 먼저, 프로젝트 규칙이 더 높은 우선순위)
- 심볼릭 링크 지원. 단, 작업 디렉터리 밖을 가리키면 외부 import와 동일하게 승인이 필요하고 승인 후에도 `paths` 없는 규칙만 로드된다

### Auto memory

Claude가 스스로 기록하는 메모. 4가지 타입: `user`, `feedback`, `project`, `reference`.

| 항목 | 값 |
| --- | --- |
| 저장 위치 | `~/.claude/projects/<project>/memory/` (git 저장소 단위, worktree 공유) |
| 인덱스 | `MEMORY.md` — **처음 200줄 또는 25KB**만 매 세션 로드 |
| 토픽 파일 | 시작 시 로드되지 않고 필요할 때 읽음 |
| 끄기 | `/memory` 토글, `autoMemoryEnabled: false`, `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` |
| 위치 변경 | `autoMemoryDirectory` (절대 경로 또는 `~/`) |
| 특성 | 머신 로컬. 세션 트랜스크립트 보존 기간(`cleanupPeriodDays`) 청소에서 제외 |

- Claude는 코드에서 유도 가능한 정보(아키텍처, 파일 경로)나 CLAUDE.md가 이미 말하는 내용은 저장하지 않는다
- 메인 대화의 auto memory는 **서브에이전트에 로드되지 않는다**(fork는 예외). 서브에이전트는 `memory` 필드로 자체 메모리를 가진다

### 압축 후 살아남는 것
프로젝트 루트 CLAUDE.md, 범위 없는 규칙, auto memory, plan mode의 계획 → **디스크에서 재주입**. 대화 중에만 준 지시는 사라진다. → [[06 컨텍스트 관리와 신뢰성]]

---

## 2. Skills

`SKILL.md` 파일 하나로 만드는 커스텀 명령/워크플로. **필요할 때만 로드**되므로 큰 참고 자료를 두어도 평소 컨텍스트 비용이 0이다.

### 파일 구조

```markdown
---
name: deploy-prod
description: Deploy to production safely. Use when the user asks to ship or release.
disable-model-invocation: true
allowed-tools: Bash(git *) Read
---

배포 절차...
```

### 주요 프론트매터

| 필드 | 의미 |
| --- | --- |
| `name` | 표시 이름(명령 이름은 디렉터리명) |
| `description` | **Claude가 자동 호출 여부를 판단하는 근거** |
| `disable-model-invocation: true` | 모델 자동 호출 금지(사용자만) — 부작용 있는 작업에 사용 |
| `user-invocable: false` | `/` 메뉴에서 숨김(모델만 호출) |
| `allowed-tools` | 사전 승인 도구(그 턴에만 유효, 다음 메시지에서 해제) |
| `context: fork` | 격리된 서브에이전트에서 실행 |
| `agent` | fork 시 서브에이전트 타입(`Explore`/`Plan`/`general-purpose`) |
| `paths` | 활성화를 제한하는 글롭 |
| `model`, `effort` | 세션 설정 오버라이드 |
| `arguments` | 명명된 위치 인자(`$name` 치환) |

### 위치와 범위

| 위치 | 경로 | 로드 범위 |
| --- | --- | --- |
| 개인 | `~/.claude/skills/<name>/SKILL.md` | 내 모든 프로젝트 |
| 프로젝트 | `.claude/skills/<name>/SKILL.md` | 이 저장소 |
| 중첩 | `<subdir>/.claude/skills/<name>/SKILL.md` | 그 하위 트리 |
| 엔터프라이즈 | 관리형 설정의 `.claude/skills/` | 조직 전체 |
| 플러그인 | `<plugin>/skills/<name>/SKILL.md` | 플러그인 활성 위치 |

**이름 충돌**: Enterprise > Personal > Project, 내 스킬 > 번들 스킬, 플러그인 스킬은 `/plugin-name:skill-name`으로 로드.

### 동적 컨텍스트 주입 · 치환

- 인라인 명령: `` !`git diff HEAD` ``, 여러 줄은 ` ```! ` 블록 — **Claude가 스킬을 보기 전에 실행**된다
- 치환: `$ARGUMENTS`, `$0`/`$1`, `$name`, `${CLAUDE_SESSION_ID}`, `${CLAUDE_SKILL_DIR}`, `${CLAUDE_PROJECT_DIR}`

### 생명주기
- 호출 시 스킬 본문이 **하나의 메시지로 컨텍스트에 진입**하고 이후 턴에도 남는다(권한 부여는 남지 않음)
- 압축 후 재주입: **스킬당 5,000토큰 / 전체 25,000토큰 상한**, 오래된 것부터 제거. 잘릴 때 **앞부분이 남으므로 중요한 지시는 SKILL.md 위쪽에** 둘 것
- SKILL.md는 500줄 이하 권장, 상세 자료는 보조 파일로 분리해 링크

### 권한
- 프로젝트 스킬은 **워크스페이스 신뢰 필요**
- 스킬 차단: `deny`에 `Skill`(전체), `Skill(commit)`, `Skill(code-review *)`
- 번들 스킬 끄기: `disableBundledSkills: true`, 개별은 `skillOverrides`

---

## 3. Hooks

생명주기 이벤트에서 셸 명령/HTTP/MCP 도구/프롬프트를 실행. **Claude의 판단과 무관하게 실행되는 강제 계층**이다.

### 주요 이벤트

| 이벤트 | 시점 | matcher |
| --- | --- | --- |
| `SessionStart` / `SessionEnd` | 세션 시작·재개 / 종료 | 시작 방식 / 종료 이유 |
| `UserPromptSubmit` | 사용자 프롬프트 제출 전 | X |
| `UserPromptExpansion` | 슬래시 명령 확장 전 | 명령명 |
| **`PreToolUse`** | 도구 실행 전 (차단 가능) | 도구명 |
| **`PostToolUse`** | 도구 성공 후 | 도구명 |
| `PostToolUseFailure` | 도구 실패 후 | 도구명 |
| `PermissionRequest` / `PermissionDenied` | 권한 결정 시 / auto 모드 거부 시 | 도구명 |
| `Stop` / `StopFailure` | 응답 완료 / API 오류 종료 | — / 오류 유형 |
| `SubagentStart` / `SubagentStop` | 서브에이전트 시작·완료 | 에이전트 타입 |
| `PreCompact` / `PostCompact` | 컨텍스트 압축 전·후 | 트리거 원인 |
| `InstructionsLoaded` | CLAUDE.md / rules 로드 | 로드 이유 |
| `PreModelSwitch` / `PostModelSwitch` | 모델 전환 전(차단 가능)·후 | 모델명 |
| `Notification`, `FileChanged`, `ConfigChange`, `CwdChanged`, `Elicitation` 등 | | |

### 설정 구조

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "if": "Bash(rm *)",
            "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/block-rm.sh",
            "timeout": 600
          }
        ]
      }
    ]
  }
}
```

- `type`: `command` / `http` / `mcp_tool` / `prompt` / `agent`
- `if`: 권한 규칙 문법으로 추가 필터
- `async`: 백그라운드 실행, `asyncRewake`: 오류 시 Claude 깨우기
- matcher 규칙: `"*"` 또는 생략 = 전체 / 영숫자·`_`·`-`·공백·`|`·`,` = 정확 매치(`Edit|Write`) / 그 외 문자 = 정규식(`^Notebook`, `mcp__.*`)

### 입력 JSON (stdin) 공통 필드
`session_id`, `prompt_id`, `transcript_path`, `cwd`, `scratchpad_dir`, `permission_mode`, `hook_event_name`, `effort`
도구 이벤트 추가: `tool_name`, `tool_input`, `tool_use_id`

### 출력 규약 (시험 포인트)

| Exit code | 동작 |
| --- | --- |
| **0** | 성공. stdout의 JSON을 파싱해 적용 |
| **2** | **차단.** stderr가 차단 사유가 되고 JSON은 무시됨 |
| 기타 | JSON이 있으면 적용, 없으면 비차단 경고 |
| 타임아웃 | 출력 무시(기본 600초, auto 모드에서는 30초). `PreModelSwitch`는 차단 |

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Destructive command blocked",
    "additionalContext": "...",
    "updatedInput": { },
    "systemMessage": "사용자에게 보이는 메시지",
    "continue": true
  }
}
```

이벤트별 제어 필드: `PreToolUse` → `permissionDecision`(allow/deny/skip) / `PostToolUse` → `additionalContext`, `systemMessage` / `UserPromptSubmit` → `continue`, `additionalContext` / `PermissionRequest` → `decision` / `Stop` → `continue`

### 훅 위치와 보안
- `~/.claude/settings.json`(개인) / `.claude/settings.json`(프로젝트, 공유) / `.claude/settings.local.json` / 관리형 정책 / 플러그인 `hooks/hooks.json` / 스킬·서브에이전트 프론트매터
- 프로젝트 훅은 **워크스페이스 신뢰 후 실행**
- 관리자는 `allowManagedHooksOnly`로 사용자·프로젝트 훅 차단
- HTTP 훅은 `allowedHttpHookUrls`·`httpHookAllowedEnvVars`로 제한, 환경변수는 `allowedEnvVars`에 명시해야 전달됨
- 경로는 `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_PLUGIN_ROOT}` 사용 권장

---

## 4. Subagents

### 파일과 우선순위

| 위치 | 범위 | 우선순위 |
| --- | --- | --- |
| 관리형 설정 | 조직 | 1 (최상) |
| `--agents` CLI (JSON) | 세션 | 2 |
| `.claude/agents/` | 프로젝트 | 3 |
| `~/.claude/agents/` | 모든 프로젝트 | 4 |
| 플러그인 `agents/` | 플러그인 | 5 |

```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and best practices
tools: Read, Glob, Grep, Bash
model: sonnet
memory: project
---

You are a code reviewer. Focus on quality, security, and best practices.
```

### 프론트매터

| 필드 | 의미 |
| --- | --- |
| `name` (필수) | 소문자-하이픈 |
| `description` (필수) | **자동 위임 판단 근거.** 짧게(합계 15,000토큰 초과 시 경고) |
| `tools` | 화이트리스트(생략 시 전체 상속) |
| `disallowedTools` | 블랙리스트(먼저 적용 후 `tools`로 필터) |
| `model` | `sonnet`/`opus`/`haiku`/`fable`/전체 ID/`inherit` |
| `permissionMode` | 메인이 bypass/acceptEdits/auto면 무시됨 |
| `isolation: worktree` | 임시 git worktree에서 실행 |
| `maxTurns`, `effort` | 실행 제한·추론 강도 |
| `memory` | `user`/`project`/`local` — 자체 영구 메모리 |
| `skills` | 미리 로드할 스킬 |
| `mcpServers`, `hooks` | 전용 MCP 서버·생명주기 훅 |
| `background: true`, `color`, `initialPrompt` | |

**모델 우선순위**: Agent 도구 호출의 `model` 파라미터 > 프론트매터 > `CLAUDE_CODE_SUBAGENT_MODEL` > 메인 대화 모델. (`CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1`로 강제)

### 내장 서브에이전트

| 이름 | 목적 | 도구 |
| --- | --- | --- |
| **Explore** | 코드베이스 탐색(읽기 전용). CLAUDE.md·git status를 스킵해 빠르고 저렴 | Read, Grep, Glob |
| **Plan** | 계획 모드의 컨텍스트 수집 | 읽기 전용 |
| **general-purpose** | 복잡한 다단계 작업 | 전체 |

비활성화: `deny`에 `Agent(Explore)`, `--disallowedTools`, 또는 `CLAUDE_CODE_DISABLE_EXPLORE_PLAN_AGENTS=1`

### 호출 방법
1. 자연어 언급 → Claude가 위임 여부 결정
2. `@"code-reviewer (agent)"` → **강제 실행**
3. `claude --agent code-reviewer` 또는 설정의 `agent` → 메인 세션 전체가 그 정의를 사용

### 포그라운드 vs 백그라운드
- 포그라운드: 완료까지 블로킹, 권한 프롬프트 직접 처리, 모든 도구
- 백그라운드: 동시 진행, 권한 프롬프트가 메인 세션에 표시, **도구 집합이 제한**됨

자세한 격리·오케스트레이션은 → [[01 에이전트 아키텍처와 오케스트레이션]]

---

## 5. Plugins

- 스킬 · 서브에이전트 · 훅 · MCP 서버 · 슬래시 명령을 **하나로 묶어 배포**하는 단위
- 마켓플레이스로 배포(`extraKnownMarketplaces`는 신뢰 후 적용)
- 로컬 테스트: `--plugin-dir <경로|zip>`, `--plugin-url <zip url>`
- 플러그인 내 경로는 `${CLAUDE_PLUGIN_ROOT}`, 데이터는 `${CLAUDE_PLUGIN_DATA}`
- `claude plugin eval`로 플러그인 eval 스위트 실행

## 연결 노트

- [[03 Claude Code 설정과 권한]]
- [[01 에이전트 아키텍처와 오케스트레이션]]
- [[07 CLI 슬래시 명령 치트시트]]
