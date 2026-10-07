#CCDV-F
#Claude
#API

---

# D2-1. Claude API 메커니즘 (6.8%)

> Applications and Integration(33.1%)의 핵심 하위 영역. Messages API의 요청/응답 구조, 스트리밍, 오류 처리를 정확히 아는지 묻는다.

## 1. Messages API 기본

```
POST https://api.anthropic.com/v1/messages
헤더: x-api-key, anthropic-version: 2023-06-01, content-type: application/json
```

### 필수 파라미터
| 파라미터 | 설명 |
| --- | --- |
| `model` | 모델 ID (예: `claude-opus-5-5`) |
| `messages` | `{role, content}` 배열. role은 **`user` 또는 `assistant`만** |
| `max_tokens` | 생성할 최대 토큰 수 (모델별 상한 존재) |

### 주요 선택 파라미터
| 파라미터 | 설명 |
| --- | --- |
| `system` | 시스템 프롬프트. **`messages` 배열이 아니라 최상위 필드** |
| `temperature` | 0.0~1.0 (기본 1.0) |
| `top_p` / `top_k` | 샘플링 제어. temperature와 동시 조정은 비권장 |
| `stop_sequences` | 생성을 멈출 문자열 배열 |
| `stream` | `true`면 SSE 스트리밍 |
| `tools` / `tool_choice` | 도구 정의와 선택 방식 → [[06 도구와 MCP]] |
| `thinking` | `{type: "adaptive" \| "enabled" \| "disabled", display, budget_tokens}` |
| `output_config` | `format`(구조화된 출력), `effort` |
| `metadata` | `{user_id}` 등 |
| `service_tier` | `auto` / `standard_only` |
| `cache_control` | 프롬프트 캐싱 |
| `inference_geo` | `global`(기본) / `us` (4.6 이후 모델, 1.1x 과금) |
| `speed` | `fast` (Fast mode, 일부 Opus) |

### 메시지 구성 규칙
- 연속된 같은 role의 턴은 **자동으로 결합**된다
- 대화는 `user` 메시지로 시작한다
- **어시스턴트 프리필**(마지막 메시지를 assistant로 두기)은 Claude 4.6 이후 모델에서 **미지원 → 400 오류**. 대신 구조화된 출력이나 시스템 프롬프트 지시 사용

## 2. 응답 객체

```json
{
  "id": "msg_01A7dGW...",
  "type": "message",
  "role": "assistant",
  "content": [ { "type": "text", "text": "..." } ],
  "model": "claude-opus-5-5",
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 150,
    "output_tokens": 42,
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 0
  }
}
```

> `usage.input_tokens`는 **마지막 캐시 브레이크포인트 이후의 토큰**만 센다.
> `총 입력 = cache_read_input_tokens + cache_creation_input_tokens + input_tokens`

### 콘텐츠 블록 타입

| 요청에 넣는 블록 | 응답에 오는 블록 |
| --- | --- |
| `text`, `image`, `document`, `container_upload`, `tool_use`(이전 턴), `tool_result`, `thinking` | `text`, `tool_use`, `server_tool_use`, `thinking`, `redacted_thinking`, 서버 도구 결과 블록 |

## 3. stop_reason (시험 필수)

| 값 | 의미 | 처리 |
| --- | --- | --- |
| `end_turn` | 자연스럽게 완료 | 그대로 사용 |
| `max_tokens` | `max_tokens` 한도 도달 | 응답이 **잘렸다**. max_tokens 증가 또는 이어받기 |
| `stop_sequence` | 사용자 정의 정지 문자열과 매칭 | `stop_sequence` 필드 확인 |
| `tool_use` | 도구 호출 요청 | 도구 실행 후 `tool_result` 반환 |
| `pause_turn` | 서버 도구 루프 반복 한도 도달 | **어시스턴트 응답을 그대로 다시 보내** 계속 진행 |
| `refusal` | 모델이 응답 거부 | `stop_details` 확인 후 폴백 모델로 재시도 또는 프롬프트 수정 |
| `model_context_window_exceeded` | 컨텍스트 윈도우 한도 도달 | 잘린 응답으로 처리 |

**핵심 구분**: `stop_reason`은 **성공 응답(HTTP 200)의 일부**이고, 오류는 **HTTP 4xx/5xx**다. 둘을 같은 로직으로 처리하면 안 된다.

### 빈 응답 방지
`tool_result` 블록 **바로 뒤에 텍스트 블록을 덧붙이지 말 것**. 계속 진행이 필요하면 별도의 새 user 메시지로 보낸다. 빈 응답을 수정 없이 그대로 재시도하지 않는다.

## 4. 스트리밍 (SSE)

`"stream": true`로 요청하면 server-sent events로 점진적 응답을 받는다.

### 이벤트 흐름

```
message_start
  → content_block_start / content_block_delta* / content_block_stop   (블록마다)
  → message_delta*
  → message_stop
(중간에 ping, error 이벤트가 섞일 수 있음)
```

| 이벤트 | 내용 |
| --- | --- |
| `message_start` | `content`가 빈 `Message` 객체 |
| `content_block_start` | 블록 시작 (`index` 포함) |
| `content_block_delta` | 블록 내용 증분 |
| `content_block_stop` | 블록 종료 |
| `message_delta` | 최종 Message의 최상위 변경(`stop_reason`, `usage`). **usage는 누적값** |
| `message_stop` | 스트림 종료 |
| `ping` | 임의 개수로 섞여 들어옴 |
| `error` | 스트림 도중 오류 (예: `overloaded_error`) |

### delta 타입

| delta | 대상 | 비고 |
| --- | --- | --- |
| `text_delta` | `text` 블록 | `delta.text`를 이어붙임 |
| `input_json_delta` | `tool_use` 블록 | **부분 JSON 문자열**. `content_block_stop` 후에 파싱 |
| `thinking_delta` | `thinking` 블록 | 사고 내용 |
| `signature_delta` | `thinking` 블록 | `content_block_stop` 직전 1회. 사고 블록 무결성 검증용 |

> `tool_use.input`의 최종 형태는 **객체**지만 델타는 **부분 JSON 문자열**이다. 현재 모델은 키·값 한 쌍 단위로 내보내므로 이벤트 사이에 지연이 생길 수 있다. `eager_input_streaming`으로 세분화된 스트리밍을 켤 수 있다.

### 스트리밍을 써야 하는 경우
- **10분을 넘길 수 있는 긴 요청** — SDK는 비스트리밍 요청이 10분 타임아웃을 넘을 것 같으면 검증에서 막는다
- 큰 `max_tokens`를 비스트리밍으로 보내면 유휴 연결이 끊겨 실패할 수 있다
- 이벤트를 직접 처리할 필요가 없으면 SDK의 `stream.get_final_message()` / `finalMessage()`로 완전한 `Message`를 받을 수 있다

### 오류 복구
- **Claude 4.5 이하**: 받은 부분 응답을 **어시스턴트 메시지 앞부분으로** 넣고 이어서 요청
- **Claude 4.6 이상**: 프리필이 안 되므로, 부분 응답과 "여기서부터 계속하라"는 **사용자 메시지**로 이어서 요청
- `tool_use`·`thinking` 블록은 부분 복구가 불가능하다. 가장 최근 텍스트 블록부터 재개

## 5. HTTP 오류

| 코드 | error type | 의미 |
| --- | --- | --- |
| 400 | `invalid_request_error` | 요청 형식/내용 문제, 직접 설정한 지출 한도 도달 |
| 401 | `authentication_error` | API 키 문제(형식 오류·취소·만료) |
| 402 | `billing_error` | 청구·결제 정보 문제 |
| 403 | `permission_error` | 해당 리소스 사용 권한 없음 |
| 404 | `not_found_error` | 리소스 없음 |
| 409 | `conflict_error` | 리소스 상태 충돌 |
| 413 | `request_too_large` | 요청 크기 초과 |
| 429 | `rate_limit_error` | 속도 제한 / 티어 월간 지출 상한 도달 |
| 500 | `api_error` | 내부 오류 → **지수 백오프 재시도** |
| 504 | `timeout_error` | 처리 시간 초과 → 스트리밍/배치 고려 |
| 529 | `overloaded_error` | API 일시 과부하 |

### 요청 크기 제한
| 엔드포인트 | 최대 |
| --- | --- |
| Messages API | **32 MB** |
| Token Counting API | 32 MB |
| Batch API | **256 MB** |
| Files API | **500 MB** |

### 오류 응답 형태

```json
{
  "type": "error",
  "error": { "type": "not_found_error", "message": "..." },
  "request_id": "req_011CSHoEeqs5C35K2UUqR7Fy"
}
```

- 모든 응답에 **`request-id` 헤더**가 있고, 오류 본문의 `request_id`와 같은 값이다. 지원 문의 시 필수
- Python/TypeScript SDK는 `_request_id` 속성으로 노출
- 공식 SDK는 연결 오류·429·5xx를 **지수 백오프로 기본 2회 자동 재시도**하고 `retry-after` 헤더를 따른다. 클라이언트 옵션으로 조정·비활성화 가능
- SDK는 타입 지정 예외를 던진다(`anthropic.NotFoundError` 등). **문자열 매칭 대신 예외 클래스로 분기**하고 구체적인 클래스를 먼저 처리
- 스트리밍은 HTTP 200 이후 오류가 날 수 있으므로 표준 오류 처리 경로를 타지 않는다 → `error` 이벤트로 처리

### 자주 만나는 400 검증 오류
| 오류 | 원인 |
| --- | --- |
| 프리필 미지원 | Claude 4.6 이후 모델에 어시스턴트 프리필 전송 |
| thinking 블록 수정 불가 | 마지막 어시스턴트 메시지의 `thinking`/`redacted_thinking` 블록을 편집·재정렬·필터링해 재전송 |
| `thinking.type.enabled` 미지원 | 4.7 이후 모델 → `adaptive` + `output_config.effort` 사용 |
| `thinking.type.adaptive` 미지원 | 4.5 이하 모델 → `enabled` + `budget_tokens` 사용 |
| `thinking.type.disabled` 미지원 | Fable/Mythos 5.x, Opus 5.5 등 thinking 상시 on 모델 |
| 강제 도구 사용 미지원 | Opus 5.5, Fable 5.1, Mythos 5.1에 `tool_choice: any`/`tool` 전송 |

## 6. 속도 제한

세 축으로 측정된다: **RPM**(분당 요청), **ITPM**(분당 입력 토큰), **OTPM**(분당 출력 토큰). 하나라도 초과하면 429 + `retry-after`.

| 티어 (Opus 5.5 / Sonnet 5 기준) | RPM | ITPM | OTPM |
| --- | --- | --- | --- |
| Start | 1,000 | 2,000,000 | 400,000 |
| Build | 5,000 | 5,000,000 | 1,000,000 |
| Scale | 10,000 | 10,000,000 | 2,000,000 |

### 핵심 규칙
- **토큰 버킷 알고리즘**: 고정 간격 리셋이 아니라 **지속적으로 보충**된다. 분당 60 RPM이 초당 1건으로 적용될 수 있어 버스트는 429를 유발
- **캐시 인식 ITPM**: `input_tokens` + `cache_creation_input_tokens`만 ITPM에 포함되고 **`cache_read_input_tokens`는 대부분의 모델에서 제외**된다 → 캐싱이 실질 처리량을 크게 늘린다
- **OTPM은 실제 생성된 토큰만** 계산한다. `max_tokens`를 크게 잡아도 불이익이 없다
- 속도 제한은 **모델별로 별도** 적용된다(단, Opus 4.x 계열, Sonnet 4.x 계열, Fable 5.x 계열은 합산 버킷)
- Batch API, Files API, Managed Agents, Fast mode는 **각각 별도의 제한**을 가진다
- 조직 사용량이 급증하면 **가속 제한(acceleration limits)** 때문에 429가 날 수 있다 → 트래픽을 점진적으로 늘릴 것

### 응답 헤더
`retry-after`, `anthropic-ratelimit-requests-{limit,remaining,reset}`, `anthropic-ratelimit-input-tokens-*`, `anthropic-ratelimit-output-tokens-*`, `anthropic-ratelimit-tokens-*`(가장 제한적인 제한 값), `anthropic-workspace-id`

## 7. 긴 요청 처리

- 10분 초과가 예상되면 **스트리밍 또는 Message Batches API**를 쓴다
- Batch API는 연결 유지 없이 **결과를 폴링**할 수 있어 네트워크 리스크를 줄인다
- 직접 통합 시 **TCP keep-alive** 설정 권장

## 연결 노트

- [[03 애플리케이션 설계와 통합]]
- [[01 모델 선택과 최적화]]
- [[09 평가 테스트 디버깅]]
