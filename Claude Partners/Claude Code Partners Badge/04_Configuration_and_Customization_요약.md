# Configuration and Customization 요약 (Claude Partner Badge: Claude Code)

총 5개 레슨: ① 설정 계층과 우선순위 ② CLAUDE.md ③ settings.json vs settings.local.json ④ 커스텀 슬래시 명령 ⑤ 출력 스타일

이 코스의 산출물: **CLAUDE.md + Slash Command Catalogue** (스코프 결정 맵, CLAUDE.md, 권한 규칙이 담긴 settings.json, 슬래시 명령, 출력 스타일)

> ※ 시나리오 문제는 시뮬레이션이라 정답이 화면에 따로 표시되지 않습니다. 아래 "권장 답"은 강의 본문 내용을 기준으로 정리한 것입니다.

---

## Lesson 1. 설정 계층과 우선순위

> **스코프는 기술 문제이기 전에 소유권 문제다.** 누가 이 설정을 통제하는가? 누구에게 적용되어야 하는가? 덮어쓸 수 있어야 하는가? 이 세 가지에 답하면 스코프는 저절로 정해진다.

### 4가지 스코프

| 스코프 | 위치 | 대상·용도 |
|---|---|---|
| **Managed** | macOS: `/Library/Application Support/ClaudeCode/`<br>Linux: `/etc/claude-code/` | IT가 MDM, 그룹 정책, 시스템 설정 파일로 배포. 머신의 모든 세션에 적용되며 어떤 사용자·프로젝트 설정으로도 덮어쓸 수 없다 |
| **Local** | `.claude/settings.local.json` | 특정 프로젝트에서 나에게만 적용되는 개인 설정. Claude Code가 생성하면 자동으로 git에서 제외된다 |
| **Project** | `.claude/settings.json` | git에 커밋되어 팀 전체가 공유. 허용/거부 도구, hooks, 승인된 MCP 서버 |
| **User** | `~/.claude/settings.json` | 모든 프로젝트에 적용되는 개인 기본값(선호 모델, 키 바인딩, 알림 등). 우선순위 최하 |

### 우선순위 (위에서부터 먼저 찾은 값을 사용)

1. **Managed** – 최고 권한. 고객 컴플라이언스 요건이 여기에 들어간다
2. **CLI 플래그** – `--allowedTools`, `--model` 등. 해당 세션에서만 모든 파일 설정보다 우선하며 저장되지 않는다
3. **Local** – 공유 파일을 바꾸지 않고 내 세션에서만 프로젝트 설정을 덮어쓴다
4. **Project** – 팀 기준선
5. **User** – 위에서 아무것도 정하지 않았을 때 쓰는 최후의 기본값

### 예외: 권한은 '병합'된다

allow/deny 목록은 덮어쓰지 않고 **스코프 간에 합쳐진다.** 로컬 allow 규칙은 프로젝트 목록에 추가된다. 개발자는 로컬에서 자기 권한을 넓힐 수 있지만, **프로젝트나 managed 스코프의 deny 규칙은 제거할 수 없다.**

### 스코프 선택의 핵심 기준

- **Project:** 저장소와 함께 이동해야 하는가? → 팀 전체가 필요한 설정(사전 승인 명령, 거부 도구, MCP, hooks)
- **Local:** 내 머신에만 있어야 하는가? → 샌드박스 URL, 테스트 자격증명, 아직 공유하기 이른 실험 설정

### 시나리오: Meridian Bank

| 상황 | 권장 답 |
|---|---|
| 1. 보안팀이 모든 개발자 PC에서 curl을 막고 싶어 함. 재설치해도 유지되어야 함 | **Managed 스코프** (MDM으로 배포, 재설치 후에도 유지) |
| 2. 한 개발자가 동료에게 영향 없이 자기 워크플로우용 모델을 바꾸고 싶어 함 | **User 스코프** (선호 모델은 User 설정의 대표 용도. 선택지 A는 "Local"이라면서 `~/.claude/` 경로를 제시해 스코프와 경로가 맞지 않음) |
| 3. 팀 전체에 `docker run`을 사전 승인 | **Project `.claude/settings.json`** |

---

## Lesson 2. CLAUDE.md: 전사 공통 vs 저장소별 메모리

> **CLAUDE.md는 프로젝트의 영구 메모리다.** 세션마다 같은 설명을 반복하지 않게 해준다.

### 메모리 계층 (덮어쓰지 않고 이어 붙임)

CLAUDE.md 파일들은 서로 덮어쓰지 않는다. Claude가 **넓은 범위에서 좁은 범위 순으로 전부 읽어서 이어 붙인다.**

| 스코프 | 위치 | 내용 |
|---|---|---|
| Managed Policy | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md` | IT가 배포. 항상 로드되며 개발자가 제외할 수 없다 |
| User | `~/.claude/CLAUDE.md` | 모든 프로젝트에 적용되는 개인 선호(코딩 스타일, 오류 처리 방식 등) |
| Project | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md` | 아키텍처 결정, 코딩 표준, 빌드 명령, 워크플로우. git으로 공유. **가장 중요한 파일** |
| Local | `./CLAUDE.local.md` | 샌드박스 URL, 개인 테스트 데이터. `.gitignore`에 추가할 것 |

- **하위 디렉터리의 CLAUDE.md**는 세션 시작 시가 아니라, Claude가 그 디렉터리의 파일을 읽을 때 **필요한 시점에 로드**된다. 모노레포에서 `frontend/`, `backend/`, `test-cases/`에 각각 두면 초기 컨텍스트를 가볍게 유지할 수 있다.

### 무엇을 넣을까

- **기준:** 지난 세션에 채팅으로 쳤던 내용이라면 CLAUDE.md에 넣는다.
- **추가할 때:** Claude가 같은 실수를 두 번 할 때, 코드 리뷰에서 Claude가 알았어야 할 내용이 지적될 때, 지난번과 같은 수정을 또 입력할 때, 신규 팀원에게도 필요한 맥락일 때.
- **넣을 것:** 매 세션 필요한 사실(빌드 명령, 네이밍 규칙, 아키텍처 결정, "항상 X 하라" 규칙).
- **다른 곳으로 옮길 것:** 여러 단계의 절차나 코드베이스 일부에만 해당하는 내용 → skill이나 경로 범위 규칙으로.
- **가지치기 원칙:** 지시가 없어도 Claude가 잘한다면 그 지시는 삭제한다. 파일이 비대해지면 지시를 따르는 정도가 떨어진다.

### 효과적인 작성법

| 모호함 (효과 없음) | 구체적 (효과 있음) |
|---|---|
| "코드를 제대로 포맷해라" | "2칸 들여쓰기를 사용하라. 커밋 전에 `npm test`를 실행하라" |

- 파일당 **200줄 이하**를 목표로 하고, 마크다운 헤더로 묶는다.
- 문서 내용을 복사하지 말고 **`@path/to/file`로 참조**한다(참조된 파일은 세션 시작 시 함께 로드).
- 새 프로젝트에서는 **`/init`**으로 CLAUDE.md 초안을 자동 생성한 뒤 다듬는다.

### 대규모 저장소 패턴

- 루트: 시스템 개요, 팀 구조, 현대화 전략 → 매 세션 로드
- 서비스별: 백엔드(API 설계, 데이터 모델), 프론트엔드(UI 패턴, 컴포넌트 라이브러리) → 필요 시 로드
- 테스트 디렉터리: 커버리지 요건, 자동화 전략
- 다른 팀의 CLAUDE.md가 잡음이 되면 로컬 설정의 **`claudeMdExcludes`**로 제외한다(managed policy를 제외한 모든 스코프에서 가능).

### O/X 퀴즈

- "CLAUDE.md 지시는 강제된다" → **X.** 강제되는 설정이 아니라 컨텍스트로 로드된다. 반드시 지켜야 하는 동작은 **hook**(정해진 시점에 실행되는 셸 명령)이나 **managed settings**로 처리한다.
- "Project CLAUDE.md에 개인 샌드박스 URL을 넣어도 된다" → **X.** `CLAUDE.local.md`를 쓰고 `.gitignore`에 추가한다.
- "CLAUDE.md가 이미 있으면 `/init`이 덮어쓴다" → **X.** 덮어쓰지 않고 개선 사항을 제안한다. 언제 실행해도 안전하다.
- "`claudeMdExcludes`로 조직 managed CLAUDE.md를 제외할 수 있다" → **X.** managed policy 파일은 제외할 수 없다(의도된 설계).

> **핵심:** CLAUDE.md는 행동을 *형성*하는 것이지 *제약*하는 계층이 아니다.

### 시나리오: Whitmore Group

| 상황 | 권장 답 |
|---|---|
| 1. 60페이지 분량의 아키텍처 결정 로그를 Claude가 알게 하고 싶음 | **3문단으로 요약하고 `@docs/architecture.md`로 원문 참조** |
| 2. 6개월 전에 버린 ORM 패턴을 Claude가 계속 제안함 | **CLAUDE.md에 '하지 말 것' 항목으로 명시** |
| 3. CLAUDE.md가 길어져서 개인 선호를 분리하고 싶음 | **User 레벨 `~/.claude/CLAUDE.md`로 이동** |

### Catalogue 기여: 저장소 CLAUDE.md

1. `## Build & Test` – 빌드·테스트 명령을 정확히
2. `## Architecture` – 구조와 명확하지 않은 아키텍처 결정을 2~3문장으로, 기존 문서는 `@`로 참조
3. `## Code Style` – 검증 가능한 구체적 규칙 2~3개

---

## Lesson 3. settings.json vs settings.local.json 규칙

> **두 파일. 하나는 공유, 하나는 내 것.** 포맷은 같고 목적은 정반대다.

| | `.claude/settings.json` (공유) | `.claude/settings.local.json` (개인) |
|---|---|---|
| git | 커밋됨, 클론하면 자동 적용 | 자동으로 제외됨, 커밋 안 됨 |
| 넣을 것 | 권한 allow/deny, hook 정의, 승인된 MCP 서버, 회사 공지 | 개인 권한 모드 선호, 실험 중인 설정, 머신별 도구 경로, 다른 팀 CLAUDE.md 제외 |

### 공유 설정 예시

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": ["Bash(npm run lint)", "Bash(npm run test *)", "Bash(git status)", "Bash(git log *)"],
    "deny": ["Bash(curl *)", "Read(./.env)", "Read(./.env.*)", "Read(./secrets/**)"]
  },
  "companyAnnouncements": ["Review our coding guidelines at docs.acme.com/claude"]
}
```

공유 파일에 **넣지 않는 것:** 모델 선호(User 설정), 자격증명이 담긴 환경변수(커밋 파일에 절대 금지), `defaultMode` 변경(개발자 로컬 파일).
`$schema` 줄을 넣으면 VS Code·Cursor에서 자동완성과 검증이 되므로 모든 프로젝트 설정에 넣을 것을 권장한다.

### 권한 평가 순서

1. **Deny** – 하나라도 맞으면 allow와 무관하게 차단 (예: `Read(./.env)`)
2. **Ask** – 확인을 요청 (예: `Bash(git push *)`)
3. **Allow** – 확인 없이 진행 (예: `Bash(npm run test *)`)
4. **기본값: Ask** – 어떤 규칙에도 해당하지 않으면 확인을 요청한다(안전한 기본값)

### 시나리오

- **개발자 15명 팀:** `npm run test/lint` 사전 승인과 curl 차단은 모두에게, 개인 실험용 allow는 각자 → **팀 규칙은 `.claude/settings.json`, 개인 실험은 `.claude/settings.local.json`.** User 설정에 넣으면 15대에 수동으로 동기화해야 하고 신규 팀원에게도 자동 적용되지 않는다.
- **Apex Logistics:**

| 상황 | 권장 답 |
|---|---|
| 1. 플랫폼 저장소만이 아니라 모든 엔지니어 PC에서 `WebFetch(domain:internal-ledger.apex.com)` 차단 | **IT가 배포하는 managed policy** |
| 2. 새 내부 MCP 서버를 팀에 추천하기 전에 로컬에서 테스트 | **`.claude/settings.local.json`** (이 프로젝트에만, git 제외) |
| 3. 팀 전체에 `Bash(docker compose up *)` 사전 승인 | **`.claude/settings.json`** |

> **핵심:** 공유 기준선은 커밋하고 개인은 로컬에서 확장한다. 이 구분을 틀리면 팀 전체에 설정 불일치가 생긴다.

---

## Lesson 4. 커스텀 슬래시 명령

> **같은 걸 두 번 넘게 프롬프트로 쳤다면, 명령으로 만들어라.**

### 구조: YAML frontmatter가 있는 마크다운 파일

| 항목 | 설명 |
|---|---|
| 위치 | `.claude/commands/` (프로젝트, git 커밋, 팀 공유) |
| 파일명 | 명령 이름이 된다. 폴더로 네임스페이스 생성: `review-pr.md` → `/review-pr`, `security/audit.md` → `/security:audit` |
| Frontmatter | 선택 사항. `allowed-tools`(사용 가능한 도구), `description`(명령 선택기에 표시될 설명) |
| 인자 | 본문 어디서든 `$ARGUMENTS`로 호출 시 넘긴 텍스트를 받는다 (예: `/review-pr 1234`) |

### 예시: `.claude/commands/review-pr.md`

```markdown
---
description: Review a pull request against the security checklist
allowed-tools: Read, Bash(git *)
---
## PR Review: $ARGUMENTS
You are reviewing PR $ARGUMENTS against Acme Corp's security checklist.
Run `git diff main` to inspect the changed files, then evaluate:
- No credentials, tokens, or API keys in changed files
- No new eval() or dynamic code execution patterns
- No new external network calls without logging
- Dependency changes have corresponding security review notes
Summarise your findings as: approved / needs changes / blocked.
For each finding, cite the specific file and line number.
```

`/review-pr 847`로 호출하면 `$ARGUMENTS`가 847로 바뀌고, 도구는 Read와 `Bash(git *)`로 제한된 상태에서 체크리스트 검토가 실행된다.

> **설계 원칙:** 스프린트 중 가장 힘든 날, 압박 속에서 이것저것 오가는 사람을 위해 작성하라. 생각 없이 실행할 수 있을 만큼 구체적이고, 30초 안에 읽을 수 있을 만큼 짧게.

### 팀 명령 vs 개인 명령

- **팀:** `.claude/commands/` → 저장소에 넣어 고객에게 전달하는 산출물
- **개인:** `~/.claude/commands/` → 그 머신의 모든 프로젝트에서 사용
- 둘 다 등록이나 재시작 없이 자동으로 인식된다.

### 시나리오

- **`/release-summary` 설계:** 짧은 프롬프트 vs `$ARGUMENTS`(버전 태그) + 허용 도구 + 출력 형식 + 대상 독자(비기술 이해관계자)
  → **후자가 정답.** 짧은 버전의 "유연성"은 착각이다. 생각할 부담을 호출 시점으로 미룰 뿐이라 재사용 자산을 만드는 의미가 없다.
- **Vantage Capital:**

| 상황 | 권장 답 |
|---|---|
| 1. 팀이 매일 스탠드업 전에 `git log --oneline -20` 실행 | **`.claude/commands/`에 넣어 git 커밋** |
| 2. "Explain this function." 초안 개선 | **`$ARGUMENTS`(함수 이름) + `allowed-tools: Read` + 대상 독자(이 모듈을 모르는 퀀트 분석가) 명시** |
| 3. 한 개발자가 자기 노트 정리용 명령을 원함 | **`~/.claude/commands/`** (개인 명령) |

### Catalogue 기여

1. 샘플 저장소의 `.claude/commands/`에 명령 작성
2. 세션에서 `/`를 입력해 선택기에 나타나는지 확인
3. 명령 카탈로그에 이름, 기능, 유용한 역할을 한 줄로 기록 → 엔게이지먼트 종료 시 고객에게 전달

> 커스텀 명령은 조직의 노하우를 도구 체인 자체에 담는다. 가장 좋은 사용법이 처음 알아낸 사람에게만 머물지 않게 해준다.

---

## Lesson 5. 출력 스타일

> **CLAUDE.md는 Claude가 *무엇을 아는지*, 출력 스타일은 Claude가 *어떻게 소통하는지*를 정한다.**

- **CLAUDE.md:** 프로젝트 메모리(아키텍처, 규칙, 빌드 명령). 세션 시작 시 시스템 프롬프트에 주입된다.
- **출력 스타일:** 시스템 프롬프트를 수정해 지식이 아닌 **어조와 형식**을 바꾼다(설명의 양, 바로 실행할지 지시를 기다릴지, TODO를 남길지 등).
- ⚠️ **세션 시작 시 적용**된다. 세션 중에 바꾸면 효과가 없고, `/clear` 후나 새 세션부터 적용된다.

### 설정 방법

- 세션에서 `/config` (→ `settings.local.json`에 저장)
- 또는 원하는 스코프의 설정 파일에 `"outputStyle": "Explanatory"`

### 내장 스타일 4가지

| 스타일 | 특징 | 적합한 시점 |
|---|---|---|
| **Default** | 효율적이고 정확, 부가 설명 없음 | 대부분의 경우. 도입 초기를 지난 프로덕션 팀 |
| **Explanatory** | 구현 선택과 코드베이스 패턴을 설명하는 "Insights" 섹션 추가 | 도입 단계, 엔지니어가 Claude의 판단을 이해해야 할 때 |
| **Proactive** | 묻지 않고 합리적으로 가정해 바로 실행. 중요한 작업에는 여전히 권한 확인 | 더 빠르고 자율적인 실행이 필요할 때 |
| **Learning** | 협업 모드. 추론을 설명하고 핵심 결정 지점에 `TODO(human)` 마커를 남겨 개발자가 직접 완성 | 개발자가 결과를 통째로 받기보다 핵심을 직접 작성해야 할 때 |

### 커스텀 스타일

`.claude/output-styles/`(프로젝트, 사용자, managed 레벨)에 YAML frontmatter가 있는 마크다운 파일로 만든다.

| 옵션 | 동작 | 용도 |
|---|---|---|
| `keep-coding-instructions: true` | Claude의 기본 엔지니어링 지침(변경 범위, 검증, 주석)을 **유지**하고 소통 형식만 추가 | "항상 다이어그램부터", "고객용 산출물 형식으로" 등 |
| `keep-coding-instructions: false` (**기본값**) | 엔지니어링 지침을 **제거**하고 페르소나를 완전히 교체 | 작문 보조, 데이터 분석, 요구사항 정리 등 소프트웨어 개발이 아닌 작업. 플래그를 생략하면 엔지니어링 맥락이 사라지니 주의 |

GSI 엔게이지먼트에서 자주 쓰는 패턴 3가지 (모두 `true`):

- **엔게이지먼트 문서 스타일:** 모든 출력을 헤더, 권고, 다음 단계가 있는 고객용 산출물로
- **보안 리뷰 스타일:** 모든 발견 사항을 심각도, 파일 경로, 줄 번호, 권장 수정과 함께
- **온보딩 모드:** 중간에 합류한 주니어 개발자를 위한 설명 추가

### 시나리오: Hartfield Engineering (3일 활성화)

| 상황 | 권장 답 |
|---|---|
| 1. Day 1: 엔지니어들이 Claude의 아키텍처 선택 이유를 계속 물어 세션이 느려짐 | **Explanatory** (각 결정을 자동으로 설명) |
| 2. Day 3: 팀이 Claude를 신뢰하게 되었고 설명이 오히려 부담이 됨 | **Default로 전환** (설명 없이 최대 속도) |
| 3. 종료 시: 팀 전체가 내부 문서 형식(헤더, 권고, 다음 단계)으로 출력받기를 원함 | **`keep-coding-instructions: true`인 커스텀 출력 스타일을 `.claude/output-styles/`에 추가** |

### Catalogue 기여 (최종)

1. 엔게이지먼트에 맞는 스타일 결정: 도입 단계면 Explanatory, 이미 신뢰가 쌓인 생산성 중심 팀이면 Default
2. 커스텀 형식이 필요하면 `.claude/output-styles/`에 스타일 파일 추가 (완전히 비엔지니어링 용도가 아니면 `keep-coding-instructions: true`)
3. 전체 세트 검토: 스코프 결정 맵, CLAUDE.md, 권한 규칙이 담긴 settings.json, 슬래시 명령, 출력 스타일 → 엔게이지먼트 종료 시 고객에게 전달

---

## 전체 정리

- **스코프:** Managed > CLI 플래그 > Local > Project > User. 단, 권한 allow/deny는 병합되며 상위 deny는 제거할 수 없다. 보안 요건은 Managed, 팀 기준선은 Project.
- **CLAUDE.md:** 덮어쓰지 않고 이어 붙는 메모리 계층. 구체적이고 검증 가능하게 200줄 이하로 쓰고, `@`로 참조하며, 필요 없어진 지시는 지운다. 강제가 필요하면 hook이나 managed settings.
- **settings 파일:** 공유는 `settings.json`(커밋), 개인은 `settings.local.json`(git 제외). 권한은 Deny → Ask → Allow → 기본 Ask 순서로 평가된다.
- **슬래시 명령:** `.claude/commands/*.md`, frontmatter, `$ARGUMENTS`. 팀 명령은 저장소에, 개인 명령은 `~/.claude/commands/`에 둔다.
- **출력 스타일:** 도입기에는 Explanatory, 숙련 후에는 Default. 커스텀 스타일은 보통 `keep-coding-instructions: true`.

다음 코스: **Extensibility** (MCP 서버, 커스텀 도구, 통합)
