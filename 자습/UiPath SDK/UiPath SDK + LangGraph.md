
---
# 개발환경 구성
Python 3.11 ~ 3.13 지원(공식)
uv 사용 권장
사용자 환경: Windows

### 가상환경 설정

```powershell
uv venv --python 3.13

.venv\Scripts\activate

uv pip install uipath-langchain
```

### UiPath CLI 설치
공식 설치 스크립트:

```powershell
irm https://download.uipath.com/uipath-cli/install.ps1 | iex
```

Node.js 22+, UiPath CLI, coding-agent skills, .NET SDK 8, Python 등을 함께 준비

```powershell
uip --version
uip --help
```

설치 결과 확인

### Coded Agent Tool 설치

```powershell
uip tools install @uipath/codedagent-tool
```

Result가 Success이면 정상 처리 완료

```powershell
uip codedagent --help
```

설치 결과 확인

### UiPath 구성
UiPath 계정에 로그인

```powershell
uip login
```

브라우저가 열리면 Automation Cloud에 로그인하고 tenant를 선택

```powershell
uip login status
```

상태 확인(Result: Success)

**현재 설정을 그대로 유지**

```powershell
uip codedagent setup --force
```

