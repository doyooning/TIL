
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

감정 분석
전반적인 감정: 수치화
감정 분석: 부정, 중립, 긍정으로 표현
신뢰 수준: 정확도와 관련






