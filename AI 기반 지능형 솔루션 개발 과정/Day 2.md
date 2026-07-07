#KDT 

---
**Excel Process Scope**
많은 엑셀 작업이 필요한 경우 스코프 안에 넣어줌
20만셀 이상 3-4개 파일 작업 필요시 권장

**Filter**
컬럼 하나당 한 개의 필터만 적용 가능
복수 적용하려면 추가로 필터 만들어줘야 함

기존 필터 지우기로 필터 액티비티 만들어서 기존 필터 지우고 새롭게 적용

**Sort Range**
열 정렬 기능

**Check App State**
특정 애플리케이션의 상태에 따라 하위 액티비티 실행
팝업이 떴을 때 닫기 처리 등에 사용
(타겟이 생겼을 때, 사라졌을 때로 나누어 처리)

### DataTable 추출 시 유의사항
CV 활용이 활성화된 상태로 한글 요소를 선택하면 한글이 깨질 수 있음
이 경우 CV를 꺼주고 text 요소를 가져오도록 타겟을 선택하면 됨

**Write DataTable to Excel**
데이터테이블 추출 후 데이터테이블 -> 엑셀로 옮기기

**Rename File**
파일 이름 재설정

파일 이름 변경과 같은 명령은 반드시 저장이 된 상태에서 실행되어야 하므로
명령 이전에 창 닫기, 저장하기와 같은 액티비티가 필요

-> Window operation 에서 Close Window + Save Excel File 처리

**Select Item**
목록에서 선택할 때 사용하는 액티비티
Type Into로 하면 너무 느려서 Select Item으로 해주는 게 좋음

**Navigate Browser**
뒤로/앞으로/홈으로 이동, 새로고침, 탭 닫기 처리





