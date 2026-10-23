
---
# Session 1
### Integration Service
RPA의 장점은 외부 시스템 접근이 불가한 부분을 자동화하기 용이함
Integration Service의 장점은 API 기반으로 커넥터 제공

RPA 방식은 UI 자동화, UI가 바뀌면 셀렉터 다시 잡아주고 해야함
대량 데이터 처리할 때 느림

Gmail 읽기나 GenAI 사용 등을 연결해서 사용 가능

### Web Studio
Web Studio로 연결해서 웹에서 워크플로 생성 가능
간단한 RPA 워크플로 만들고 배포(Gmail 받은메일함에 오늘날짜를 포함하는 메일을 트리거로 작동)
배포 성공 로그가 뜨면 오케스트레이터 접속해서 해당 워크스페이스 -> 자동화 -> 프로세스 -> 새 프로세스

Connection을 만들 때, 다음 내용 확인:
- 작업 폴더에 만들었는지, 작업 폴더 내 하위 디렉토리에 만들었는지
  -> 로봇 작업 권한 차이때문에 만든 위치에 따라 연결이 되는지 여부가 다름
  ex) My Workspace에 연결 생성 != My Workspace > Solution에 연결 생성

### Connector Builder
커넥터를 직접 만들어줄 수 있음
API 정의에서 시작은 Swagger나 Postman에서 가져올 수 있음
==수업에서 안 다룸, 나중에 직접 해보기==


# Session 2
### GenAI Activities

**Storage Bucket**
클라우드 저장 공간 AWS S3 느낌(연동 됨)
워크플로에 가져올 땐 Download Storage Bucket 액티비티로 가져옴

**Image Analysis**
이미지를 프롬프트에 맞게 분석
(사진을 보고 ~ 를 판단)

**Sentiment Analysis**
전반적인 감정: 수치화
감정 분석: 부정, 중립, 긍정으로 표현
신뢰 수준: 정확도와 관련


# Session 3
### RPA와 Agent
대체 존재가 아니라 상호 보완
반복적, 정확한 처리 = RPA
문맥 이해, 판단 = Agent

Agent는 목적 기반:
시작 -> 툴 활용 -> 에스컬레이션(HITL) -> 정제 -> 끝
정제 후 맘에 안 들면 그걸 컨텍스트화 -> 계획 -> 다시 툴 활용

### Agent Builder
자율형
대화형
코딩됨

자율형은 사용자 프롬프트를 입력해서 해당 기준에 부합하는 결과를 얻을 수 있고,
대화형은 사용자 프롬프트가 없음, 시스템 프롬프트만 해놓고 채팅 방식으로 계속 대화

**Guardrail 설정**
결과를 필터링하거나 특정 결과는 제외하고 싶을 때 사용

Guardrail > Prompt Injection으로 결과물 검증 가능
ex) 폰 추천에서 삼성 폰을 추천해주지 마라 -> 삼성 폰을 추천할 경우 가드레일에 의해 프롬프트 인젝션으로 판단, 실행을 어떻게 처리할지는 따로 설정해줌(Info, Warn, Error - 에러는 실행 멈춤)

**피드백**
Agent 결과물에 대해 피드백(좋아요, 싫어요)을 남기면 성능 개선에 도움이 됨

**실습**
Agent를 하나 만들고, Agent가 사용할 도구를 RPA 워크플로로 만든 다음 가져다 쓰도록 연결함
이 때, 시스템 프롬프트로 도구 호출 전 에스컬레이션을 진행하도록 설정
결과 확인: 에스컬레이션 제기로 오류를 뱉음
원인: 항상 에스컬레이션을 제기하라고 해서 하긴 했는데 에스컬레이션 설정이 안 되어 있으니 오류로 처리

에스컬레이션에 액션 앱 지정,
수신자 입력(자기자신)하고 다시 진행
실행 중에 액션 태스크가 생성됨(링크) -> Action Center로 연결되어 앱을 열고 승인/거절 진행

### Apps
UiPath Apps 기능
액션과 페이지와 같은 UI를 만들어 놓을 수 있음
-> 에스컬레이션 적용시, Agent가 하게 될 일과 예상 결과값 등을 보여주고 승인/거절 버튼을 만들 수 있음
==앱스 구성 방법은 교재 참조==


# Session 4
### BPMN & DMN
BPM = 비즈니스 프로세스 관리
비즈니스 프로세스를 효율적으로 개선하기 위한 체계적인 관리 방법론
업무 처리에 대한 명확한 절차와 규칙이 필요하다
PDD 등 개발 문서들도 이러한 절차와 규칙에 해당함

최대한 간결하고 정확하게
텍스트보다는 시각적으로 정형화된 기호로 가독성을 높여야 함
BPMN = Business Process Model and Notation

일반적인 순서도(Flowchart)

프로세스 구성 주체를 구분하기 위한 풀(Pool)을 사용
업무 주체 또는 역할을 레인(Lane)이라 함

Exclusive = 참/거짓(단일 분기)
Parallel = AND 게이트웨이
Inclusive = OR 게이트웨이

Business Rule Task = DMN

![[Pasted image 20261022152757.png]]

**DMN**
주관적인 판단의 차이는 비즈니스 환경에서 큰 리스크임
사람이 내리는 의사결정을 명시적으로 모델링, 자동화 가능한 방식으로 표현

보여주는 건 BPMN
규칙은 DMN

Decision = 가장 메인, 결정
Input Data = 입력 값
Knowledge Source = 금액 정책 등
Knowledge Model = 계산 공식 등 

---
### 활용 - 송장 검증 프로세스

**비즈니스 프로세스 모델링**
BPMN 만들기
각 Task에 수행 주체(로봇, 에이전트, 사람, Integration Service, Data Fabric) 주석으로 서술
모델링 다운받고 Cloud Studio에서 Maestro BPMN 추가
다운받은 .bpmn 업로드
로봇 Task에는 RPA 프로세스 연결,
에이전트 Task에는 새 에이전트 추가해서 연결

>JSON 타입에서의 Description 명시가 프롬프트보다 더 우선시되는 경향
ex) Description에서 A 써놓고 프롬프트에서 B로 써놓으면 A로 나옴

### Data Fabric
DB같은 느낌으로 사용함 데이터 저장, 불러오기
Integration Service로 테넌트나 폴더가 연결되어야 함
엔티티를 만들어서 필드 구성 및 속성 지정


# Session 5
### Maestro Case
Case는 그때그때마다 필요한 업무를 처리함
정해진 작업 방식과 같은 형식이 없음
1. 목표 중심 관리
2. 장기 실행 지원
3. 비선형 업무 진행
4. 다중 실행 주체 조정
5. 데이터 및 이력 유지
6. SLA 및 에스컬레이션 관리
7. 감사 추적성 제공

구성 요소:
Case(Case Key)
Entity, Stage, Case Manager
Rule, Task
SLA & Escalation
Case Persona
Case App

Case
특정 업무 상황의 데이터, 규칙, 작업 이력을 한눈에 볼 수 있음
상당히 동적인 플로우를 구성할 수 있음

Case Key
Case로 여러가지 할 때 필요

Stage
주요 업무 단계
==Primary Stage: 일반적인 진행 경로==
Secondary Stage: 예외가 있는 경로

Task
Stage 안에서 수행되는 내부 개별 업무

Rule
Case 이벤트와 데이터를 평가, 최종 상태를 결정하는 실행 규칙

SLA
Service Level Agreement
Case 또는 Stage를 정해진 목표 시간 안에 처리 ~ 업무를 언제까지 처리해야 하는지 정의하는 시간 기준

### 활용 - 고객 문의 분류 프로세스
Maestro Case 구성
: 접수 -> 분류 -> 배정 -> 완료

접수: 문의사항을 다듬어주는(정규화) Agent
분류: 자동으로 카테고리 분류 Agent, 우선도를 판단해주는 Agent, 요약해주는 Agent
배정: 요청을 적합한 부서에 할당해주는 Agent

자동으로 분류했을 때 confidence가 낮을 경우, 수동 분류로 이동
수동 분류 필드가 채워진 경우 수동 분류를 적용

진입 규칙, 완료 규칙, 종료 규칙을 설정

# Session 6
### Maestro Flow
개발자 친화적인 플로우를 구성하기 용이함
실행이랑 테스트 등등 강점

나온지 2개월정도 된 따끈따끈한 기능
아직 데모 프로세스도 많이 없다
기능들도 계속 바뀌고 있어서 안정화가 안 된듯

