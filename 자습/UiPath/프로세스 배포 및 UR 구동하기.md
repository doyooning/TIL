
---
## 머신 생성
Orchestrator → Tenant → Machines → 추가 → 표준 머신
**Unattended 슬롯 1개**를 배정

## 머신 키로 로그인
만든 머신의 "Machine key"를 복사
UiPath Assistant 로그아웃
Cloud 말고 Assistant → 설정 → Orchestrator 연결에서
서비스 URL(`https://cloud.uipath.com/dyniiverse/DefaultTenant/orchestrator_`)과 머신 키를 입력

## UR 설정
사용자 → 자동화 사용자 설정에서 Windows 계정(자격 증명)을 지정
Orchestrator Database로 되어 있고 비밀번호를 현 Windows 비밀번호를 입력

**폴더에 머신 할당:** `Shared` 폴더 → Machines에서 `dynii-pc`(머신 이름)를 추가
로봇 PC는 Chrome과 UiPath 브라우저 확장이 있어야 함
Windows 계정으로 로그인된(잠기지 않은) 세션이어야 하고, 실행 중에는 Chrome 창이 다른 창에 덮이지 않아야 함

