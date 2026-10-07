# Security and Governance 요약 (Claude Partner Badge: Claude Code)

총 6개 레슨: ① managed-settings.json ② 권한 시스템과 작업 게이팅 ③ 샌드박스 bash와 파일시스템 범위 ④ SSO·SCIM·IP 허용 목록 ⑤ ZDR·보존 정책·데이터 사용 ⑥ SOC 2·HIPAA와 고객 통제

이 코스의 산출물: **보안 질의서**(FinCo 사례 기준)와 **OTel 설정 + 대시보드 스펙**

### 코스 전체에 쓰이는 고객 사례: FinCo Financial Services

- 개발자 1,800명, 핵심 뱅킹 인프라 현대화 중
- AWS 네이티브(us-east-1, us-west-2), ID 관리는 Okta, macOS 관리는 Jamf
- 보안팀의 3대 필수 조건:
  1. 모든 데이터 처리는 승인된 AWS 리전 안에서
  2. 모든 개발자 계정은 Okta로 추적 가능해야 함
  3. 새 AI 도구는 서면 보안 검토를 통과해야 함

---

## Lesson 1. managed-settings.json: 강제 로그인, MCP 허용·거부, hooks 제한

> **사용자가 덮어쓸 수 없는 정책 파일.** 다른 설정 계층은 개인을 위한 것이고, managed-settings.json은 조직을 위한 것이다.

### 설정 계층 (위가 우선)

1. **Managed settings** – MDM이나 서버 푸시로 IT가 강제. 어떤 계층도 덮어쓸 수 없음
2. **커맨드라인 인자** – 세션 단위의 임시 설정(자동화 스크립트용)
3. **Local 프로젝트 설정** – `.claude/settings.local.json`
4. **공유 프로젝트 설정** – `.claude/settings.json`
5. **User 설정** – `~/.claude/settings.json`

### 배포 방법 3가지

| 방법 | 내용 | 적합한 고객 |
|---|---|---|
| **서버 관리형** | Claude.ai 관리 콘솔에서 푸시(Teams·Enterprise). Enterprise는 전체 기능, Teams는 기본 관리 콘솔 | MDM 인프라가 없는 곳. Jamf·Intune이 없을 때 가장 간단 |
| **MDM / OS 정책** | macOS는 Jamf로 managed preferences, Windows는 Intune·그룹 정책으로 레지스트리 | 이미 기기를 관리하는 곳. **FinCo는 Jamf가 있으므로 이 방식** |
| **파일 기반** | 시스템 디렉터리에 수동 설치 | 파일럿·소규모·단계적 테스트. 확장되지 않으므로 전체 배포 전에 MDM이나 서버 관리형으로 전환 계획 |

### Day 0 통제 (누구든 키보드를 만지기 전에)

| 분류 | 설정 | 역할 | FinCo 적용 |
|---|---|---|---|
| 인증 | `forceLoginMethod` | `"claudeai"` 또는 `"console"` 강제. SSO와 감사를 우회하는 API 키 로그인을 차단 | `"claudeai"`로 설정해 모든 로그인이 Okta SSO를 거치게 |
| 인증 | `forceLoginOrgUUID` | 지정한 조직 UUID 계정만 로그인 허용 | 회사 이메일로 만든 개인 Claude 계정 사용 차단 |
| MCP | `allowedMcpServers` | 승인된 MCP 서버 목록. **빈 배열이면 MCP 전면 차단** | 처음엔 좁게 시작. 보안 승인 전에는 빈 배열 |
| MCP | `allowManagedMcpServersOnly` | 관리자가 정의한 서버만 활성. 사용자 추가 서버는 차단 | "개발자가 직접 서버를 추가하는" 위험을 범주째 차단 |

### Day 0 이후 통제

| 통제 | 설정 | 의미 |
|---|---|---|
| Hooks | `allowManagedHooksOnly`, `disableAllHooks` | 승인된 자동화만 허용하고 개발자가 임의로 붙인 hook은 차단 |
| 권한 | `allowManagedPermissionRulesOnly`, `disableAutoMode` | 사용자가 승인 규칙을 바꾸거나 자동 승인을 켜지 못하게. 규제 환경에서 사람의 확인을 유지 |
| 버전 | `requiredMinimumVersion`, `requiredMaximumVersion` | 범위 밖 버전이면 **시작 시 종료.** 문서가 아니라 강제 수단 |
| 조직 CLAUDE.md | `claudeMd` 키 | 조직 지시를 모든 세션에 주입. 프로젝트·사용자 CLAUDE.md보다 **먼저 로드**되고 덮어쓸 수 없음 |

### 조직 CLAUDE.md 작성 4원칙

1. **선호가 아니라 정책:** 보안, 컴플라이언스, IP 처리만. 코딩 스타일은 프로젝트 CLAUDE.md로. 판단 기준: 위반하면 컴플라이언스·법적 문제가 생기는가?
2. **Claude로 초안 작성:** 고객의 사용 정책이나 데이터 처리 지침을 주고 초안을 받은 뒤 검토·정리
3. **과도한 제한 금지:** 컴플라이언스 3줄에 스타일 규칙 30줄을 섞으면 중요한 신호가 묻힌다. Claude는 모든 지시를 똑같이 지키려 해서 정작 중요한 것이 희석된다
4. **이유를 함께:** "규제 요건에 따라", "IP 정책에 따라" 같은 맥락을 붙이면 개발자가 우회하지 않고, 나중에 감사·갱신하기도 쉽다

### 시나리오

일부 개발자가 승인 목록 밖 저장소에 접근하는 개인 GitHub MCP 서버를 연결함. 설정 하나로 이 위험을 범주째 막으려면?
→ **`allowManagedMcpServersOnly: true`.** `deniedMcpServers`로 특정 서버만 막으면 다음에 추가될 다른 미승인 서버는 여전히 열려 있다.

---

## Lesson 2. 권한 시스템과 작업 게이팅

> **Claude Code는 모든 것에 허락을 구하지 않는다. 구해야 할 것에 구한다.**

### 분류 원칙

- **Allow:** 파괴적이지 않고 자주 하는 작업
- **Ask:** 되돌릴 수 없는 작업, 외부 네트워크 호출
- **Deny:** 실행될 이유가 전혀 없는 작업

| 작업 | 분류 | 이유 |
|---|---|---|
| 저장 후 테스트 실행 (`npm test`) | Allow | 비파괴적이고 빈번함. 막으면 위험은 줄지 않고 흐름만 끊긴다 |
| 소스 파일 읽기 (`cat src/app.js`) | Allow | 로컬 읽기라 영향 범위가 없다 |
| 공유 저장소에 push (`git push origin`) | Ask | 되돌릴 수 없고 공유 저장소로 간다. 의도적이고 추적 가능한 승인 필요 |
| 외부 API 첫 호출 (`POST api.jira`) | Ask | 로컬 환경을 벗어난다. 일상적이어도 첫 실행엔 확인 |
| 강제 삭제 (`rm -rf /`) | Deny | 치명적이고 되돌릴 수 없다. 승인 창을 꼼꼼히 읽길 기대하지 말고 선택지 자체를 없앤다 |

→ 분류는 설정 파일에 두고, managed-settings.json의 **`allowManagedPermissionRulesOnly: true`**로 잠근다. 잠그면 사용자는 분류를 바꿀 수 없다.

### 소유권 구분 (보안 검토에서 "누가 무엇을 책임지나"에 대한 답)

| Anthropic 소유 | 고객이 설정 |
|---|---|
| **모델 안전:** Constitutional AI, RLHF, 가중치 수준 거부, 사용 정책 집행. 어떤 고객도 끄거나 바꿀 수 없음 (약점이 아니라 장점) | **작업 게이팅:** allow/ask/deny, 역할 기반 접근, 커넥터 동의, 권한 정책 |
| **플랫폼 무결성:** 런타임 분류기, 출시 전 평가. 모든 테넌트에 동일 | **감사·거버넌스:** managed settings, SSO·SCIM, 보존 기간, Compliance API, 검토 워크플로우 → FinCo의 필수 조건과 직결 |

### 역할과 동의

| 역할 | 권한 | FinCo에서 누가 |
|---|---|---|
| **Primary Owner** | 조직 전체 통제, 결제, 모든 설정 (조직당 1명) | 배포 전체를 책임지는 고객 측 1인 |
| **Owner** | 멤버 관리, 커넥터 승인, managed-settings 설정 | 일상 거버넌스를 맡은 보안·플랫폼팀 |
| **Member** | Claude Code 사용, 관리자가 정한 범위 안에서 개인 설정. managed settings는 못 바꿈 | 개발자 1,800명 |
| **커넥터 동의** | OAuth 범위 MCP 연결은 사용자별 리소스 접근 전에 Owner 승인 필요 | 허용 목록(어떤 서버가 존재할 수 있나)과 별개의 두 번째 관문(그 사용자의 리소스에 접근해도 되나) |

"누가 이걸 바꿀 수 있나요?"라는 질문에는 이렇게 답한다. managed settings에 있으면 Owner만 바꿀 수 있고, 사용자 설정이면 사용자가 바꿀 수 있지만 Owner가 정한 범위 안에서만 가능하다.

### 커스텀 역할과 그룹

- **역할:** 할 수 있는 일을 정의한다. 기본 역할(강의 표기: Primary Owner, Owner, Admin, Member) 외에 커스텀 역할도 만들 수 있다. **역할은 누적된다.** 여러 그룹에 속하면 모든 역할이 합쳐지므로 의도적으로 배정해야 한다.
- **그룹:** 사용자 묶음이며 용도가 여럿이다. 접근 권한뿐 아니라 팀·부서별 지출 한도, 플러그인 배포 대상을 정한다. 수동으로 만들거나 IdP에서 SCIM으로 동기화한다.
- 좋은 질문은 "이 사람에게 어떤 역할이 필요한가?"가 아니라 **"이 기능은 어느 그룹 소유이고, 이 사람이 거기 속해야 하는가?"**다.
- **현재 방법:** 승인된 도구를 엔터프라이즈 플러그인으로 묶어 RBAC로 그룹에 배정한다.
- **로드맵:** 커넥터별 역할 통제는 아직 없다(먼저 커넥터 on/off, 이후 기능 범위 단위로 계획). 고객이 물으면 "예정"이라고 알리고 플러그인 + RBAC 방식을 안내한다.

### O/X

- Member는 자기 개인 `settings.json`을 설정할 수 있다 → **O**
- 관리자는 managed-settings로 사용자가 자기 allow/deny 규칙을 못 만들게 할 수 있다 → **O** (`allowManagedPermissionRulesOnly`)
- Claude Code는 bash 명령마다 항상 승인을 구한다 → **X.** allow로 분류된 작업은 조용히 실행된다
- Enterprise에서 그룹은 순수하게 접근 통제 용기다 → **X.** 지출 한도, 플러그인 배포에도 쓰인다

### 시나리오: FinCo 작업 분류

저장 후 테스트 자동 실행 / 사내 GitLab push / 내부 Jira API로 티켓 상태 업데이트
→ **테스트는 Allow, GitLab push는 Ask, Jira API는 Ask.** push를 Allow로 두면 되돌릴 수 없는 작업의 감사 공백이 생기고, 전부 Ask로 두면 경고 피로 때문에 읽지도 않고 승인하게 된다.

---

## Lesson 3. 샌드박스 bash 실행과 파일시스템 범위

> **샌드박스는 능력을 제한하는 것이 아니라, 자율 실행을 자신 있게 승인할 수 있게 해주는 것이다.**

### 샌드박스가 하는 일

1. **경계 정의:** 접근 가능한 디렉터리와 네트워크 호스트를 명시한다
2. **승인 프롬프트 감소:** 범위가 명확하면 그 안에서는 물어볼 게 없다. 좋은 범위 설정은 오히려 자율성을 높이고, 남는 프롬프트는 정말 중요한 것뿐이다
3. **실패 격리:** 범위 밖 호스트 시스템에 접근할 수 없으므로 최악의 경우가 경계 안으로 제한된다 → 보안 검토에서 "최악의 경우는?"에 대한 답

### 범위 설정 4가지 키 (managed-settings.json)

| 키 | 의미 | 예시 |
|---|---|---|
| `sandbox.filesystem.allowWrite` | 읽기·쓰기를 허용할 프로젝트 디렉터리 | 프로젝트 루트와 하위 |
| `sandbox.filesystem.denyWrite` / `denyRead` | 허용 경로 안에 있어도 금지. `denyWrite`는 수정 금지(동기화 폴더 등), `denyRead`는 읽기 금지(자격증명 등) | `~/Dropbox` → denyWrite, `~/.aws/credentials` → denyRead |
| `sandbox.network.allowedDomains` | 허용 도메인(와일드카드 지원, `*.finco.internal`) | `npm.finco.internal` |
| `sandbox.network.deniedDomains` | 금지 도메인. **allowedDomains보다 우선** | 개인 클라우드 저장소, 미승인 API, `registry.npmjs.org` |

※ 키 이름은 작성 시점 기준이므로 첫 배포 전에 최신 문서로 확인할 것.

- **egress 통제는 두 단계:** Anthropic이 모든 배포에 기본 통제(도메인 허용 목록, URL 정리, 유출 방어)를 적용하고, 그 위에 조직 관리자가 managed-settings.json으로 네트워크 경계를 설정한다. **개발자 개인 설정이 아니라 보안팀이 소유한다.**

### 시나리오

| 상황 | 정답 |
|---|---|
| 테스트와 Git 커밋은 허용하되 `~/Dropbox`는 건드리지 못하게 하려면 어디에 설정? | **managed-settings.json의 디렉터리 deny 규칙.** CLAUDE.md는 지침일 뿐 정책이 아니고, 개인 settings.json은 통제 대상자가 지울 수 있으므로 통제가 아니다 |
| 사내 npm 레지스트리는 열고, 공용 레지스트리 직접 접근은 막아야 함 | **`npm.finco.internal`은 allowedDomains에, 공용 레지스트리는 deniedDomains에 둘 다 명시.** 기본이 차단이라도 보안팀은 명시적 차단 기록을 원하고, 기본 동작은 버전마다 바뀔 수 있다. 프록시를 쓰기로 했다면 프록시 호스트를 허용하고 공용 레지스트리를 차단한다 |

> **핵심:** 보안팀이 "그 통제가 어디 있나요?"라고 물으면 답은 지시문이 아니라 **설정**이어야 한다. 지시는 가이드이고, 설정은 정책이다.

---

## Lesson 4. SSO(SAML, OIDC), SCIM, IP 허용 목록

> **SSO가 없으면 Claude 계정들이 있을 뿐이고, SSO가 있으면 통제되는 인력이 있다.**

- SSO(SAML 2.0 또는 OIDC)는 모든 사용자를 고객 IdP로 보낸다 → **하나의 ID, 하나의 감사 추적, 하나의 퇴사 처리 흐름.** 퇴사하면 Claude 접근도 함께 회수된다.
- ⚠️ **SSO만으로는 공백이 남는다.** 회사 도메인으로 만든 개인 계정으로 접속할 수 있으므로 **도메인 캡처**를 켜야 한다.

### 프로비저닝 모델

| 모델 | 특징 | 적합한 경우 |
|---|---|---|
| **SCIM (권장)** | IdP 주도로 전체 생애주기(생성, 그룹 변경, 해지)를 자동 동기화. 규모와 무관 | 운영 환경 기본값. 핵심은 **자동 해지** |
| **JIT** | 첫 SSO 로그인 시 자동 생성. 사전 프로비저닝·그룹 관리 없음. **만들기만 하고 지우지 않는다** | 시작 단계. SCIM으로 전환 계획 필요 |
| **수동** | 관리자가 좌석 초대 | 20~50명 파일럿. 수백 명 규모에선 부담 |

**SCIM 롤아웃 패턴:** 50~100명 파일럿 → 2~4주 모니터링 → 부서별 확대 → 전사. ID 설정은 파일럿 설계와 **동시에** 진행한다(파일럿 이후가 아니라).

### 네트워크·ID 통제

| 통제 | 답하는 질문 |
|---|---|
| **IP 허용 목록** | "누가 Claude에 접근할 수 있나" – 회사 IP 대역 밖의 요청은 연결되지 않음 |
| **테넌트 제한** | "어느 Claude 조직을 쓸 수 있나" – 회사망에서 개인·비승인 워크스페이스 로그인 차단. IP 허용 목록만으로 남는 섀도우 AI 공백을 막음 |
| **도메인 캡처** | 회사 도메인의 모든 로그인을 회사 워크스페이스로 보내 개인 계정 구멍을 막음 |
| **세션 보안** | 세션 타임아웃과 재인증 주기. 탈취되거나 방치된 세션이 유효한 시간을 제한 |

→ 섀도우 AI가 우려되는 엔터프라이즈라면 IP 허용 목록과 테넌트 제한을 **둘 다** 적용한다.

### 퀴즈: FinCo 운영 전 필수 통제 (1,800명, Okta, VPN 필수)

- **필수 4가지:** Okta SSO, 도메인 캡처, Okta에서 SCIM 프로비저닝, 회사 VPN 대역 IP 허용 목록
- **운영 후 조정:** 세션 타임아웃 설정
- **잘못된 방식:** 전 개발자에게 수동 좌석 초대

---

## Lesson 5. ZDR, 커스텀 보존, 데이터 사용 정책

> **No-training, 커스텀 보존, ZDR은 서로 다른 것이다.** 고객은 하나만 필요한데 세 가지를 다 요청하곤 한다.

### 네 가지 레버

| 레버 | 성격 | 내용 | 주의 |
|---|---|---|---|
| **No-training 약속** | 기본 적용 | 엔터프라이즈 데이터로 모델을 학습하지 않음. 모든 엔터프라이즈 플랜에 기본 적용, 별도 계약·설정 불필요 | 보존 기간이나 접근 주체는 바꾸지 않음 |
| **커스텀 데이터 보존** | 관리자 설정 | 표면별로 1일~무기한 보존 기간 설정. 관리 콘솔에서 셀프 서비스 | "데이터를 얼마나 보관하나요?"에 대한 **대부분의 실제 답** |
| **ZDR (Zero Data Retention)** | 계약 | Anthropic이 입력·출력을 전혀 저장하지 않음. **별도 Anthropic 계약 필요**, 콘솔에서 켤 수 없음. Claude Code 등 대상 API에 적용 | **판매 단계에서** 꺼낼 것(운영 직전이 아니라) |
| **HIPAA 경로** | 계약 | **BAA + ZDR.** 둘 다 있으면 BAA가 Claude Code로 자동 확장됨 | **BAA만으로는 Claude Code가 커버되지 않는다.** ZDR이 전제 조건. 헬스케어 고객이면 Anthropic AE에 일찍 둘 다 제기 |

### 고객 질문 해석하기

"Anthropic이 우리 코드를 보관하지 않게 할 수 있나요?" → 대부분 ZDR이 아니라 **커스텀 보존**(누가, 얼마나 오래 접근하나)에 관한 질문이다. 먼저 이렇게 물어볼 것: **"Anthropic 직원의 접근이 걱정이신가요, 아니면 모델 학습에 쓰이는 것이 걱정이신가요?"**

| 고객의 말 | 해당 통제 |
|---|---|
| "대화를 얼마나 보관하고, 우리가 통제할 수 있나요?" | **커스텀 보존** (콘솔에서 설정, 계약 불필요) |
| "우리 코드가 모델 개선에 절대 쓰이지 않는다는 보장이 필요해요" | **No-training** (이미 기본 적용) |
| "이 워크로드는 입력·출력이 어디에도 저장되면 안 돼요" | **ZDR** (별도 계약) |
| "PHI를 다루니 Claude Code도 BAA 적용을 받아야 해요" | **BAA + ZDR** |

### 표면별 기본 보존

| 표면 | 기본값 | 운영 전 체크 |
|---|---|---|
| Claude.ai Enterprise | 관리자 설정 가능, 최소 30일, **기본은 무기한** | 기간 제한을 원하면 누군가 직접 설정해야 함 |
| **Claude Code** | 관리자 설정 가능, 정책이 없으면 **무기한** | 보안 검토 중에 가장 놀라기 쉬운 항목. 운영 전에 설정할 것 |
| Cowork | 채팅 기록은 기기에만 로컬 저장, 서버 측 보존 없음 | Anthropic 보존이 아니라 엔드포인트 관리 문제 |
| Office Agents | 30일 후 자동 삭제, **관리자 변경 불가** | 다른 기간은 불가능하다는 기대치를 미리 설정 |

※ 강의 안에서 커스텀 보존 범위는 "1일~무기한"으로, Claude.ai Enterprise는 "최소 30일"로 적혀 있어요. 표면마다 하한이 다를 수 있으니 실제 적용 전에 확인할 것.

### O/X

- ZDR은 조직 데이터를 모델 학습에 쓰지 않는다는 뜻이다 → **X.** 그건 no-training이다. ZDR은 아예 저장하지 않는 것
- Enterprise 고객은 Claude Code 대화를 7일 후 삭제하도록 설정할 수 있다 → **O.** 별도 계약 없이 콘솔에서 가능
- 헬스케어 고객의 Claude Code HIPAA 적용에는 BAA와 ZDR이 모두 필요하다 → **O**

> **핵심:** No-training은 등급별 정책, 커스텀 보존은 설정, ZDR은 계약이다. 고객의 우려가 어느 수준에 해당하는지 파악하면, 설정 화면을 가리킬지 조달 논의를 시작할지가 정해진다.

---

## Lesson 6. SOC 2 Type II, HIPAA, 고객 통제

> **Anthropic이 컴플라이언스의 바닥을 제공하고, 고객은 그 위에 쌓는다.**

### 인증 스택 (모든 자료는 trust.anthropic.com)

SOC 2 Type 2, ISO 27001:2022, ISO 27017, ISO 27018, CSA STAR Level 2, UK Cyber Essentials, HIPAA(BAA 가능), FedRAMP High(Claude for Government), GDPR, NIST 800-171, ISO 42001:2023

| 분류 | 인증 | 답변 요령 |
|---|---|---|
| 보안 | SOC 2 Type 2, ISO 27001:2022 | "둘 다 있고 매년 감사받습니다" + 최신 보고서는 Trust Center로 |
| 클라우드·개인정보 | ISO 27017, 27018, CSA STAR Level 2 | SOC 2 외에 꼼꼼한 보안팀이 추가로 묻는 항목 |
| 규제 산업 | HIPAA(BAA), FedRAMP High, GDPR | ⚠️ **FedRAMP High는 Claude for Government 전용**, 일반 Enterprise 아님. 어떤 제품이 필요한지 먼저 확인 |
| 표준 | NIST 800-171, ISO 42001:2023 | ISO 42001은 **AI 관리 시스템 인증** → AI 거버넌스 성숙도를 보는 고객에게 강한 신호 |

> **원칙:** 모든 보안 질의서와 CIO 대화는 trust.anthropic.com으로 안내한다. 기억에 의존해 인증 정보를 답하면 위험하고, 그 페이지는 항상 최신이다.

### 소유권 모델

| Anthropic 소유 (모델 바닥) | 고객 설정 (거버넌스 계층) |
|---|---|
| Constitutional AI, RLHF, 사용 정책 집행, 런타임 분류기, 플랫폼 무결성, 출시 전 평가. 보편적이고 끌 수 없음 | managed settings, SSO·SCIM, 보존 기간, 감사 로그 접근, 커넥터 거버넌스, IP 허용 목록, 권한 규칙 → **엔게이지먼트 산출물이 있는 곳** |

### 질의서 답변 연습

| 질문 | 답변 방향 |
|---|---|
| "SOC 2 Type II가 있나요? 보고서를 볼 수 있나요?" | **Trust Center로 안내** (기억에 의존하지 말 것) |
| "관리자가 런타임 안전 분류기를 끄거나 바꿀 수 있나요?" | **Anthropic 소유.** 어떤 설정으로도 바꿀 수 없으며, 이건 제약이 아니라 안심 요소 |
| "대화 데이터 보존 기간은 누가, 어디서 정하나요?" | **고객이 설정** (관리 콘솔) |
| "FedRAMP High가 우리가 사는 일반 Enterprise에도 적용되나요?" | **Trust Center 안내 + 주의사항:** Claude for Government 전용 |

### 운영 통제: 감사 로그와 Compliance API

- **감사 로그:** 사용자 활동, 대화, 관리자 변경을 추적한다. **180일 롤링 보관**, SIEM으로 내보낼 수 있다. 첫 사용부터 쌓이므로 운영 전에 접근을 설정할 것.
- **Compliance API:** DLP, eDiscovery, 법적 보존을 위해 대화 내용에 프로그램으로 접근한다. **최대 6년 보관.**
- 둘 다 Enterprise 기능이며 **기본으로 설정되어 있지 않다.**

### O/X

- FedRAMP High는 모든 Claude Enterprise 플랜에 적용된다 → **X**
- 고객 시스템 프롬프트로 런타임 안전 분류기를 재설정할 수 있다 → **X**
- SOC 2 Type 2 보고서 등 컴플라이언스 자료는 trust.anthropic.com에서 받을 수 있다 → **O**

### 코스 산출물: OTel 설정 + 대시보드 스펙

1. 제공된 OTel 참조 설정을 고객 환경에 맞게 조정 (OTLP 엔드포인트, 서비스 이름, 수집기 연결 확인)
2. 활성화 스코어카드를 기준으로 지표 3개짜리 대시보드 설계 (지표별 이름, 데이터 소스, 정상 기준치)
3. 둘 다 고객 프로젝트 디렉터리에 커밋 → 운영팀이 넘겨받아 첫 알림까지의 시간을 줄임

---

## 전체 정리

- **managed-settings.json:** 최상위 정책 파일. Day 0에 `forceLoginMethod`, `forceLoginOrgUUID`, `allowedMcpServers`, `allowManagedMcpServersOnly`를 설정한다. 이후 hooks·권한·버전·조직 CLAUDE.md를 잠근다. 배포는 MDM, 서버 관리형, 파일 중 고객 환경에 맞게.
- **권한:** 비파괴·빈번한 작업은 Allow, 되돌릴 수 없거나 네트워크로 나가는 작업은 Ask, 쓸 일 없는 위험 작업은 Deny. `allowManagedPermissionRulesOnly`로 잠근다. 역할은 누적되므로 그룹 중심으로 설계한다.
- **샌드박스:** 파일시스템 allowWrite/denyWrite/denyRead, 네트워크 allowedDomains/deniedDomains(deny 우선). 통제는 지시문이 아니라 설정으로.
- **ID:** SSO + 도메인 캡처 + SCIM(자동 해지) + IP 허용 목록 + 테넌트 제한.
- **데이터:** No-training은 기본, 커스텀 보존은 설정, ZDR은 계약, HIPAA는 BAA + ZDR. Claude Code 기본 보존은 무기한이므로 운영 전에 설정한다.
- **컴플라이언스:** 인증은 Trust Center로 안내하고, FedRAMP는 Government 전용임을 짚는다. 감사 로그(180일)와 Compliance API(최대 6년)는 운영 전에 설정한다.

다음 코스: **Administration and Measurement** (조직 설정, 좌석·접근 관리, 사용량 API, 도입·ROI 리포팅, 감사 로그와 운영 주기)
