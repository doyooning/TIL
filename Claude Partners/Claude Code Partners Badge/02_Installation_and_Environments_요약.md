# Installation and Environments 요약 (Claude Partner Badge: Claude Code)

총 5개 레슨: ① CLI 설치와 사전 요건 ② IDE 연동 ③ 터미널 패턴과 셸 설정 ④ 개발 컨테이너와 샌드박스 ⑤ 헤드리스·CI 모드

이 코스의 산출물: **Installation Configuration Pack** (IT·DevOps 팀에 넘기는 설치 설정 문서)

---

## Lesson 1. CLI 설치와 OS·플랫폼별 사전 요건

> **설치는 명령어 한 줄. 진짜 일은 거기까지 가는 준비 과정이다.**
> 설치 당일에 처음 환경을 점검한다면 이미 늦은 것이다.

### 킥오프 전에 IT와 확인할 3가지

1. 대상 OS가 지원되는가
2. 고객 네트워크에서 Anthropic 엔드포인트에 접근 가능한가
3. 개발자 그룹의 인증 방식은 무엇인가

→ 모두 개발자 없이 IT와의 사전 통화로 확인 가능하다.

### 플랫폼 지원

| OS | 내용 |
|---|---|
| **macOS** | 네이티브 지원(Apple Silicon·Intel 동일). 사내 프록시/커스텀 CA가 있으면 `NODE_EXTRA_CA_CERTS` 설정 |
| **Linux** | Ubuntu 20.04+, Debian 10+, Alpine 3.19+ 지원. `HTTPS_PROXY`(또는 `HTTP_PROXY`), `NODE_EXTRA_CA_CERTS` 사용 |
| **Windows (네이티브)** | WSL 없이 PowerShell/CMD에서 실행. 대부분의 팀에 더 간단. Git for Windows는 선택이지만 권장(없으면 Git Bash 대신 PowerShell 사용) |
| **Windows (WSL)** | Linux 툴체인이나 샌드박싱이 필요할 때. 관리형 PC에서는 IT가 활성화해야 함. WSL 2 권장 |

### 설치 명령어

```bash
# macOS / Linux
curl -fsSL https://claude.ai/install.sh | bash

# Windows PowerShell (권장)
irm https://claude.ai/install.ps1 | iex

# Windows CMD
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

### 설치 후 인증 방식

- **브라우저 로그인**: Enter를 누르면 브라우저에서 Anthropic 계정(Team/Enterprise)으로 로그인. 대화형 개발자의 표준 방식이며 IT의 키 관리가 필요 없다.
- **API 키 입력**: 헤드리스, CI, 자동화용. 콘솔에서 키를 생성하고 안전하게 배포(환경변수, 시크릿 볼트 등)하며 주기적으로 교체해야 하므로 IT 협업이 필요하다.

### 엔터프라이즈 환경의 흔한 장애 요인 4가지

1. **네트워크 접근** – 다음 도메인이 열려 있어야 한다. 방화벽 규칙 변경은 며칠이 걸리므로 가장 먼저 IT에 요청할 것.
   - `api.anthropic.com` (API)
   - `claude.ai`, `platform.claude.com` (인증)
   - `downloads.claude.ai` (설치·자동 업데이트)
   - `raw.githubusercontent.com` (변경 로그)
2. **프록시·인증서** – Zscaler, CrowdStrike 등은 루트 인증서가 OS 신뢰 저장소에 있으면 대부분 추가 설정 없이 동작한다. 커스텀 CA·mTLS도 지원한다.
3. **인증 구조** – 대화형은 브라우저 로그인, CI는 API 키. 두 방식을 섞어 쓸 수도 있다.
4. **Windows 네이티브 vs WSL** – WSL은 관리형 이미지에서 기본 비활성일 수 있어 IT 티켓이 필요할 수 있다.

**시나리오:** IT 담당자와 30분 통화에서 첫 질문은? → **"어떤 엔드포인트에 접근해야 하나요?"(네트워크)**. 설치 자체를 막는 요인이고, 고치는 데 가장 오래 걸린다.

### 설치 검증

| 명령 | 용도 |
|---|---|
| `claude --version` | 설치 확인. "command not found"가 나오면 PATH 미반영 → 터미널을 재시작하거나 셸 프로필을 다시 로드 |
| `claude doctor` | 설정 검증. 잘못된 설정을 파일·필드 단위로 알려준다. 전체 배포 전 대표 머신에서 실행할 것 |
| `/status` (세션 내) | 프록시·게이트웨이 설정이 제대로 적용됐는지 확인 |

### 트러블슈팅

- `curl: (7) Failed to connect to claude.ai port 443` → 네트워크가 HTTPS 아웃바운드를 차단하고 있다. 차단된 호스트명을 IT에 전달할 것.
- 설치·인증 후 `command not found: claude` → PATH 문제. 새 터미널을 열면 해결된다(인증 실패나 실행 디렉터리 문제가 아님).

---

## Lesson 2. IDE 연동: VS Code와 JetBrains

> **두 개의 진입점, 하나의 엔진. 개발자의 IDE에 맞춰 진입점을 고른다.**

### 설치 경로

| IDE | 설치 방법 |
|---|---|
| **VS Code** | 마켓플레이스에서 "Claude Code" 검색 또는 `vscode:extension/anthropic.claude-code`. 확장에 CLI가 포함되어 있고 전용 패널이 추가된다 |
| **Cursor** | 같은 확장: `cursor:extension/anthropic.claude-code`. 기능 동일 |
| **기타 VS Code 포크** (Devin Desktop, Kiro 등) | Open VSX 레지스트리나 확장 뷰에서 설치 |
| **JetBrains** (IntelliJ, PyCharm 등) | **CLI + 플러그인 둘 다 설치 필요.** CLI가 엔진이고 플러그인은 IDE 연동만 담당한다. 통합 터미널에서 `claude` 실행 |

JetBrains 사용 시 IT에 알릴 점: 일부 엔터프라이즈 환경에서는 통합 터미널이 기본으로 막혀 있을 수 있고, 플러그인 설치 후 IDE를 완전히 재시작해야 한다.

### 첫날 시연할 3가지 기능

1. **Diff 보기** – 변경 사항이 터미널이 아니라 IDE의 기본 diff 뷰어에 나란히 표시된다. `/config`에서 설정 가능.
2. **선택 영역 컨텍스트** – 현재 선택한 코드와 열린 탭이 자동으로 공유된다. 함수를 드래그하고 질문하면 바로 그 코드에 대해 답한다.
3. **파일 참조 단축키** – `@app.ts#5-10`처럼 특정 파일과 줄 범위를 참조한다.
   - VS Code: `Option+K`(Mac) / `Alt+K`(Win·Linux)
   - JetBrains: `Cmd+Option+K`(Mac) / `Alt+Ctrl+K`(Win·Linux)

### `/config` 설정

세션 안에서 여는 대화형 설정 메뉴. 설정은 여러 레벨에 존재한다: 엔터프라이즈 관리형 설정 → 커맨드라인 플래그 → 프로젝트 `.claude/settings.json` → 사용자 `~/.claude/settings.json`. 엔터프라이즈 배포에서는 어느 레벨이 설정을 소유하고 사용자가 덮어쓸 수 있는지 확인해야 한다.

IDE 관련 주요 설정:

| 설정 | 키 |
|---|---|
| 외부 터미널에서 실행 시 IDE 자동 연결 | `autoConnectIde` |
| 첫 연결 시 IDE 확장 자동 설치 | `autoInstallIdeExtension` |
| 마지막 응답을 외부 에디터에 표시 | `externalEditorContext` |
| 입력 키 바인딩(normal / vim) | `editorMode` |

**시나리오:** "저는 VS Code 말고 Cursor를 쓰는데 괜찮나요?" → **지원된다. VS Code와 같은 방식으로 설치하면 된다.** 답이 "된다"일 때는 망설이지 말고 분명하게 말할 것. 한 명만 다른 설정으로 돌리면 팀에 불필요한 마찰이 생긴다.

---

## Lesson 3. 터미널 패턴과 셸 설정

### 인증 방식과 환경변수

- 대화형 개발자는 브라우저 로그인, CI·자동화는 API 키, AWS·GCP 배포 고객은 Bedrock·Vertex 라우팅.
- API 키는 셸 프로필(`~/.zshrc`, `~/.bashrc`)이나 시크릿 저장소에 둔다. **`ANTHROPIC_API_KEY`가 설정되어 있으면 구독 로그인보다 우선한다.**

**퀴즈:** 사내 HTTPS 프록시 + TLS 검사 + 버전 고정 정책이 있는 고객이라면, 첫 인증 전에 설정할 변수는?
→ `ANTHROPIC_API_KEY`, `HTTPS_PROXY`, `NODE_EXTRA_CA_CERTS`, `DISABLE_AUTOUPDATER` (4개).
`CLAUDE_CODE_USE_BEDROCK`은 AWS 경유 고객에게만 해당하고, `CLAUDE_CODE_MAX_TURNS`는 나중에 비용 통제용으로 쓴다.

### 주요 환경변수

| 변수 | 분류 | 설명 |
|---|---|---|
| `ANTHROPIC_API_KEY` | 인증 | API 키. 헤드리스/CI에서 필수. 설정되면 구독 로그인보다 우선 |
| `ANTHROPIC_AUTH_TOKEN` | 인증 | 커스텀 Bearer 토큰(게이트웨이 등). API 키보다 우선 |
| `ANTHROPIC_BASE_URL` | 배포 | API 트래픽을 LLM 게이트웨이·프록시로 리다이렉트 |
| `CLAUDE_CODE_USE_BEDROCK` | 배포 | `1`이면 Amazon Bedrock 경유 |
| `CLAUDE_CODE_USE_VERTEX` | 배포 | Vertex AI 경유(`ANTHROPIC_VERTEX_PROJECT_ID`, `CLOUD_ML_REGION` 필요) |
| `ANTHROPIC_BEDROCK_BASE_URL` | 배포 | Bedrock 엔드포인트 재지정(게이트웨이 경유용) |
| `HTTPS_PROXY` / `HTTP_PROXY` | 네트워크 | 사내 프록시 URL |
| `NO_PROXY` | 네트워크 | 프록시를 우회할 호스트 목록 (`*`이면 전체 우회) |
| `NODE_EXTRA_CA_CERTS` | 네트워크 | 커스텀 CA 인증서 경로(TLS 검사 환경) |
| `CLAUDE_CODE_CERT_STORE` | 네트워크 | 신뢰할 인증서 저장소(`bundled,system` 기본) |
| `CLAUDE_CODE_CLIENT_CERT` | 네트워크 | mTLS 클라이언트 인증서(`CLAUDE_CODE_CLIENT_KEY`와 함께 사용) |
| `DISABLE_TELEMETRY` | 거버넌스 | 텔레메트리 비활성화 |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` | 거버넌스 | 비필수 아웃바운드 트래픽 차단(에어갭 환경) |
| `DISABLE_AUTOUPDATER` | 거버넌스 | 자동 업데이트 끄기(버전 고정 정책) |
| `CLAUDE_CODE_MAX_TURNS` | 동작 | 최대 에이전트 턴 수. 파일럿 비용 통제용 |

전체 목록: code.claude.com/docs/en/env-vars

### 단축키 4가지 범주 (가르치는 순서가 중요)

1. **Bash 모드 `!`** – 셸 명령을 바로 실행하고, 그 출력이 Claude의 컨텍스트에 들어간다. **첫날 가장 먼저 가르칠 것.** 개발자가 이 도구를 "내 터미널의 연장"으로 보게 만드는 기능이다.
   ```
   ! npm test
   Fix the failing tests in the output above
   ```
2. **컨텍스트 관리** `/clear`, `/compact`, `/context` – **2주차에 가르칠 것** (개발자가 컨텍스트 한계에 부딪힌 뒤).
   - `/clear`: 대화 초기화. **파일 변경은 되돌리지 않는다** (되돌리려면 git 사용)
   - `/compact`: 흐름을 유지하면서 컨텍스트 압축
   - `/context`: 컨텍스트 사용량 시각화
3. **내비게이션**
   - `Esc`: 실행 중 취소
   - `Esc Esc`: 입력이 비어 있으면 되감기 메뉴(대화 분기 / 코드 되감기 / 둘 다), 입력이 있으면 초안 삭제
   - `Shift+Tab`: 실행 모드 순환 – default → auto-accept edits → **plan**(변경 전 승인 대기). 코드 리뷰 규정이 엄격한 엔터프라이즈 고객에게 중요하다.
4. **파일 컨텍스트·세션**
   - `@`: 파일·폴더를 컨텍스트에 추가
   - `--continue`(`-c`): 최근 대화 재개, `--resume <id>`: 특정 세션 재개 (세션 안 명령이 아니라 실행 시 CLI 플래그)
   - `/model`: 세션 중 모델 변경

### 셸 파이핑 — CI 자동화로 가는 다리

Claude Code는 셸 입장에서 일반 Unix 명령과 같다. 어떤 출력이든 파이프로 넘길 수 있다.

```bash
cat error.log | claude
gh pr diff "$1" | claude -p --append-system-prompt "Review for security vulnerabilities"
git log --oneline -20 | claude -p "Summarise these commits for a release note"
```

> **핵심:** 정착하는 패턴은 개발자의 기존 습관에 맞는 것이다. Bash 모드와 파이핑은 이미 있는 습관에 들어맞고, 컨텍스트 관리는 2주차에 생기는 문제를 해결한다. 이 순서대로 교육할 것.

---

## Lesson 4. 개발 컨테이너와 샌드박스 실행

> **"샌드박스돼 있어요"는 완전한 보안 답변이 아니다. 어떤 격리 계층이 적용되는지를 말해야 한다.**

### 격리 계층별 적용 범위

| 접근 대상 | 내장 Bash 샌드박스 | 샌드박스 런타임 | 컨테이너 / VM |
|---|---|---|---|
| Bash 명령 | ✓ 제한 | ✓ 제한 | ✓ 제한 |
| 파일 도구(Read/Write) | ✗ 전체 접근 | ✓ 제한 | ✓ 제한 |
| MCP 서버 | ✗ 전체 접근 | ✗ 전체 접근 | ✓ 제한 |
| Hooks | ✗ 전체 접근 | ✗ 전체 접근 | ✓ 제한 |
| 웹 접근 | ✗ 전체 접근 | ✓ 제한 | ✓ 제한 |

→ 내장 Bash 샌드박스는 **Bash 하위 프로세스만** 제한한다. 파일 도구, MCP, hooks, 웹까지 막으려면 더 넓은 격리 계층이 필요하다.

### 샌드박스 모델의 4가지 속성

1. **디렉터리 범위 제한** – 지정한 디렉터리 밖은 읽기·쓰기 불가 (홈 디렉터리, 다른 코드베이스, 시스템 파일 등)
2. **네트워크 허용 목록** – 허용된 호스트로만 아웃바운드 요청 가능. 규제 산업의 데이터 유출 통제에 중요
3. **범위 안에서는 승인 없이 동작** – 매번 승인하지 않아도 되므로 생산성은 유지되고, 자율 실행 범위는 명확히 제한된다
4. **범위 밖 시도는 알림** – 조용히 실패하거나 권한을 올리지 않고, 시도한 내용을 사용자에게 보여준다

### settings.json 설정 예시

`sandbox`(프로세스·파일시스템), `permissions`(도구 허용/거부), `network`(도메인 허용 목록) 세 그룹으로 구성된다.

```json
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "denyRead": ["/"],
      "allowRead": ["/project", "~/.claude"],
      "allowWrite": ["/project"]
    },
    "network": {
      "allowedDomains": ["api.anthropic.com", "*.client-internal.com"]
    }
  },
  "permissions": {
    "allow": ["Bash(npm run test *)", "Bash(npm run lint)", "Read(/project/**)"],
    "deny": ["Bash(curl *)", "Read(./.env)", "Read(~/.ssh/**)"],
    "defaultMode": "acceptEdits"
  }
}
```

- `denyRead` / `allowRead`: 루트를 막고 프로젝트만 다시 허용 → "SSH 키는 *아마* 안 읽을 거예요"가 아니라 "*못* 읽습니다"라고 말할 수 있다.
- `allowedDomains`: 와일드카드 지원. 방화벽에만 의존하지 않고 egress 정책을 강제한다.
- `permissions.deny`: 관리형 설정으로 배포하면 로컬 설정으로 덮어쓸 수 없다.
- `defaultMode`: `default`, `acceptEdits`, `plan`, `auto` 중 선택. `acceptEdits`는 편집은 자동 승인하고 bash는 확인을 받는다.

### 개발 컨테이너

컨테이너가 바깥쪽 경계를, Claude Code 샌드박스가 안쪽 경계를 담당하는 **이중 격리** 구조다. 공식 devcontainer feature를 제공한다.

```json
{
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/anthropics/devcontainer-features/claude-code:1.0": {}
  }
}
```

feature 버전은 설치 스크립트를 고정할 뿐 Claude Code 버전을 고정하지 않는다. 컨테이너 안에서도 기본적으로 자동 업데이트되므로, 버전 고정이 필요하면 `DISABLE_AUTOUPDATER`를 설정할 것.

### 시나리오

보안 책임자: "개발자가 실수로 SSH 키 같은 프로젝트 밖 파일을 읽게 설정할 수 있나요?"
→ **"민감 경로에 대한 명시적 deny 규칙을 설정하고 배포 전에 검증합니다."** 보안팀에는 "기본적으로 안 됩니다" 같은 수동적인 답이 아니라 **능동적 설정 + 검증**으로 답해야 한다. 단, Bash 샌드박스는 bash 명령만 막으므로 파일 도구로 SSH 키에 접근하는 것을 막으려면 `permissions.deny`나 컨테이너/VM이 필요하다.

### Configuration Pack 기여 (설정 섹션)

- 환경(dev/staging/prod)별 샌드박스 설정: 파일시스템 범위, 허용 도메인, 거부 도구
- 인증 방식(OAuth / API 키 / SAML·SCIM SSO)과 프록시 설정
- 적용 격리 계층(Bash 샌드박스 / 샌드박스 런타임 / 컨테이너·VM)

---

## Lesson 5. 헤드리스·비대화형 모드 (스크립트·CI)

> **헤드리스는 별도 제품이 아니라 플래그 하나다.** 같은 바이너리, 같은 인증, 같은 모델을 사람 없이 실행할 뿐이다.

### 핵심 플래그 5가지

| 플래그 | 역할 |
|---|---|
| `-p` | 비대화형 실행. 프롬프트를 받아 실행하고 종료한다. 파이프라인에 필수 |
| `--allowedTools` | 사용 가능한 도구 제한 (예: `"Bash,Read,Edit"`) |
| `--append-system-prompt` | 이번 실행에 적용할 시스템 지시 추가 ("보안 리뷰", "JSON 출력" 등) |
| `--output-format json` | JSON으로 출력. 스키마는 프롬프트에 직접 명시하고, 운영 환경에서는 검증·재시도 로직을 둘 것 |
| `--bare` | 로컬 hooks, skills, plugins, MCP, auto memory, CLAUDE.md 로딩 비활성화 → 환경과 무관하게 재현 가능한 CI 실행 |

### 실전 패턴 3가지

```bash
# 1. PR 자동 코드 리뷰
gh pr diff "$1" | claude -p \
  --append-system-prompt "Review for security vulnerabilities" \
  --output-format json

# 2. 테스트 실행 후 실패 수정
claude -p "Run test suite, fix any failures" --allowedTools "Bash,Read,Edit"

# 3. 구조화된 추출 (API 엔드포인트 목록)
claude -p 'List all API endpoints in this codebase. Output a JSON array where each item has "path" (string) and "method" (string).' \
  --output-format json
```

### GitHub Actions 연동

- **기본 방식:** 세션에서 `/install-github-app` 실행 → 조직에 GitHub 앱 설치 → 이슈·PR 댓글에서 `@claude`를 태그하면 동작한다.
- **커스텀 워크플로우 YAML의 4가지 구성 요소:**
  1. 트리거 – PR마다, 스케줄, 커스텀 이벤트
  2. 환경 – GitHub Secrets에서 `ANTHROPIC_API_KEY` 주입
  3. Claude 명령 – `-p`, `--output-format json`, `--allowedTools`
  4. 출력 처리 – PR 댓글, 파일 저장, 다른 액션으로 전달

### CI 인증

CI 인증은 Claude Code 문제가 아니라 **시크릿 관리 문제**다. `${{ secrets.ANTHROPIC_API_KEY }}`처럼 참조하며, GitLab CI 변수나 Jenkins 자격증명 저장소에서도 같은 패턴을 쓴다.

### 시나리오

"모든 PR을 자동 리뷰해서 보안 대시보드용 JSON 리포트를 받고 싶다" → **`-p` + `--output-format json` + 프롬프트에 스키마 명시.** 스키마를 지정하지 않으면 JSON 구조가 정해지지 않아 대시보드가 파싱할 수 없다.

### 실습: 3개 환경 설치·실행

1. 로컬 CLI – `claude --version`
2. IDE 연동 – 확장 설치 후 간단한 질문에 응답하는지 확인
3. 개발 컨테이너 – devcontainer 구성 후 컨테이너 안에서 `claude --version`
4. 헤드리스 스크립트를 CLI와 컨테이너 양쪽에서 실행해 둘 다 출력이 나오면 성공

### Configuration Pack 기여 (컨테이너 스펙 섹션)

- devcontainer feature 참조와 버전 고정 설정(자동 업데이트 비활성화 포함)
- 헤드리스 실행 패턴: 비대화형 플래그, 출력 형식, 입력 소스(git log, 테스트 출력 등)
- CI 연동 경로: GitHub Actions 트리거, 워크플로우 파일 위치, `@claude` 사용법

→ 설정 섹션과 컨테이너 스펙 섹션을 IT·DevOps 리드에게 전달하면, 롤아웃 전에 해결할 요건이 모두 정리된다.

---

## 전체 정리

- **사전 점검:** OS 지원, 네트워크 엔드포인트, 인증 방식은 킥오프 전에 IT와 확인한다. 네트워크가 1순위다.
- **설치·검증:** 한 줄로 설치한 뒤 `claude --version`, `claude doctor`, `/status`로 확인한다.
- **IDE:** VS Code·Cursor는 확장 하나, JetBrains는 CLI와 플러그인 둘 다 필요하다. 첫날에는 diff 보기, 선택 영역 컨텍스트, 파일 참조를 시연한다.
- **터미널:** 첫날은 `!` bash 모드와 파이핑, 2주차는 컨텍스트 관리. 코드 리뷰 규정이 엄격한 곳에는 plan 모드를 보여준다.
- **보안:** 격리 계층을 구체적으로 명시하고, deny 규칙을 능동적으로 설정해 검증한다. 컨테이너는 이중 경계를 제공한다.
- **CI:** `-p`가 핵심이고 `--output-format json`, `--allowedTools`, `--bare`를 조합한다. CI 인증은 시크릿 관리로 해결한다.

다음 코스: **Deployment Architecture** (배포 경로, 네트워크 토폴로지, 파일럿에서 엔터프라이즈 롤아웃으로 가는 설계 결정)
