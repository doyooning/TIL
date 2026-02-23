#Grafana
#Prometheus

---
# Grafana
시각화 대시보드 툴

# Prometheus
시계열(Time Series) 데이터 수집 및 저장에 특화된 오픈소스 모니터링 시스템
K8s, Docker, Spring Boot 등 다양한 환경과 쉽게 연동할 수 있다는 장점
매트릭 수집에 최적화된 구조

|                     |                                                                 |
| ------------------- | --------------------------------------------------------------- |
| 특징                  | 내용                                                              |
| **시계열 저장**          | 시간(Time)과 메트릭 값(Value) 쌍으로 저장되며, 라벨 기반 필터링이 가능함                 |
| **Pull 방식**         | Prometheus 서버가 각 서비스(Exporter)로부터 **직접 메트릭을 주기적으로 가져옴(scrape)** |
| **PromQL 지원**       | 강력한 쿼리 언어(PromQL)를 통해 복잡한 조건의 메트릭 계산, 필터링 가능                    |
| **Exporter 기반**     | 다양한 시스템(Spring, Redis, Node 등)에 메트릭 수집을 위한 Exporter가 존재         |
| **Grafana와 연동**     | Grafana를 통해 Prometheus 데이터를 시각화할 수 있음                           |
| **Alertmanager 연동** | 조건부 경고(예: CPU > 90%) 발생 시 이메일, 슬랙 등으로 알림 전송 가능                  |

# 모니터링 흐름
Spring Boot App → Actuator (/prometheus) → Prometheus → Grafana

#### 1. Spring Micrometer
- Spring Boot의 메트릭 수집을 도와주는 내장 라이브러리
- JVM, HTTP 요청, GC, 메모리, I/O 등의 지표를 수집
- 내부적으로 수집된 데이터를 MeterRegistry에 저장

#### 2. MeterRegistry
- Micrometer에서 수집한 메트릭 데이터를 보관하는 저장소 역할
- Prometheus, Datadog, CloudWatch 등의 외부 시스템에 맞게 변환/출력 가능

#### 3. Spring Boot Actuator
- /actuator/prometheus 경로를 통해 메트릭을 HTTP로 노출(외부에 노출할 때, 보안에 취약점이 발생할 수 있으므로 Security에서 접근 가능한 권한을 가진 사용자만 접근할 수 있도록 변경하거나 하는 등의 취약점을 보완해야 함.)
- Prometheus가 접근 가능한 엔드포인트 역할
- 실제 지표는 MeterRegistry에서 가져옴

#### 4. Prometheus
- prometheus는 Pull(필요할 때마다 수집) 방식으로 작동
- 설정한 주기에 따라 Spring boot 서버의 /actuator/prometheus를 호출하여 메트릭 정보 수집
- 수집한 데이터(Time-Series)는 자체 시계열 DB(Prometheus TSDB(Time Series Database))에 저장  
    이 때, 저장 구조는 (time series = metric name + labels + timestamp + value) 형식 -> chunk라는 단위로 압축 저장

#### 5. Grafana
- Prometheus를 데이터 소스로 연결
- 사용자는 PromQL을 사용하여 다양한 그래프와 지표를 조회하고 시각화 함  
    (Prometheus Query Language : 시계열 데이터를 조회하고 분석하기 위해 사용하는 전용 쿼리 언어)
- 대시보드, 경고(Alert) 설정도 가능


# 적용하기
의존성 추가
```yml
implementation 'org.springframework.boot:spring-boot-starter-actuator'
implementation 'io.micrometer:micrometer-registry-prometheus'
```

application.yml에 설정 추가
```yml
management:
  endpoints:
    web:
      exposure:
        include: health, beans, httptrace, info, metrics, prometheus
```

다양한 엔드포인트들이 존재
필요한 엔드포인트 확인 후 include에 기입

**자주 사용하는 Acutator 엔드포인트**

|   |   |
|---|---|
|**엔드포인트**|**설명**|
|health|시스템 상태 확인 (기본 노출)|
|info|application.yml에 설정한 정보 노출 (info.*)|
|metrics|메트릭 조회 (예: JVM, 메모리, HTTP 요청 수 등)|
|prometheus|Prometheus가 수집할 수 있는 메트릭 포맷으로 노출|
|env|애플리케이션 환경 변수 정보 (Environment)|
|beans|등록된 Spring Bean 목록|
|mappings|요청 URI → 핸들러 매핑 정보|
|threaddump|JVM 쓰레드 덤프|
|loggers|로깅 설정 동적 조회 및 변경|
|httptrace|최근 HTTP 요청-응답 목록|
|heapdump|힙 덤프 다운로드 (주의: 리소스 사용 많음)|
|caches|Spring Cache 상태 및 정보|
|conditions|자동 구성 조건 및 평가 결과|
|scheduledtasks|예약된 스케줄러 목록|
|shutdown|애플리케이션 종료 요청 (보안상 기본 비활성)|

SecurityConfig 수정
```java
        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/actuator/**").permitAll()
                        .anyRequest().denyAll());
```

테스트를 위해 permitAll()로 설정
실제 프로젝트에서는 해당 정보들이 모두에게 보이면 보안에 취약점이 발생할 수 있으므로 
허용한 네트워크에서만 접근 가능하도록 하거나 `hasRole()`을 설정해 인가된 사용자만 접근 가능하도록 설정해야 함

