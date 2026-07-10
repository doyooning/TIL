#KDT 

---
# UiPath에서 API 활용
### OpenAI Whisper 사용하기
변수에 API Key 할당

**HTTP Request(legacy)**
레거시이지만 레거시 아님 자주 사용함
속성의 OAuth2 토큰에 API Key 넣어줌
(가끔 구성 변경하면 없어지는데 잘 들어가 있는지 확인 필요)

시간 제한은 15000ms 정도로 설정

==구성==
엔드포인트: api 요청 주소, "https://api.openai.com/v1/audio/transcriptions"
요청 메서드: POST
파라미터:
model : "whisper-1" (가장 저렴한 모델)
response_format : "text"
language : "ko"
첨부파일:
file : 음성파일경로

### 파일 경로명 불러오는 반복문
**For Each File In Folder**
폴더 내 파일 각각을 불러옴

필터링 기준에 와일드카드를 적용할 수 있음
`"*샘플*"` : 샘플이라는 문자열을 포함한 모든 파일
`"*.xlsx"` : 모든 엑셀 파일
`*` : 글자 개수나 존재 여부
`!` : 글자 1개당 느낌표 1개

`CurrentFile.FullName` : 파일 절대주소까지 전부
`CurrentFile.Name` : 파일명 + 확장자

# 예외 처리
### Retry Scope
재시도 대상 로직을 넣어줌
Condition에는 완료 상태를 확인할 수 있는 액티비티를 넣어줌

### try catch
오류를 캐치캐치
System.Exception 등 다양한 예외에 대한 처리 가능

# JSON 다루기
RPA에서는 JSON -> DataTable을 많이 씀
여기서 문제는 JSON의 기본 형태는
```json
{
	"이름": "홍길동",
	"나이": "28",
	"활성": true
}
```

이런 형태이나, 데이터 테이블은 반드시 배열 `[]`로 감싸야 함

그렇기 때문에 JSON 앞뒤에 대괄호를 붙여서 배열로 감싸줌
`String.Format("[{0}]", str_json)` 이런 식으로 포맷을 지정

### 객체 안에 배열이 또 있는 경우
```json
{
	"결과": [
		{...},
		{...}
	]
}
```

이런 구조라면, Deserialize JSON으로 JObject로 변환한 후 
`JObj("결과").ToString` 으로 따로 빼내야 함


