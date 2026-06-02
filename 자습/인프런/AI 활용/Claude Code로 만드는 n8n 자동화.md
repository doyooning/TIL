#AI 
#자동화
#n8n

---
# Claude Code x n8n 자동화 구조
![[Drawing 2026-06-02 10.50.44.excalidraw.png]]
워크플로우 JSON 핵심 구조
nodes: 노드 배열(이름, 타입, 설정)
connections: 노드 간 연결 라우팅
settings: 워크플로우 메타데이터

# Docker로 n8n 설치하기
Claude Code에 입력:
```
"Docker로 n8n 서버를 로컬에 설치해줘"
```

Claude Code가 생성하는 명령어:
```bash
docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n n8nio/n8n
```

`-d` → 백그라운드 실행 (터미널 닫아도 서버 유지) 
`-p` 5678:5678 → 내 컴퓨터 5678 포트 ↔ n8n 5678 포트 연결 
`-v` n8n_data:... → 데이터 영속 저장 (컨테이너 삭제해도 유지)

준비물: docker-compose
# n8n 웹 UI 접속 + 초기 설정
`localhost:5678`에서 바로 확인
로그인창 로그인
API Key 발급

# n8nMCP 서버 연결

