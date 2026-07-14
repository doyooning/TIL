#KDT 

---
# Orchestrator
UiPath Assistant 실행 후 설정에서 orchestrator connection
cloud.uipath.com으로 들어가서 진행

Shared 디렉토리에 asset과 connection을 추가하면 해당 조직에 소속된 사용자들이 자신의 프로젝트에 가져다가 사용할 수 있음

API키같은 민감정보는 asset에서 secret으로 관리
액티비티로 가져올 땐 'Get Secret' 사용
SecureString -> String 타입 변환 필요
`New System.Net.NetworkCredential("", apikey).Password`

