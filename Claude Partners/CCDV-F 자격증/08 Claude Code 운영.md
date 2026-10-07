#CCDV-F
#Claude
#ClaudeCode

---

# D3. Claude Code 운영 (3.1%)

> 비중은 작지만 "개발자가 Claude Code를 어떻게 쓰고 자동화하는가"를 묻는다. 깊은 설정 문법보다 **무엇을 할 수 있고 어디에 무엇이 있는지**를 알면 충분하다.
> 더 깊은 내용은 [[03 Claude Code 설정과 권한]], [[04 Claude Code 확장 - 메모리 스킬 훅 서브에이전트]] (CCAR-F 노트) 참조.

## 1. Claude Code란

코드베이스를 읽고, 파일을 수정하고, 명령을 실행하고, 개발 도구와 통합되는 **에이전틱 코딩 도구**. 터미널·IDE·데스크톱 앱·웹에서 실행되며, **모두 같은 엔진**을 쓰므로 저장소의 CLAUDE.md·설정·MCP 서버가 공통 적용된다.

### 설치
```bash
curl -fsSL https://claude.ai/install.sh | bash
```
```powershell
irm https://claude.ai/install.ps1 | iex
```
Homebrew(`brew install --cask claude-code`), WinGet(`winget install Anthropic.ClaudeCode`)도 가능하나 **자동 업데이트가 없다**. 네이티브 설치는 백그라운드 자동 업데이트.

## 2. 기본 사용

```bash
claude                        # 대화형 세션
claude "이 프로젝트 설명해줘"
claude -p "이 함수 설명해줘"    # 비대화형(headless), 실행 후 종료
cat logs.txt | claude -p "에러 분석해줘"
```

### 자주 쓰는 플래그
| 플래그 | 설명 |
| --- | --- |
| `--continue`, `-c` | 최근 대화 이어가기 |
| `--resume`, `-r <session>` | 특정 세션 재개 |
| `--model` | 모델 지정 |
| `--effort` | 추론 강도 |
| `--permission-mode` | `default`/`acceptEdits`/`plan`/`auto`/`dontAsk`/`bypassPermissions` |
| `--allowedTools` / `--disallowedTools` | 도구 허용·차단 |
| `--add-dir` | 추가 작업 디렉터리 |
| `--settings` / `--setting-sources` | 설정 오버라이드·소스 선택 |
| `--mcp-config` | MCP 서버를 JSON에서 로드 |
| `--append-system-prompt` / `--system-prompt` | 시스템 프롬프트 추가·교체 |
| `--agent` / `--agents` | 서브에이전트 지정·정의 |
| `--output-format` | `text` / `json` / `stream-json` |
| `--max-turns`, `--max-budget-usd` | 실행 상한 |
| `--safe-mode`, `--bare` | 커스터마이징 비활성화(진단용) |

## 3. 헤드리스 실행과 자동화 (개발자 관점)

```bash
# CI에서 번역 자동화
claude -p "translate new strings into French and raise a PR for review"

# 변경 파일만 보안 검토
git diff main --name-only | claude -p "review these changed files for security issues"

# 구조화된 출력
claude -p --output-format json "query"
```

| 자동화 수단 | 설명 |
| --- | --- |
| **GitHub Actions / GitLab CI** | PR 리뷰·이슈 트리아지 자동화 |
| **Routines** | 클라우드에서 정기 실행. API 호출·GitHub 이벤트로도 트리거 |
| **데스크톱 예약 작업** | 로컬 파일·도구에 접근하며 정기 실행 |
| **`/loop`** | 세션 내에서 프롬프트 반복 |
| **Agent SDK** | 완전 커스텀 에이전트 → [[04 에이전트와 워크플로]] |

## 4. 설정 계층 (핵심만)

| 스코프 | 파일 |
| --- | --- |
| User | `~/.claude/settings.json` |
| Shared project | `.claude/settings.json` (커밋) |
| Project local | `.claude/settings.local.json` (gitignore) |
| Command line | `claude --settings` |
| Managed | `managed-settings.json` / MDM / 콘솔 |

**우선순위(높음 → 낮음)**: Managed → Command line → Project local → Shared project → User.
리스트형 키(`permissions.allow` 등)는 **덮어쓰지 않고 병합**된다.

## 5. 권한

### 규칙 문법
`Tool` 또는 `Tool(specifier)` — 예: `Bash(npm run *)`, `Read(./.env)`, `WebFetch(domain:example.com)`, `mcp__github__get_pull_request`, `Agent(Explore)`

### 평가 순서: **deny → ask → allow**
- 첫 매칭이 결과를 결정하고 **구체성은 순서를 바꾸지 않는다**
- 어느 스코프의 deny든 다른 스코프의 allow를 이긴다
- 맨 이름 deny(`Bash`)는 **도구를 컨텍스트에서 제거**한다

### 권한 모드
| 모드 | 프롬프트 없이 실행되는 것 |
| --- | --- |
| `default`(Manual) | 읽기만 |
| `acceptEdits` | 읽기 + 파일 편집 + 기본 파일시스템 명령 |
| `plan` | 탐색만, 소스 편집 금지 |
| `auto` | 전부, 백그라운드 분류기가 검토 |
| `dontAsk` | 사전 승인된 것만, 나머지는 **거부** |
| `bypassPermissions` | 전부 — **격리 컨테이너 전용** |

## 6. 프로젝트 컨텍스트

| 메커니즘 | 위치 | 로딩 |
| --- | --- | --- |
| **CLAUDE.md** | `./CLAUDE.md`, `./.claude/CLAUDE.md`, `~/.claude/CLAUDE.md`, `./CLAUDE.local.md` | 매 세션 시작 시 항상 |
| **`.claude/rules/`** | 주제별 `.md` | 항상 또는 `paths:` 매칭 시 |
| **Skills** | `.claude/skills/<name>/SKILL.md` | **호출될 때만** |
| **Subagents** | `.claude/agents/<name>.md` | 위임될 때, 별도 컨텍스트 |
| **Hooks** | `settings.json`의 `hooks` | 생명주기 이벤트마다 |
| **auto memory** | `~/.claude/projects/<project>/memory/` | `MEMORY.md` 첫 200줄/25KB |

- CLAUDE.md는 **200줄 이하** 권장. 시스템 프롬프트가 아니라 **사용자 메시지로 전달**되므로 강제력이 없다
- "항상 X 해야 한다" 같은 강제 요구는 **훅**으로 구현한다
- `/init`로 CLAUDE.md 초안 생성, `/memory`로 편집, `/context`로 실제 로드 여부 확인

## 7. 알아두면 좋은 슬래시 명령

| 명령                                          | 용도                 |
| ------------------------------------------- | ------------------ |
| `/init`                                     | CLAUDE.md 생성       |
| `/memory`, `/context`                       | 메모리 편집·컨텍스트 사용량 확인 |
| `/compact`, `/clear`                        | 압축·초기화             |
| `/permissions`                              | 권한 규칙 관리           |
| `/mcp`                                      | MCP 서버 상태·인증       |
| `/model`, `/effort`                         | 모델·추론 강도           |
| `/status`, `/doctor`                        | 설정 확인·진단           |
| `/code-review`, `/security-review`, `/diff` | 검토                 |
| `/plan`                                     | 계획 모드              |

## 8. MCP 연결

```bash
claude mcp add --transport http <name> <url>
claude mcp add --transport stdio <name> -- <command> [args]
claude mcp list / get / remove / login / logout
```
스코프는 `--scope local|project|user`. 팀 공유는 **project 스코프(`.mcp.json` 커밋)**. → [[06 도구와 MCP]]

## 연결 노트

- [[04 에이전트와 워크플로]]
- [[07 보안과 안전]]
- [[07 CLI 슬래시 명령 치트시트]] (CCAR-F 노트 — 전체 플래그·명령)
