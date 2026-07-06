
---
# UiPath
ks 아카데미 - 자료실 25.10.4 버전
마에스트로 기능 다뤄볼 것
UiPath Studio Cloud

# RPA
UiPath가 가장 대중적, 근데 비쌈

다른 RPA로는 MS의 PowerAutomate 한번 찍먹
비용적인 부분에서 대안책

기존의 프로세스를 자동화하는 것이기 때문에 개발 시간이 짧음

RPA는 항상 개발자가 1명... 팀 협업과 거리가 멀다

On-Premise 버전은 로컬에서 작동하는 느낌이고
Cloud 버전은 로그인 과정이 있음

RE Framework(Robotic Enterprise)를 가장 많이 사용
따로 포트폴리오 만들거면 이거로 열심히 만들기

# 시작하기
프로젝트 설명에는 기능, 누가 언제 개발했는지 + 수정했는지를 보통 기입

UiPath에서 제공하는 익스텐션에 한해 업무 자동화 가능
ex) Java 프로젝트에서 자동화 적용 시 Java 익스텐션 설치
미제공하는 분야는 적용이 매우매우 힘들다

### UiPath 자동화 플랫폼 세부 기능:
1. UiPath Studio(개발 도구)
2. UiPath Assistant(실행 보조 도구)
3. UR(Unattended Robot, 무인형 / 비협업로봇)
   사람 개입 없이 작업
4. AR(Attended Robot, 유인형 / 협업로봇)
   사용자와 동일 PC에서 작업, 사용자가 개입

**플로우차트**
복잡한 관계가 구성될 때는 플로우차트 그리기

**시퀀스**
여러 가지 액티비티를 하나의 묶음 단위로 유지
그룹화같은 기능임

**Use Excel File**
엑셀 파일 참조하기
참조할 파일 입력

**For Each Excel Row**
엑셀 파일의 한 행마다 실행
반드시 Use Excel File 내부 실행에 넣어주어야 함

CurrentRow는 한 행(데이터 1개 단위)
'범위 내' 속성은 엑셀 시트를 지정

엑셀에서 해당 범위를 드래그하여 범위 지정
헤더가 포함되어있지 않으면 헤더 포함에 체크 해제

하나의 컬럼을 Item이라고 부름
현재 행의 컬럼을 만들어 주려면
```
CurrentRow.Item("사업자번호")
CurrentRow.ByField("사업자번호")
```

**Write Line**
로그찍기와 동일한 기능
텍스트를 출력함

