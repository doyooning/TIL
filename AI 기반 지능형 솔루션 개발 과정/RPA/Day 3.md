#KDT 

---
# 생성형 AI 활용
프로젝트마다 패키지를 추가해주어야 함
패키지 추가 - 모든 패키지 - GenAI 검색 후 프로젝트에 추가 - 저장

최초 액티비티 실행시 연결 설정 필요
API 키가 필요한데 60일 무료 Trial은 아마 모든 모델 다 사용하게 만들어 놓은 듯

**Reformat**
입력받은 텍스트를 원하는 포맷으로 변환해줌
ex) 주문번호는 12345고... JSON으로 변환 -> {orderNumber: 12345, ...}

**Translate**
번역 기능

**PII Filtering**
개인정보와 같은 민감정보를 감지하여 평문으로 바꿔줌
근데 기능이 너무 별로임..
사람 이름이면 Person-4 이런 식으로 바꿔주는데 특수문자랑 숫자 들어가는 것부터 너무 밤티

**Detect Language**
언어 감지

### Content Generation
LLM을 이용하여 프롬프트대로 결과물을 출력해내는 기능
User Prompt, System Prompt 기반

컨텍스트 그라운딩 기능을 통해 RAG 활용 가능
(파일을 선택 가능, 50MB 제한 있음...)
결과 수 선택으로 Top-K도 가능

**Get Local File or Folder**
파일이나 폴더를 가져옴
파일 경로 지정시 파일 -> 속성 -> 보안 탭에서 그대로 긁어오는 것이 정확
파일 또는 폴더 '존재' 는 Boolean 타입, '개체' 는 ILocalResource 타입
존재 여부 파악과 개체 자체의 차이

### Entity 생성
JSON 형태로 출력하기 위한 Dictionary를 생성하기
Dictionary<String, String> 타입의 변수를 만들고,
Set Variable Value로 이 변수에 특정 값을 대입함

```VB
New Dictionary(Of String, String) From {
	{"key1", "value1"},
	{"key2", "value2"},
	...
}
```

생성형 AI로 해당 엔티티를 읽어서 분석 등을 하려면
`"비판불만", "급전개 등 작품에 대한 아쉬움과 비판"`
이런 식으로 설명을 value에 붙임

**Sentiment Analysis**
텍스트의 감정을 분석
전반적인 감성에 변수를 넣어 출력
출력값은 JSON, score와 label로 구성

**Categorize**
텍스트를 분석하여 카테고리화
카테고리 분류 방법을 담은 엔티티 필요

**Build Data Table**
데이터 테이블을 만들어줌

**Add Data Row**
행 단위로 데이터 테이블에 추가
DataRow 개체를 추가하거나, 배열을 만들어 추가할 수 있고 열 형식에 올바르게 매핑되어야 함
배열 행: `{변수1, 변수2, ...}`

### Data Table과 Excel
하나씩 엑셀 파일에 쓰는 것보다는 데이터 테이블을 만들어놓고 엑셀에 한번에 쓰는 것이 효율적임
자주 엑셀 파일을 읽고 쓰면 메모리에 부담

==!! 주의사항 !!==
데이터 테이블을 엑셀에 입력할 때,
Write DataTable to Excel 에서
대상: `Excel.Sheet(...).Range("E1")`, 추가: `true`
이런 식으로 되어 있으면 E1 스킵하고 E2부터 입력되는 문제가 있음
때문에 Range로 셀을 잡아서 입력할 때는 추가에 체크 해제하고 해야 함



