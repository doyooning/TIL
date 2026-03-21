#AI  

---
# Skills 사용
스킬 공식 레포지토리
https://github.com/openai/skills

skill-installer : 큐레이트된 skill 리스트를 보여줌


### Document Skill
pdf - pdf 파일을 구조화된 데이터로 읽고 쓴다.
doc - doc 파일을 구조화된 데이터로 읽고 쓴다.
spreadsheet - 엑셀 파일을 구조화된 데이터로 읽고 쓴다.

```
// 예시
$pdf sn74hc595.pdf 를 분석해서 $doc docs파일에 한글로 요약정리해줘
$spreadsheet 유형검사.xlsx 읽고 점수 계산하는 로직과 점수에 따른 유형 $doc 에 정리해줘
```


### Notion Skill
notion-knowledge-capture
: 대화나 문서에서 핵심 인사이트를 추출해 Notion에 구조화하여 작성한다.
notion-research-documentation
: 조사 결과를 정리해 Notion에 구조화하여 작성한다.
notion-meeting-notes
: 내용을 논의/결정/액션 아이템 기준으로 정리해 Notion 회의록 템플릿으로 구조화하여 작성한다.

외부 시스템에 접근하면 필요한 MCP 연결을 자동으로 설정함
재시작 필요하다고 하면 codex resume --last로 다시 들어와서 진행

# 기타 Codex 사용법
실패 요인 해결 예시

- 필요한 환경 사용할 수 있도록 세팅 pip, node, apt-get, ...
- 네트워크나 접근 관련 권한 문제라면 "권한 올려서 진행해 줘"
- root가 필요하다면 "root 비밀번호 xxxx야 sudo apt-get ~~ 진행해 줘"

반복적인 실패 요인이 있다면, AGENTS.md 에 명시해 두는 것도 좋다.

작업 중간에 끼어들어 말할 때는 "Ctrl + c"로 중단하고 말하면 된다.



---
출처: https://cornpip.tistory.com/158