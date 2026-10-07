# Administration and Measurement 요약 (Claude Partner Badge: Claude Code)

총 6개 레슨: ① 좌석 관리와 관리 콘솔 ② 사용량·도입 분석 ③ Compliance API ④ OpenTelemetry 파이프라인 ⑤ 감사 로그 ⑥ 비용 모니터링·귀속·지출 통제

이 코스의 산출물: **OTel Configuration + Dashboard Spec** (OTel 설정 블록 + 대시보드 패널 4개: 세션, 활성 개발자, 모델별 토큰량, 팀별 비용)

> ※ 시나리오 문제는 시뮬레이션이라 정답이 화면에 따로 표시되지 않습니다. 아래 "권장 답"은 강의 본문 내용을 기준으로 정리한 것입니다.

---

## 핵심 한 장 요약: 무엇이 어떤 질문에 답하나

| 표면 | 답하는 질문 | 대화 내용 | 보관 | 접근 권한 |
|---|---|---|---|---|
| **Admin Console Analytics** | "잘 쓰이고 있나?" (DAU, 좌석 활용, 지출) | ✗ | 일별 스냅샷 | Owner, Primary Owner |
| **OTel** | "무엇이 **실행**됐나?" (도구 호출, bash, MCP) | ✗ (기본값) | SIEM 설정에 따름 | 옵트인 설정 필요 |
| **Compliance API** | "무엇이 **논의**됐나?" (대화 내용) | **✓ 유일** | 최대 6년, 법적 보존 | **Primary Owner만** |
| **감사 로그** | "조직에서 무엇이 **바뀌었나**?" (로그인, 멤버, 설정) | ✗ | 180일 롤링 | Owner, Primary Owner |
| **Admin API** | 조직 관리 (사용자 프로비저닝, API 키, 지출 한도) | ✗ | – | Owner, Primary Owner |

---

## Lesson 1. 좌석 관리와 관리 콘솔

> **관리 콘솔은 인프라다.** 30일 활성화에서 킥오프 전 기간(Day -14 ~ Day 0)에 SSO, 좌석, 지출 한도를 끝내야 한다. 놓치면 Day 1에 에스컬레이션으로 돌아온다.

흔한 실패는 기능 부족이 아니라 **잘못된 사람이 잘못된 역할에 있는 것**이다. 디스커버리 때 잡으면 5분, 킥오프 때 잡으면 반나절짜리 에스컬레이션이 된다.

- **역할 확인:** Admin은 이름과 달리 할 수 있는 게 적다
- **좌석 유형 확인:** Standard 좌석은 Claude Code 인증 자체가 안 된다
- **지출 한도 설정:** 사용량 기반 플랜의 기본값은 **$0**이라 모든 API 호출이 거부된다

### 좌석 유형

| 과금 모델 | 좌석 | Claude Code | 비고 |
|---|---|---|---|
| 좌석 기반 (레거시) | **Standard** | ✗ | Claude.ai 채팅만. 레거시 플랜에서 원인 모를 인증 오류가 나면 좌석 유형부터 확인 |
| 좌석 기반 (레거시) | **Premium** | ✓ | Primary Owner만 구매·배정 가능. 다음 계약 갱신 때 사용량 기반으로 전환됨 |
| 사용량 기반 (전환기) | **Chat / Chat + Code** | Chat+Code만 ✓ | 단계적 폐지 중. 다음 갱신 이후 사용 불가 |
| 사용량 기반 (현재) | **Claude Enterprise** | ✓ | Claude.ai + Claude Code + Cowork 통합. 좌석 요금에 API 요금이 추가됨. **조직 지출 한도 기본값 $0** |

### 역할 계층

| 역할 | 할 수 있는 것 | 비고 |
|---|---|---|
| **Primary Owner** (조직당 1명) | 좌석 구매·배정, 데이터 내보내기 요청, 소유권 이전, Owner의 모든 권한 | 보통 CTO, IT 디렉터, 서비스 계정. **일정 잡기 가장 어려운 사람**이므로 킥오프 전 주 전에 미리 확보 |
| **Owner** | SSO·SCIM 설정, 감사 로그, 데이터 보존 설정, 사용량 분석, Admin·Owner 초대·제거 | 대부분의 IT 담당자. 역할을 하나만 줄 수 있다면 **반드시 Owner** |
| **Admin** | 멤버 초대·제거, 사용량 분석(Enterprise만) | ✗ SSO·SCIM 설정 불가, ✗ 좌석 구매 불가, ✗ 감사 로그 불가. **킥오프 전 가장 흔한 공백** |
| **User** | 채팅, 프로젝트, Claude Code(좌석 유형이 맞을 때) | 관리 기능 없음 |

### 킥오프 전 체크리스트 (이 순서로)

1. **ID 설정** (Owner 이상): SAML 2.0/OIDC SSO를 소규모 파일럿 그룹으로 먼저 테스트 → 도메인 캡처 → 개발자 20명 이상이면 SCIM (JIT는 간단하지만 통제가 약함)
2. **좌석 배정과 지출 한도** (좌석은 Primary Owner): Settings > Organization > Members. **누가 API를 호출하기 전에 조직 지출 한도 설정**, 필요하면 사용자별 한도도
3. **managed-settings.json 배포** (개발자 설치 **전**): 권한 잠금, 인증 방식 강제, 최소 버전, MCP 허용 목록. 이미 진행 중인 세션에는 소급 적용되지 않는다
4. **개발자 인증 확인:** `claude` 실행 → "Claude account with subscription" 선택 → Enterprise SSO 인증 → `/status`로 좌석과 인증 제공자 확인. 개인 계정으로 로그인한 적이 있으면 먼저 `/logout`

### 시나리오: 14일, IT 담당자 1명 (Priya)

| 상황 | 권장 답 |
|---|---|
| 1. 첫 30분 통화에서 먼저 확인할 것 | **관리 콘솔에서 그녀의 역할과 그 역할로 할 수 있는 일** |
| 2. 역할이 Admin. SSO 설정과 좌석 30개 추가가 필요 | **둘 다 막혀 있다. 지금 바로 역할을 올린다** (Admin은 SSO도 좌석도 불가) |
| 3. Owner로 올려 좌석 배정 완료. 킥오프 당일 모든 API 호출 실패 | **조직 지출 한도 확인** (사용량 기반 기본값 $0, 전원 동시 실패의 전형) |
| 4. 보안팀이 미승인 MCP 서버 연결을 지적 | **managed-settings.json에 MCP 허용 목록 설정** (온보딩 문서나 건별 승인은 강제력이 없음) |

---

## Lesson 2. 사용량·도입 분석 (Analytics)

> **측정할 수 없으면 방어할 수 없다.** Admin Console은 "잘 되고 있나?"에, OTel은 "비즈니스에 도움이 되나?"에 답한다.

### 측정 도구 2가지

| | Admin Console | OTel 파이프라인 |
|---|---|---|
| 설정 | 없음. 배포 직후 Owner가 Settings > Analytics에서 확인 | `CLAUDE_CODE_ENABLE_TELEMETRY=1` 필요. managed-settings로 조직 배포 |
| 데이터 | 일·주간 활성 사용자, 사용자당 세션, 조직 지출, 좌석 활용률 | `session.count`, `active_time.total`, `lines_of_code.count`, `commit.count`, `pull_request.count`, `cost.usage`, `token.usage` |
| 용도 | Day 7/14/30 리포트, 챔피언 브리핑 | 엔지니어링 지표, ROI 모델, 대규모 배포의 **개발자별 비용 추적**(Admin Console에는 없음) |

- 대부분의 Day 30 리포트는 Admin Console만으로 충분하다.
- **Analytics Chat:** Admin Console에서 대화형으로 사용량을 조회할 수 있다(예: "지난주 세션이 가장 많은 팀은?").
- ※ 강의 메모: Anthropic **Analytics API**는 claude.ai 일반 사용 데이터용이다. Claude Code의 세션·비용 지표는 OTel이나 Admin Console에서 가져와야 한다. (Lesson 6에서는 Analytics API가 "모든 Claude 사용"을 포함한다고 설명해 표현이 다소 다르다.)

### Day 30 스코어카드 지표 4가지

| 지표 | 목표 | 주의 신호 | 출처 |
|---|---|---|---|
| **DAU** (도입) | 3주차까지 60% | 2주차 50% 미만 정체는 조기 경고. 4주차 40% 미만이면 제품이 아니라 **변화 관리** 문제 | Admin Console > Analytics > Active users |
| **사용자당 세션** (깊이) | 하루 2~3회 | 1.5회 미만이면 열어보기만 하고 워크플로우에 녹아들지 않은 것. `active_time.total`과 함께 볼 것 | Admin Console 또는 OTel |
| **2주차 리텐션** (정착) | 2주차 이후 80% 이상 | 첫 주 이후 하락은 흔하다(신기함이 사라짐). 2주차 이후 80% 미만이면 확대 시 이탈 예고 | Admin Console 코호트 뷰 |
| **개발자당 비용** (비즈니스 근거) | 매주 추적 | 세션·산출물 증가 없이 비용만 치솟으면 과사용이나 지출 한도 설정 오류 | Admin Console > Billing 또는 OTel `cost.usage` |

**슈퍼 유저 패턴:** 대부분 10~15명이 비용과 산출물의 큰 부분을 차지한다. 비용이 튀면 합계가 아니라 개발자별로 먼저 볼 것. 해결책은 **모델 매칭**(최상위 모델이 필요 없는 워크플로우는 가벼운 모델로)이며, 청구서에 놀라기 전에 4주차 리포트에서 다룬다.

### OTel exporter 3가지 경로

| 경로 | 설정 | 용도 |
|---|---|---|
| **Console** (디버그) | `CLAUDE_CODE_ENABLE_TELEMETRY=1` | 터미널에 출력. `session.count`, `cost.usage`가 보이면 파이프라인 정상 |
| **Prometheus** (표준) | `+ OTEL_METRICS_EXPORTER=prometheus`, `OTEL_EXPORTER_PROMETHEUS_PORT=9464` | Grafana/Prometheus 보유 팀. 머신별 설정이 필요하므로 20명 이상이면 OTLP로 |
| **Enterprise OTLP** | managed-settings.json으로 조직 전체 배포 | **50명 이상** 또는 일관성·중앙 통제가 중요할 때. 개발자 설치 전에 배포 |

※ 강의 예시는 managed-settings.json에 `"telemetry": {"enabled": true, "otlpEndpoint": "..."}` 형태로 보여준다. Lesson 4에서는 같은 내용을 managed-settings의 env 블록으로 배포하는 방식으로 설명하니, 실제 키 이름은 최신 문서로 확인할 것.

### Day 30 리포트 4단계

1. **기준선 확보** (Day 1 또는 킥오프 전): 스프린트당 처리 티켓, 개발자당 PR, 반복 작업 시간(자기 보고). 놓쳤다면 리포트에 명시하고 다음 롤아웃 권고로 남긴다
2. **4주차 데이터 수집:** Settings > Analytics에서 DAU, 사용자당 세션, 2주차 리텐션. 필요하면 CSV로, OTel이 있으면 `cost.usage`도 교차 확인
3. **스코어카드 구성:** 지표별로 Green(목표 이상), Amber(목표 대비 15% 이내), Red(기준 미달) + 한 문장 해석. **맥락 없는 숫자 금지** ("DAU 52%"만으로는 좋은지 나쁜지 모른다)
4. **챔피언 브리핑:** 전체 판정(G/A/R) → 확대 결정을 가장 직접 뒷받침하는 지표 → 세부는 질문용으로 남김. Red가 있으면 **개선 계획과 함께**

### 시나리오: Day 28, 리포트 48시간 전

| 상황 | 권장 답 |
|---|---|
| 1. OTel이 배포되지 않음. 도입 수치는 어디서? | **Admin Console > Analytics** |
| 2. DAU 48%, 세션 1.6회, 리텐션 84%, 비용 $3.10/일 (목표 60% / 2~3회 / 80%) | **DAU와 세션은 Amber, 리텐션과 비용은 Green + 격차 해소 계획 제시** (DAU 48%는 목표의 80%라 엄밀히는 15% 범위를 살짝 벗어나지만, 강의의 의도는 "부족한 점을 이름 붙이고 계획을 제시하라"로 보인다) |
| 3. 기준선 없이 before/after를 요구받음 | **4개 목표 대비 절대 진척으로 보고하고, 기준선 공백을 명시하고 다음 롤아웃에서 확보하도록 권고** (기억에 의존한 추정이나 일정 지연은 X) |
| 4. "CFO에게 연간 비용을 어떻게 정당화하죠?" | **세션당 비용으로 제시:** $3.10 ÷ 1.6 ≈ AI 지원 작업 세션당 $2 미만. 개발자 시간당 인건비와의 비교는 CFO가 하게 두고, 결론이 아니라 올바른 입력값을 준다 |

---

## Lesson 3. Compliance API (감사·감독)

> **대화 내용을 담는 유일한 표면.** 다른 모든 표면은 메타데이터만 수집한다.

### 개요

| 항목 | 내용 |
|---|---|
| 데이터 | 채팅 입력·출력, 파일 업로드, 활동 피드 이벤트 |
| 보관 | **최대 6년.** Enterprise 기본값은 무기한, 커스텀은 1일~무기한. **법적 보존(legal hold)**으로 특정 사용자·대화를 보존 기간 이후까지 유지 |
| 접근 | **Primary Owner만** 활성화하고 접근 키를 만들 수 있다 (Owner 불가) |
| 기본 상태 | **꺼져 있음** |

### 4가지 표준 연동 (보안 벤더 28곳 이상 지원)

| 분류 | 연동 예시 | 엔드포인트 |
|---|---|---|
| **SIEM** | Splunk, Microsoft Sentinel, OTLP 백엔드 | `GET /v1/organizations/{id}/audit-events` (Enterprise·Platform) |
| **DLP / CASB** | Nightfall, Microsoft Purview, Symantec | `GET /v1/organizations/{id}/conversations` (Enterprise만) |
| **eDiscovery** | Relativity, Exterro | `GET …/conversations/{conversation_id}` (사용자·대화 ID로 조회, Enterprise만) |
| **AI 보안 상태** | 사용 패턴 기반 행동 위험 탐지 | 위와 동일 엔드포인트 |

→ "우리 기존 보안 스택과 연동되나요?"라는 질문에 대부분의 엔터프라이즈 환경에서 답은 "예"다.

> ⚠️ **Analytics API와 혼동 금지.** Analytics API는 도입·사용 신호(DAU, 좌석, 지출)이지 대화 내용이 아니다.

### 활성화

- **Enterprise:** Organization settings › API → Compliance API에서 Enable → + Create key → 키는 **한 번만 표시**되므로 안전하게 보관. **연동마다 별도 키**(교체 시 영향 범위 제한)
- **Claude Platform(API 고객):** 플랫폼 콘솔에서 활성화. 문서: `platform.claude.com/docs/en/manage-claude/compliance-api`. 고객이 Enterprise인지 Platform인지 먼저 확인할 것(절차가 다르다)

### 운영 전 컴플라이언스 체크리스트 (SSO·프로비저닝과 함께)

- Compliance API 활성화 + 접근 키 생성
- 감사 로그 접근 확인 (Organization settings › Data and Privacy)
- 데이터 보존 정책 문서화 및 설정
- 법무·컴플라이언스팀과 법적 보존 절차 합의
- 보안 정책상 필요하면 DLP/SIEM 연동

→ 운영 전에 끝내면, 사고나 감사 요청이 온 뒤 과거 데이터를 끌어오는 느린 복구 경로를 피할 수 있다.

### 활성화 5단계

1. 요구사항 확인 (eDiscovery / DLP / 법적 보존 / SIEM)
2. **Primary Owner 참석 확인** (Owner는 설정에서 옵션 자체가 안 보인다)
3. Organization Settings › API에서 활성화 (1분 이내, 엔지니어 불필요)
4. 연동별 접근 키 생성
5. DLP, SIEM, eDiscovery 플랫폼에 연결

### 시나리오: 컴플라이언스 요청 라우팅

| 상황 | 권장 답 |
|---|---|
| 1. HR 조사를 위해 특정 직원의 최근 90일 대화 열람 | **Compliance API (사용자 ID로 조회)** – 대화 내용이 있는 유일한 곳 |
| 2. bash 명령 실행 시 실시간 보안 알림 | **OTel + `OTEL_LOG_TOOL_DETAILS=1` → SIEM** |
| 3. 특정 직원 대화를 3년간 법적 보존 | **Compliance API로 해당 사용자 법적 보존** (조직 전체 보존 기간 변경이나 감사 로그 CSV는 X) |
| 4. 대화 속 PII 공유 모니터링 | **Compliance API에 DLP 도구 연결** |

---

## Lesson 4. OpenTelemetry 파이프라인 설정

> **OTel은 무엇이 실행됐는지 알려준다.** 개발자가 운영 DB에 특정 bash 명령을 실행했는지 답할 수 있는 유일한 수단이다.

### 개요

- **지표 8개:** 세션, 토큰, 비용(`cost.usage`, USD), 코드 라인, 커밋, PR, 코드 편집 결정, 활성 시간
- **이벤트:** 모든 도구 호출, API 요청, MCP 호출이 구조화된 로그로
- **내보내기:** 지표는 OTLP, Prometheus, console. 이벤트는 OTLP, console. 지표와 이벤트를 다른 백엔드로 보낼 수도 있다. 기본 프로토콜은 **gRPC, 4317 포트**

### 개인정보 경계 (옵트인 플래그)

| 플래그 | 캡처 내용 | 기본값 |
|---|---|---|
| `OTEL_LOG_USER_PROMPTS=1` | **프롬프트(대화) 내용** | 꺼짐 |
| `OTEL_LOG_TOOL_DETAILS=1` | 도구 파라미터, **bash 명령** | 꺼짐 |
| `OTEL_LOG_TOOL_CONTENT=1` | 도구 출력 전체(파일 내용, 명령 출력). **추가로 `CLAUDE_CODE_ENHANCED_TELEMETRY_BETA=1` 필요**, 60KB에서 잘림 | 꺼짐 |

### 파이프라인 5단계

1. **마스터 스위치:** `CLAUDE_CODE_ENABLE_TELEMETRY=1` (없으면 다른 OTel 변수는 모두 무시됨)
2. **exporter 선택:** `OTEL_METRICS_EXPORTER=otlp`, `OTEL_LOGS_EXPORTER=otlp` (디버그는 console)
3. **엔드포인트와 인증:** `OTEL_EXPORTER_OTLP_ENDPOINT`, `OTEL_EXPORTER_OTLP_HEADERS` (Bearer 토큰). gRPC를 지원하지 않는 수집기라면 `OTEL_EXPORTER_OTLP_PROTOCOL=http/json`
4. **managed settings로 배포:** MDM으로 env 블록을 managed-settings.json에 넣는다. 개발자가 덮어쓸 수 없고 토큰도 개발자 환경에 남지 않는다. **100명 이상이면 유일하게 확장 가능한 방법**
5. **짧은 주기로 검증:** `OTEL_METRIC_EXPORT_INTERVAL=10000`(기본 60초 → 10초)으로 확인한 뒤 운영 전에 제거

### 환경별 설정

```bash
# 1) 로컬 디버그
CLAUDE_CODE_ENABLE_TELEMETRY=1
OTEL_METRICS_EXPORTER=console
OTEL_LOGS_EXPORTER=console
OTEL_METRIC_EXPORT_INTERVAL=10000   # 운영 전 제거

# 2) 운영 OTLP (managed-settings.json으로 배포, .env 금지)
CLAUDE_CODE_ENABLE_TELEMETRY=1
OTEL_METRICS_EXPORTER=otlp
OTEL_LOGS_EXPORTER=otlp
OTEL_EXPORTER_OTLP_PROTOCOL=grpc
OTEL_EXPORTER_OTLP_ENDPOINT=http://collector.example.com:4317
OTEL_EXPORTER_OTLP_HEADERS=Authorization=Bearer your-token

# 3) 멀티팀 귀속 (팀마다 다른 값을 MDM으로)
OTEL_RESOURCE_ATTRIBUTES=department=engineering,team.id=platform,cost_center=eng-123
```

### 신호 유형

| 유형 | 조건 | 내용 |
|---|---|---|
| **지표** | 마스터 스위치만 | 8개 시계열. `cost.usage`에는 model, query_source, speed, effort, 도구 속성이 붙음 |
| **이벤트** | `OTEL_LOGS_EXPORTER` 필요 | 도구 호출, API 요청, 권한 모드 변경, MCP 연결, 인증 이벤트 |
| **비용 귀속** | `cost.usage`에 내장 | model, query_source, speed, effort, agent.name, skill.name, plugin.name, mcp_server.name, mcp_tool.name + 사용자 정의 리소스 속성 |
| **보안 이벤트** | 일부 옵트인 | 도구 결정(승인/거부), 권한 모드 변경, MCP 연결은 기본 수집. bash 내용은 `OTEL_LOG_TOOL_DETAILS` 필요 |

### 시나리오: 개발자 500명 배포

| 상황 | 권장 답 |
|---|---|
| 1. CISO가 모든 bash 명령을 Splunk에서 보고 싶어 함 | **`OTEL_LOGS_EXPORTER=otlp` + `OTEL_LOG_TOOL_DETAILS=1`** |
| 2. 한 조직 안에서 부서별 비용 분리 | **MDM으로 `OTEL_RESOURCE_ATTRIBUTES`(department, cost_center) 배포** (부서별 API 키나 조직 분리는 X) |
| 3. 500대에 개발자가 못 바꾸게 배포 | **MDM으로 managed-settings.json에 env 블록 배포** (런북이나 공유 .env는 X) |
| 4. CISO: "대화 내용이 OTel에 수집되나요?" | **"아니요. 프롬프트는 기본 수집되지 않으며, `OTEL_LOG_USER_PROMPTS=1`은 현재 설정에 없습니다."** |

---

## Lesson 5. 감사 로그와 조회

> **감사 로그는 관리 이벤트 기록이지 대화 기록이 아니다.**

### 개요

| 항목 | 내용 |
|---|---|
| 이벤트 | **33종:** 로그인, 프로젝트·파일 이벤트, 멤버 추가·제거, 관리 설정 변경. 대화나 도구 호출 수준은 없음 |
| 보관 | **180일 롤링** (그 이전은 자동 삭제). 더 오래 필요하면 주기적으로 내보내거나 OTel SIEM으로 보냈어야 한다 |
| 접근 | Owner, Primary Owner. API 키나 설정 없이 Organization settings › Data and Privacy에서 CSV 다운로드 |
| 없는 것 | 대화 내용, 도구 호출, bash 명령, 프롬프트·응답 |

### 접근 방법 3가지

| 방법 | 언제 | 한계 |
|---|---|---|
| **CSV 내보내기** (일회성) | 로그인 이력·관리 변경 기록 요청, 일회성 거버넌스 질문 | 180일 전체, 날짜 필터 없음 |
| **Compliance API** (프로그램) | 특정 사용자·이벤트 유형·날짜 범위 조회, **180일을 넘는 기간** (예: 특정 사용자의 18개월 활동) | Compliance API 활성화 + Primary Owner 키 필요 |
| **OTel / SIEM** (실시간) | 권한 모드 변경 실시간 알림, 실행 이벤트와 관리 이벤트 통합 대시보드 | 파이프라인 활성화 시점 이후만. **과거 데이터 백필 없음** |

### 감사 로그 vs OTel

- **감사 로그:** 가입·탈퇴·설정 변경, 로그인, 프로젝트 생성·삭제. 어떤 Owner든 바로 볼 수 있다. ✗ 도구 호출, bash, 대화, 180일 이전 이벤트
- **OTel:** 모든 도구 호출, bash, MCP, 권한 모드 변경, API 요청이 실시간으로. 보관 기간은 SIEM이 정한다. ✗ 스키마에 없는 조직 관리 변경, 활성화 이전 이벤트, 대화 내용

### 시나리오

| 상황 | 권장 답 |
|---|---|
| 1. 지난주 Claude Code로 실행된 모든 bash 명령 기록 | **OTel SIEM에서 해당 사용자의 도구 이벤트 조회** (`OTEL_LOG_TOOL_DETAILS=1` 전제) |
| 2. 규제 감사를 위해 특정 직원의 18개월치 대화 | **Compliance API (사용자 ID + 날짜 범위)** |
| 3. 권한 모드가 default에서 bypass로 바뀌면 60초 안에 Slack 알림 | **`claude_code.permission_mode_changed` OTel 이벤트에 SIEM 규칙** |

> **원칙:** 감사 로그는 거버넌스, OTel은 실행, Compliance API는 내용. 각자 자기 영역이 있다.

---

## Lesson 6. 비용 모니터링, 귀속, 지출 통제

> **귀속되지 않은 비용은 그냥 숫자일 뿐이다.**

### 비용 가시성 3계층 (대부분 셋 다 쓴다)

| 계층 | 내용 | 용도 |
|---|---|---|
| **Admin 대시보드** | Organization settings의 모델·기능별 지출. 매일 갱신, API 불필요 | 경영진 요약 (팀별 분석이나 프로그램 리포트에는 부적합) |
| **Analytics API** | 워크스페이스별 집계 지출·사용량. 일별 스냅샷, 날짜 범위 조회, BI 내보내기. Claude Code만이 아니라 모든 Claude 사용. **관리자 API 키** 필요 | 내부 비용 리포트 |
| **OTel `cost.usage`** | 세션별 USD 지출. **`OTEL_RESOURCE_ATTRIBUTES`로 팀별 귀속이 가능한 유일한 표면**. 실시간 | 부서별 비용 배분(charge-back) |

### `claude_code.cost.usage` 지표

- **측정:** 세션 단위로 누적되는 USD 지출(입력·출력 토큰, 캐시 읽기·생성 포함). 요청 단위가 아니라 세션 단위이므로 리포트에는 세션 합계를 쓴다.
- **짝 지표 `claude_code.token.usage`:** input, output, cacheRead, cacheCreation별 토큰 수. 비용의 **구성**을 이해할 때 쓴다(캐시 읽기와 생성 비율 → 세션 설계 효율).
- **내장 속성:** `model`, `query_source`(human / subtask / tool_use), `agent.name`, `skill.name`, `plugin.name`, `mcp_server.name`, `mcp_tool.name`
- **사용자 정의 라벨:** MDM 프로필 범위별로 다른 `OTEL_RESOURCE_ATTRIBUTES`. 내장 속성에 **추가**된다(대체가 아님).

⚠️ **근사치 주의:** `cost.usage`는 공개 가격표를 토큰 수에 적용한 **클라이언트 측 추정치**다. 실제 청구 금액은 API 제공자(Anthropic Console, Amazon Bedrock, Google Vertex)가 기준이다.

- OTel 비용 → 팀별 배분, 팀 예산, 추세 모니터링
- 제공자 청구 대시보드 → 실제 청구서 금액. **재무팀에 OTel 수치를 청구 금액으로 말하지 말 것**
- 차이는 보통 작지만 가격 변경이나 캐싱 동작 변화 때 생길 수 있다 → **분기별 대사**

### 지출 통제 4가지

| 통제 | 방법 |
|---|---|
| **지출 한도 (예산 강제)** | Admin API `/organizations/{id}/spend-limits`로 워크스페이스별 월 한도. 도달하면 한도가 리셋되거나 올라갈 때까지 요청이 정상적으로 거부된다. 조직 단위가 아니라 **팀 워크스페이스 단위로** |
| **멀티팀 귀속** | MDM managed-settings 프로필별 `OTEL_RESOURCE_ATTRIBUTES` → SIEM에서 `cost.usage`를 라벨별로 묶는 대시보드 |
| **지출 알림** (OTel 필요) | 팀 라벨별 일·주간 누적 `cost.usage`가 임계치를 넘으면 SIEM 규칙 → Slack/이메일. 사전 경보이고, Admin API 한도는 최종 차단 장치 |
| **재무 브리핑** | OTel 비용은 추정치임을 항상 고지. 실제 청구서는 제공자 콘솔, 분기별 대사 |

### 시나리오

| 상황 | 권장 답 |
|---|---|
| 1. CTO가 리더십 회의용 월간 모델별 지출 요약을 원함 (팀별 불필요) | **Admin 대시보드** (Organization settings) |
| 2. 재무팀이 8개 엔지니어링 팀별 월 비용 배분을 원함 | **MDM으로 팀 라벨(`OTEL_RESOURCE_ATTRIBUTES`) 배포 + SIEM에서 `cost.usage` 조회** |
| 3. SIEM 월 합계가 Console 청구서보다 3% 낮음 | **"OTel 수치는 추정치이고, Console 청구서가 공식 금액입니다."** 분기별 대사, 작은 차이는 정상 |

### 코스 산출물 완성

- 비용 귀속 패널 추가: 팀별 주간 `cost.usage` (보조 속성: model, query_source)
- 고객의 지출 알림 임계치 정의 (사용자별 월 한도, Admin API 또는 Admin Console)
- 최종 산출물: **OTel 설정 블록 + 대시보드 패널 4개** (세션, 활성 개발자, 모델별 토큰량, 팀별 비용) → Day 1 DevOps 인수인계

---

## 전체 정리

- **좌석·역할:** 사용량 기반 플랜은 지출 한도 기본값이 $0. Standard 좌석은 Claude Code가 불가하다. Admin은 SSO·좌석·감사 로그를 못 다루므로 IT 담당자는 Owner, 좌석은 Primary Owner. managed-settings는 개발자 설치 전에 배포한다.
- **도입 측정:** Admin Console부터 본다. DAU 60%, 세션 하루 2~3회, 2주차 리텐션 80%, 개발자당 비용을 G/A/R과 한 줄 해석으로 보고한다. 기준선은 Day 1 전에 확보한다.
- **Compliance API:** 대화 내용을 볼 수 있는 유일한 곳(최대 6년, 법적 보존). Primary Owner만 다루며, 연동마다 키를 따로 만든다.
- **OTel:** `CLAUDE_CODE_ENABLE_TELEMETRY=1`이 마스터 스위치. 지표는 기본, 이벤트는 logs exporter 필요, 프롬프트·도구 상세는 옵트인. managed-settings로 배포하고, 팀 라벨은 `OTEL_RESOURCE_ATTRIBUTES`.
- **감사 로그:** 관리 이벤트, 180일, CSV. 더 오래된 기간은 Compliance API, 실시간은 OTel SIEM.
- **비용:** 요약은 대시보드, 리포트는 Analytics API, 팀별 배분은 OTel `cost.usage`(추정치). 한도는 Admin API로 팀 워크스페이스별로 건다.

다음 코스: **Delivery Methodology** (엔게이지먼트 범위 설정, 파일럿 운영, 롤아웃 관리, 인수인계)
