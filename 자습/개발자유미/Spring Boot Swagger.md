#Java 
#Spring 

---
https://cafe.naver.com/xxxjjhhh/94
개발자 유미 강의 참조


# API 명세
**엔드포인트 공개**
서버에 존재하는 자원은 API를 통해 End User 또는 다른 개발자들에게 제공되는데
이런 API를 사용하기 위해선 “기능, 요청 방법 (세부), 응답 형식”을 파악해야 함

따라서 개발한 API를 잘 활용하도록 API 명세서 (스펙)를 작성해야 함

**API 명세서 획일화**
하지만 전세계에는 수많은 API가 존재하고 ..
각각의 개발자들이 자신만의 형태로 명세서를 작성하게 된다면 
새로운 API를 사용할 때 마다 명세서의 형태를 파악해야 하는 어려움이 있음

이 문제를 해결하기 위해 국제적으로 API 명세 표준을 몇 가지 만들어 두었음

- OpenAPI (OAS)
- RAML
- API Blueprint

이 중 가장 많이 사용되는 명세서 작성 형식은 OpenAPI이며 내부적으로 JSON/YAML 형태의 포맷이 있음

**OAS**
OpenAPI는 버전이 존재
현재 3.0 이라 불리는 3.X 번대의 버전이 stable하고 많이 사용됨

**OAS를 기반으로 시각화**
작성한 OAS를 Swagger UI에 넣는다면 흔히 Swagger라 부르는 시각화를 진행할 수 있음

![[Pasted image 20260128120410.png]]

# 엔드포인트 OAS 생성
**의존성 추가**
스프링 부트 컨트롤러 기반으로 OpenAPI 명세서를 생성하기 위해 관련된 의존성을 추가하겠습니다.

build.gradle
```
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.8'
```
주로 사용되는 버전은 2.2.0, 2.3.0

**엔드포인트**

OAS
- JSON : /v3/api-docs
- YAML : /v3/api-docs.yaml

Swagger UI
- /swagger-ui/index.html

