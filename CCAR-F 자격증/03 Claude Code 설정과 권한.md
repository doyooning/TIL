#CCAR-F
#Claude
#ClaudeCode

---

# D3-1. Claude Code 설정 & 권한 (D3 = 20%)

## 1. 설정 파일과 스코프

Claude Code는 4개의 설정 파일 + 조직의 관리형 설정을 읽는다.

| 스코프 | 파일 | 영향 범위 | 용도 |
| --- | --- | --- | --- |
| User | `~/.claude/settings.json` | 이 머신의 모든 프로젝트 | 테마, 에디터 모드, 기본 모델, 개인 권한 규칙 |
| Shared project | `.claude/settings.json` | 해당 폴더의 모든 사람 (git 커밋) | 팀 권한, 훅, 플러그인, 프로젝트 환경변수 |
| Project local | `.claude/settings.local.json` | 나, 이 프로젝트만 (gitignore) | 개인 오버라이드 |
| Command line | `claude --settings <file-or-json>` | 이번 세션만 | 일회성 |
| Managed | `managed-settings.json` / MDM / claude.ai 콘솔 | 조직 전체 | 정책 강제 |

## 2. 설정 우선순위 (시험 필수)

**높음 → 낮음**

1. **Managed settings** (조직) — 어떤 것도 덮어쓸 수 없음 (`--settings`도 불가)
2. **Command line** (`--settings`, 플래그)
3. **Project local** (`.claude/settings.local.json`)
4. **Shared project** (`.claude/settings.json`)
5. **User** (`~/.claude/settings.json`)

### 핵심 규칙

- **리스트형 키는 병합된다.** `permissions.allow` 같은 배열은 각 파일의 항목이 합쳐진다(덮어쓰기 X)
  - 예외: `fallbackModel`(순서가 의미를 가지므로 통째로), `modelPicker`, `availableModels`, `modelSettings`
- **환경변수는 우선순위 스택의 레벨이 아니다.** 키마다 다르다
  - `ANTHROPIC_MODEL`(쉘) → 어떤 파일의 `model`보다 우선
  - `ANTHROPIC_DEFAULT_MODEL` → 파일에 `model`이 없을 때만 적용
- **보안 키는 낮은 레벨의 더 엄격한 값이 이긴다.** (managed 예외)
  - `disableClaudeAiConnectors: true`(어느 스코프든), `enableArtifact: false`, `isolatePeerMachines: true`, `maxEffortLevel`의 더 낮은 상한 등
- `permissions.defaultMode`의 `auto` / `bypassPermissions`는 **project·local 설정에서는 효력이 없다**. user/managed 설정이나 `--permission-mode`로 지정해야 한다
- 잘못된 JSON이면 해당 파일 또는 항목을 건너뛴다 → `/status`로 로드된 파일 확인

## 3. 권한 규칙 문법

형식: `Tool` 또는 `Tool(specifier)`

| 규칙 | 효과 |
| --- | --- |
| `Bash` | 모든 Bash 명령 (`Bash(*)`와 동일) |
| `Bash(npm run build)` | 정확히 그 명령만 |
| `Bash(npm run *)` | `npm run build`, `npm run test --watch`, `npm run` |
| `Read(./.env)` | 해당 파일 읽기 |
| `WebFetch(domain:example.com)` | 해당 호스트 |
| `Agent(Explore)` | Explore 서브에이전트 |
| `mcp__puppeteer__puppeteer_navigate` | 특정 MCP 도구 |
| `Cd(~/code/**)` | `/cd` 이동 대상 디렉터리 |

### 와일드카드 규칙

- `*`는 공백 포함 임의 텍스트와 매칭. **`*` 앞의 문자열은 문자 그대로** 매칭되므로 `*`는 서브커맨드 뒤에 두는 것이 안전
- 끝의 `*` 앞에 공백이 있으면 **맨몸 명령도 매칭**: `Bash(ls *)` → `ls`도 매칭, 하지만 `lsof`는 매칭 안 됨
- 공백이 없으면: `Bash(ls*)` → `lsof`도 매칭
- `Bash(ls:*)`는 `Bash(ls *)`와 동일(`:*`는 끝에서만 인식)
- **위험 예시**: `Bash(git * main)`은 `git -c core.fsmonitor=<script> diff main`까지 매칭된다 → 넓은 allow 규칙은 우회 경로를 만든다

### 파라미터 매칭 (deny/ask 전용)

`Tool(param:value)` 형식. 예: `Agent(model:opus)`, `Agent(isolation:worktree)`, `Bash(run_in_background:true)`

- 최상위 직접 필드만 매칭(중첩 불가), 규칙 하나당 파라미터 하나
- 값에 `*` 사용 가능, 생략된 파라미터는 매칭되지 않음
- **주요 콘텐츠 필드는 이 방식으로 매칭할 수 없다**: Bash/PowerShell의 `command`, Read/Edit/Write의 `file_path`, Grep/Glob의 `path`, WebFetch의 `url`

### 도구 이름 와일드카드
- deny/ask에서 `"*"`(모든 도구), `"mcp__*"`(모든 MCP 도구) 사용 가능
- allow에서는 `mcp__<server>__` 접두사 뒤에만 글롭 허용

## 4. 평가 순서 — **deny → ask → allow**

- 순서대로 **첫 매칭이 결과를 결정**하며, 규칙의 구체성은 순서를 바꾸지 않는다
- 따라서 `deny: Bash(aws *)`는 `allow: Bash(aws s3 ls)`를 무력화한다 (**deny 규칙에 예외를 둘 수 없다**)
- **어느 스코프에서든 deny가 있으면 다른 스코프의 allow로 뚫을 수 없다.** user deny가 project allow를 막고, managed deny는 `--allowedTools`로도 못 뚫는다
- 맨 이름 deny 규칙(`Bash`)은 **도구 자체를 컨텍스트에서 제거**한다(모델이 존재조차 모름). 괄호가 붙은 패턴 deny는 도구는 남기고 호출만 거부
- 권한 규칙은 **모델이 아니라 Claude Code가 강제**한다. 프롬프트나 CLAUDE.md는 시도를 바꿀 뿐 허용 범위를 바꾸지 못한다

### 훅과의 관계
- `PreToolUse` 훅은 권한 프롬프트 **전에** 실행된다
- 훅이 `"allow"`를 반환해도 **deny/ask 규칙은 그대로 적용**된다
- 훅이 **exit 2**로 차단하면 권한 규칙 평가 전에 호출이 중단되므로 allow 규칙보다 우선한다

## 5. 권한 모드

| 모드 | 프롬프트 없이 실행되는 것 | 적합한 상황 |
| --- | --- | --- |
| `default` (**Manual**) | 읽기만 | 모든 행동을 직접 검토, 민감한 작업 |
| `acceptEdits` | 읽기 + 파일 편집 + 일반 파일시스템 명령(`mkdir`, `touch`, `mv`, `cp`) | 코드 반복 작업 |
| `plan` | 읽기(+ auto 모드 가능 시 분류기 승인 명령) | 변경 전 코드베이스 탐색 |
| `auto` | 전부, 단 백그라운드 안전성 검사(분류기 모델)가 검토 | 긴 작업, 프롬프트 피로 감소 |
| `dontAsk` | 읽기 + 사전 승인 도구. **프롬프트가 뜰 상황은 전부 거부** | 잠긴 CI·스크립트 |
| `bypassPermissions` | 전부 | 격리된 컨테이너/VM **전용** |

- CLI/확장/데스크톱 UI에서는 `default`가 **Manual**로 표시되며 `manual` 별칭도 받는다
- 전환: `Shift+Tab`, `--permission-mode <mode>`, `permissions.defaultMode`
- Pro/Max/Team 플랜의 기본 시작 모드는 **auto 모드**

### 어떤 모드도 자동 승인하지 않는 것
- 명시적 `ask` 규칙에 매칭된 도구
- 조직이 `ask`로 설정한 커넥터 도구
- 사용자 상호작용이 필요한 도구(`AskUserQuestion`, `requiresUserInteraction` MCP 도구)
- **critical path**를 대상으로 하는 `rm` / `rmdir` (`rm -rf /`, `rm -rf ~`) — allow 규칙도 PreToolUse 훅의 allow도 통하지 않음
- 교차 세션 메시징 안전장치
- `permissions.blockReadsOutsideWorkingDirectories`가 켜졌을 때 작업 디렉터리 밖 읽기

### bypassPermissions 제약
- 세션 시작 시에만 활성화 가능(`--permission-mode bypassPermissions`, `--dangerously-skip-permissions`, user/managed의 `defaultMode`)
- Linux/macOS에서 **root/sudo로는 실행 거부**(인식된 샌드박스 내부는 예외)
- **Claude Code on the web은 설정 파일의 `bypassPermissions`/`dontAsk`를 무시**한다 → 저장소에 커밋된 설정으로 클라우드 세션을 bypass 모드로 시작시킬 수 없다
- `--restricted`로 시작한 세션에서는 거부됨
- 관리자는 `permissions.disableBypassPermissionsMode` / `permissions.disableAutoMode`를 `"disable"`로 설정해 차단

### 보호 경로(Protected paths)
`.git`, `.claude` 등 저장소·설정 상태 경로에 대한 쓰기는 `bypassPermissions`(및 bypass가 가능한 plan 세션)를 제외하면 자동 승인되지 않는다.

## 6. 작업 디렉터리와 워크스페이스 신뢰

- 기본: 실행한 디렉터리. 확장 방법 3가지
  - 시작 시 `--add-dir <path>`
  - 세션 중 `/add-dir`
  - 영구 설정 `permissions.additionalDirectories`
- `/cd <path>`로 주 작업 디렉터리 이동(대화 유지, 새 디렉터리의 CLAUDE.md 로드)
- UNC 등 네트워크 경로는 작업 디렉터리로 추가 불가(Windows에서는 드라이브 매핑 사용)

### 신뢰(trust)가 필요한 것
프로젝트 `.claude/settings.json`의 다음 항목은 **워크스페이스 신뢰 수락 후에만** 적용된다.
- `permissions.allow` 규칙
- `permissions.additionalDirectories`
- `extraKnownMarketplaces`
- 대부분의 `env` 값
- 프로젝트 훅 / 프로젝트 스킬

`deny`와 `ask`는 신뢰 이전에도 적용된다(제한은 항상 유효).

신뢰 저장 방식:
- git 저장소 안 → **저장소 루트** 기준(중첩 저장소·서브모듈 제외). worktree는 메인 체크아웃 기준
- 저장소 밖 → 시작한 디렉터리 기준, 하위 디렉터리 포함
- 홈 디렉터리에서 시작 → 그 세션에만 유효, 디스크에 저장하지 않음
- **신뢰 대화상자는 대화형 세션에서만** 표시된다. `claude -p`나 SDK 세션에는 뜨지 않는다

## 7. 예시 설정

```json
{
  "permissions": {
    "defaultMode": "acceptEdits",
    "allow": [
      "Bash(npm run *)",
      "Bash(git commit *)",
      "Read",
      "mcp__github__get_pull_request"
    ],
    "ask": [
      "Bash(docker *)"
    ],
    "deny": [
      "Bash(git push *)",
      "Read(./.env)",
      "Read(./secrets/**)",
      "Agent(Explore)",
      "WebFetch"
    ],
    "additionalDirectories": ["../shared-lib"],
    "disableBypassPermissionsMode": "disable"
  },
  "env": {
    "NODE_ENV": "development"
  },
  "claudeMdExcludes": ["**/monorepo/other-team/CLAUDE.md"],
  "autoMemoryEnabled": true
}
```

## 8. 관리자 통제 (Enterprise)

| 목적 | 설정 |
| --- | --- |
| 특정 도구/명령/경로 차단 | `permissions.deny` (managed) |
| 관리형 권한 규칙만 허용 | `allowManagedPermissionRulesOnly` |
| 관리형 훅만 허용 | `allowManagedHooksOnly` |
| 샌드박스 강제 | `sandbox.enabled` |
| 로그인 방식/조직 제한 | `forceLoginMethod`, `forceLoginOrgUUID` |
| 조직 공통 지침 | 관리형 `CLAUDE.md` 또는 `claudeMd` 키 |
| MCP 서버 접근 통제 | managed MCP (`managed-mcp`) |

**구분 포인트**: 관리형 *설정*은 클라이언트가 **기술적으로 강제**하고, 관리형 *CLAUDE.md*는 **행동 지침**일 뿐 강제가 아니다.

## 9. 클라우드 세션의 설정

`claude --cloud` / Claude Code on the web 세션은 내 머신이 아니라 **저장소를 새로 클론한 클라우드 환경**에서 실행되므로, 머신 로컬 설정(`~/.claude/settings.json` 등)은 그대로 전달되지 않는다. 저장소에 커밋된 설정과 클라우드 환경 설정이 기준이 된다.

## 10. 설정 확인·문제 해결

| 상황 | 조치 |
| --- | --- |
| 어떤 설정 파일이 로드됐는지 | `/status` |
| 컨텍스트에 실제 로드된 메모리 파일 | `/context` |
| 설정 UI로 변경 | `/config` |
| 권한 규칙 확인/편집 | `/permissions` |
| 전반 진단 | `/doctor` |
| 커스터마이징 전부 끄고 테스트 | `claude --safe-mode` 또는 `--bare` |

## 연결 노트

- [[04 Claude Code 확장 - 메모리 스킬 훅 서브에이전트]]
- [[02 도구 설계와 MCP 통합]]
- [[07 CLI 슬래시 명령 치트시트]]
