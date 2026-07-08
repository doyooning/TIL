#UiPath

---
# 디버깅 팁
### 로깅
Write Line 자주 사용
반복문 도는 중에 몇 번째 반복중인지 등을 확인하고 싶을 때,

For each ... 실행 내부에서
Assign 액티비티 생성
`counter = counter + 1` 선언,
Write Line으로 `counter` 출력

### try catch
503 에러 등 불가항력적인 자연재해를 만났을 때
재시도(retry) 등으로 생명 연장 가능


