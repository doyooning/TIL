#CCAR-F
#Claude
#ClaudeCode
#CheatSheet

---

# CLI · 슬래시 명령 치트시트

## 1. 설치

- **macOS / Linux / WSL**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

- **Windows PowerShell**

```powershell
irm https://claude.ai/install.ps1 | iex
```

- **Homebrew** (자동 업데이트 X — `brew upgrade claude-code` 필요)

```bash
brew install --cask claude-code
```

- **WinGet** (자동 업데이트 X)

```powershell
winget install Anthropic.ClaudeCode
```

네이티브 설치는 백그라운드 자동 업데이트. Windows에서는 Bash 도구를 위해 **Git for Windows** 설치가 권장되며, 없으면 PowerShell이 셸 도구로 쓰인다.

## 2. 실행 표면(Surface)

터미널 CLI / VS Code·Cursor 확장 / JetBrains 플러그인 / 데스크톱 앱 / 웹(claude.ai/code) / 모바일.
**모두 같은 엔진**을 쓰므로 저장소의 CLAUDE.md·설정·MCP 서버가 공통 적용된다.

| 하고 싶은 것 | 방법 |
| --- | --- |
| 로컬 세션을 폰/다른 기기에서 이어가기 | Remote Control |
| 외부 이벤트를 실행 중 세션에 밀어넣기 | Channels |
| 로컬에서 시작 → 모바일에서 계속 | `claude --cloud` + 모바일 앱 |
| 정기 실행 | Routines(클라우드) / 데스크톱 예약 작업(로컬) / `/loop`(세션 내) |
| PR 리뷰·이슈 트리아지 자동화 | GitHub Actions / GitLab CI/CD |
| Slack에서 버그 리포트 → PR | Slack 연동 |
| 웹앱 디버깅 | Chrome 연동 |
| 커스텀 에이전트 구축 | Agent SDK |

## 3. 기본 명령

```bash
claude                        # 대화형 세션
claude "이 프로젝트 설명해줘"     # 초기 프롬프트와 함께
claude -p "이 함수 설명해줘"      # 비대화형(print/headless) 실행 후 종료
cat logs.txt | claude -p "에러 분석해줘"
```

## 4. 주요 플래그

### 세션
| 플래그 | 설명 |
| --- | --- |
| `--continue`, `-c` | 현재 디렉터리의 최근 대화 이어가기 |
| `--resume`, `-r <session>` | 세션 ID/이름으로 재개 |
| `--fork-session` | 재개 시 새 세션 ID로 복사 |
| `--name`, `-n` | 세션 표시 이름 |

### 모델·추론
| 플래그 | 설명 |
| --- | --- |
| `--model` | 모델 지정 (`sonnet`, `opus`, `haiku`, 전체 ID) |
| `--effort` | `low`/`medium`/`high`/`xhigh`/`max` |
| `--fallback-model` | 기본 모델 불가 시 대체 체인 |
| `--advisor <model>` | advisor 도구 활성화 |

### 권한·도구
| 플래그 | 설명 |
| --- | --- |
| `--permission-mode` | `default`/`acceptEdits`/`plan`/`auto`/`dontAsk`/`bypassPermissions` |
| `--dangerously-skip-permissions` | `bypassPermissions`와 동일 |
| `--allowedTools` | 프롬프트 없이 실행할 도구 |
| `--disallowedTools` | 거부 규칙(도구 제거 또는 호출 제한) |
| `--permission-prompt-tool` | 권한 프롬프트를 처리할 MCP 도구 |
| `--restricted` | 제한 모드(명령 도구·사용자 설정 제거). bypass 불가 |

### 파일·설정
| 플래그 | 설명 |
| --- | --- |
| `--add-dir` | 추가 작업 디렉터리 |
| `--settings <file|json>` | 이번 세션 설정 오버라이드 |
| `--setting-sources` | 로드할 설정 소스 선택(`user`/`project`/`local`) |
| `--mcp-config` | MCP 서버를 JSON 파일/문자열에서 로드 |
| `--plugin-dir`, `--plugin-url` | 플러그인 로드 |

### 시스템 프롬프트·서브에이전트
| 플래그 | 설명 |
| --- | --- |
| `--append-system-prompt` / `--append-system-prompt-file` | 기본 시스템 프롬프트 뒤에 추가 |
| `--system-prompt` / `--system-prompt-file` | 기본 시스템 프롬프트 교체 |
| `--exclude-dynamic-system-prompt-sections` | 동적 섹션을 첫 사용자 메시지로 이동(캐시 재사용 개선) |
| `--agent <name>` | 이 세션 전체를 해당 서브에이전트 정의로 실행 |
| `--agents <json>` | 서브에이전트를 JSON으로 즉석 정의 |
| `--append-subagent-system-prompt` | 모든 서브에이전트 프롬프트에 추가 |

### 출력·제어
| 플래그 | 설명 |
| --- | --- |
| `--output-format` | `text` / `json` / `stream-json` |
| `--input-format` | `text` / `stream-json` |
| `--json-schema` | 스키마에 맞는 검증된 JSON 출력 |
| `--include-partial-messages` | 부분 스트리밍 이벤트 포함 |
| `--include-hook-events` | 훅 생명주기 이벤트 포함 |
| `--max-turns` | 에이전트 턴 상한(print 모드) |
| `--max-budget-usd` | 지출 상한 |
| `--verbose`, `-v` / `--debug` | 상세/디버그 출력 |

### 진단·기타
| 플래그 | 설명 |
| --- | --- |
| `--bare` | 훅·스킬·명령·플러그인 자동 검색 건너뛰기 |
| `--safe-mode` | 모든 커스터마이징 비활성화(설정 문제 격리) |
| `--bg`, `--background` | 백그라운드 에이전트로 시작 |
| `--cloud` | 클라우드(웹) 세션 생성 |
| `--teleport` | 웹/모바일 세션을 터미널로 가져오기 |

## 5. 슬래시 명령

| 명령 | 용도 |
| --- | --- |
| `/init` | CLAUDE.md 생성(있으면 개선 제안). `CLAUDE_CODE_NEW_INIT=1`이면 대화형 다단계 흐름 |
| `/import` | 다른 코딩 에이전트 설정(AGENTS.md, MCP, 명령, 스킬) 가져오기 |
| `/memory` | CLAUDE.md·auto memory 열람/편집, auto memory 토글 |
| `/context [all]` | 컨텍스트 사용량 시각화 |
| `/compact [지시]` | 대화 요약으로 공간 확보 |
| `/autocompact <토큰>` | 자동 압축 임계치 설정 |
| `/clear [name]` | 새 대화 시작 |
| `/resume`, `/branch`, `/fork`, `/subtask` | 세션 재개·분기·포크 |
| `/rewind` | 체크포인트로 되돌리기 / 부분 요약 |
| `/model`, `/effort` | 모델·추론 강도 전환 |
| `/permissions` | 권한 규칙 관리(auto 모드 분류기 규칙 포함) |
| `/mcp` | MCP 서버 상태·인증·재연결 |
| `/config` | 설정 메뉴 |
| `/status` | 로드된 설정 파일·관리형 소스 확인 |
| `/doctor` | 설정 진단 및 자동 수정 |
| `/debug` | 디버그 로깅 |
| `/cd`, `/add-dir` | 작업 디렉터리 이동·추가 |
| `/plan` | 계획 모드 |
| `/diff` | 작업 트리 변경 검토 |
| `/code-review [level] [--fix]` | 코드 리뷰 (`ultra`는 클라우드 멀티 에이전트) |
| `/security-review` | 브랜치 변경에 대한 보안 검토 |
| `/verify`, `/debug`, `/batch`, `/run` | 번들 스킬 |
| `/tasks`, `/background` | 백그라운드 작업 |
| `/schedule`, `/loop` | 정기 실행 |
| `/export` | 대화 내보내기 |
| `/help` | 도움말 |

### 실행 중 입력 시 처리
- `/status`, `/tasks`, `/usage` → 즉시 실행(중단 없음)
- `/model`, `/effort` → 현재 응답 완료 후 적용
- `/permissions`, `/theme` → 즉시 다이얼로그

### 커스텀 슬래시 명령
슬래시 명령은 **스킬로 구현**된다. `.claude/skills/<name>/SKILL.md`를 만들면 `/<name>`으로 호출된다.
- 인자: `/skill-name arg1 arg2` → `$ARGUMENTS`, `$0`, `$1`, 명명 인자
- 스킬 체이닝: `/skill-a /skill-b /skill-c do XYZ` (최대 6개)
- 자세한 내용 → [[04 Claude Code 확장 - 메모리 스킬 훅 서브에이전트]]

## 6. `claude mcp` 하위 명령

```bash
claude mcp add --transport http <name> <url>
claude mcp add --transport stdio <name> -- <command> [args]
claude mcp add-json <name> <json>
claude mcp list
claude mcp get <name>
claude mcp remove <name>
claude mcp login <name>
claude mcp logout <name>
claude mcp reset-project-choices
```

## 7. 주요 환경변수

| 변수 | 용도 |
| --- | --- |
| `ANTHROPIC_API_KEY` | API 키 인증 |
| `ANTHROPIC_MODEL` | 파일의 `model` 설정보다 우선 |
| `ANTHROPIC_DEFAULT_MODEL` | 파일에 `model`이 없을 때만 적용 |
| `MAX_MCP_OUTPUT_TOKENS` | MCP 도구 출력 제한(기본 25,000) |
| `MCP_TIMEOUT` | MCP 서버 시작 타임아웃(ms) |
| `CLAUDE_CODE_MCP_TOOL_IDLE_TIMEOUT` | MCP 도구 유휴 타임아웃(기본 5분) |
| `ENABLE_TOOL_SEARCH` | `auto` / `false` — MCP 도구 스키마 선로딩 제어 |
| `CLAUDE_CODE_SUBAGENT_MODEL` / `_FORCE` | 서브에이전트 모델 지정/강제 |
| `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` | 중첩 깊이(기본 3) |
| `CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS` | 동시 서브에이전트(기본 20) |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | auto memory 비활성화 |
| `CLAUDE_CODE_DISABLE_EXPLORE_PLAN_AGENTS` | 내장 Explore/Plan 에이전트 비활성화 |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | `--add-dir` 디렉터리의 CLAUDE.md도 로드 |
| `CLAUDE_CONFIG_DIR`, `CLAUDE_CODE_PROJECT_DIR_NAME` | 설정/프로젝트 디렉터리 지정 |

## 8. `.claude` 디렉터리 구조 요약

```text
project/
├── CLAUDE.md                    # 프로젝트 지침 (또는 .claude/CLAUDE.md)
├── CLAUDE.local.md              # 개인 지침 (gitignore)
├── .mcp.json                    # project 스코프 MCP 서버 (커밋)
└── .claude/
    ├── settings.json            # 팀 공유 설정 (커밋)
    ├── settings.local.json      # 개인 설정 (gitignore)
    ├── rules/                   # 주제별·경로별 규칙
    ├── skills/<name>/SKILL.md   # 커스텀 스킬/슬래시 명령
    ├── agents/<name>.md         # 커스텀 서브에이전트
    ├── hooks/                   # 훅 스크립트
    └── launch.json              # 프리뷰용 개발 서버 정의
```

```text
~/.claude/
├── settings.json                # 사용자 설정
├── CLAUDE.md                    # 사용자 지침
├── rules/ · skills/ · agents/   # 사용자 스코프
└── projects/<project>/memory/   # auto memory (MEMORY.md + 토픽 파일)

~/.claude.json                   # local/user 스코프 MCP 서버, 신뢰 상태
```

## 연결 노트

- [[03 Claude Code 설정과 권한]]
- [[04 Claude Code 확장 - 메모리 스킬 훅 서브에이전트]]
- [[02 도구 설계와 MCP 통합]]
