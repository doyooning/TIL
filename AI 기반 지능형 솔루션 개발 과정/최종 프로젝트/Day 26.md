#KDT 

---
# 검색 기능 설계
LLM은 RPA까지 연결해본다

### 화면 구성
FastAPI에서 검색하는 기능을 만들어놓고 연결
간단하게 가운데에 자연어 입력하는 검색바 만들기?


### 로직 설계

`llama-server + Gemma → Structured Query → FastAPI Validation → MongoDB`


```
LLM의 책임
자연어 이해
       ↓
검색 의도 + 조건 추출
       ↓
JSON 생성
       ↓
끝

FastAPI의 책임
JSON 검증
       ↓
권한 검사
       ↓
MongoDB Query 생성
       ↓
DB 조회
       ↓
결과 반환
```


# LLM과 업무 자동화
```
사용자
  │
  │ "오늘 결석한 학생 목록 확인해서
  │  담당자에게 보고서 보내줘"
  ▼
FastAPI / Chat UI
  │
  ▼
n8n
  │
  ├── LLM(Gemma / llama-server)
  │      └─ 의도 분석
  │      └─ 필요한 Tool 선택
  │      └─ 파라미터 생성
  │
  ├── FastAPI 조회 Tool
  │      └─ 학생 상태 조회
  │
  ├── 보고서 생성 Workflow
  │
  ├── 메일/Slack 등 전달 Workflow
  │
  └── 필요 시 관리자 승인
```

|구성요소|책임|
|---|---|
|**Gemma**|사용자의 자연어 의도 해석|
|**llama-server**|Gemma 추론 API 제공|
|**n8n**|업무 순서 결정 및 Workflow 실행|
|**FastAPI**|프로젝트의 실제 비즈니스 API|
|**MongoDB**|학생/상태/출결 데이터|
|**RPA**|외부 서비스에서 실제 작업 수행|
**n8n이 MongoDB에 직접 들어가는 구조보다는 FastAPI API를 Tool로 제공하는 구조**를 권합니다.

```
O

n8n AI Agent
   │
   ├─ get_absent_students()
   ├─ get_student_status()
   ├─ create_attendance_report()
   └─ request_notification()
          │
          ▼
       FastAPI
          │
          ▼
       MongoDB
```

```
get_student_status
    student_id
    date

get_absent_students
    class_id
    date
    period

get_wrong_seat_students
    class_id
    date
    period

create_attendance_report
    class_id
    date

send_attendance_report
    report_id
    recipient
```


