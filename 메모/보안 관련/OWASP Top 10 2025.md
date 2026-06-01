#OWASP
#보안

---
# OWASP
 : Open Web Application Security Project
 전 세계의 개발자, 보안 전문가, 기업, 교육 기관 등이 모여 웹 애플리케이션 보안을 개선하기 위해 활동하는 비영리 커뮤니티

**OWASP Top 10**  
웹 애플리케이션 보안 분야의 업계 표준으로 통용되는 가장 중요한 문서
업계 표준으로 인정받는 위협 리스트가 있다면, 개발자와 보안 전문가는 가장 시급하게 해결해야 할 보안 위험에 우선순위를 두고 집중할 수 있음
3~4년 주기로 업데이트, 2025 버전은 Release Candidate(RC) 버전으로 공개됨

2025버전: 
- [**A01:2025 – Broken Access Control**](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/)
- [**A02:2025 – Security Misconfiguration**](https://owasp.org/Top10/2025/A02_2025-Security_Misconfiguration/)
- [**A03:2025 – Software Supply Chain Failures**](https://owasp.org/Top10/2025/A03_2025-Software_Supply_Chain_Failures/)
- [**A04:2025 – Cryptographic Failures**](https://owasp.org/Top10/2025/A04_2025-Cryptographic_Failures/)
- [**A05:2025 – Injection**](https://owasp.org/Top10/2025/A05_2025-Injection/)
- [**A06:2025 – Insecure Design**](https://owasp.org/Top10/2025/A06_2025-Insecure_Design/)
- [**A07:2025 – Authentication Failures**](https://owasp.org/Top10/2025/A07_2025-Authentication_Failures/)
- [**A08:2025 – Software or Data Integrity Failures**](https://owasp.org/Top10/2025/A08_2025-Software_or_Data_Integrity_Failures/)
- [**A09:2025 – Logging & Alerting Failures**](https://owasp.org/Top10/2025/A09_2025-Logging_and_Alerting_Failures/)
- [**A10:2025 – Mishandling of Exceptional Conditions**](https://owasp.org/Top10/2025/A10_2025-Mishandling_of_Exceptional_Conditions/)

|          |                                                               |                                                                                                                                                                                   |
| -------- | ------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **코드**   | **명칭**                                                        | **설명**                                                                                                                                                                            |
| A01:2025 | 취약한 접근 통제  <br>(Broken Access Control)                        | 2021년에 이어 2025년에도 1위를 유지하며 여전히 가장 큰 위협으로 자리 잡고 있습니다.                                                                                                                              |
| A02:2025 | 보안 설정 오류  <br>(Security Misconfiguration)                     | 2021년 5위에서 2위로 크게 상승했습니다. 테스트 된 애플리케이션의 약 3.00%에서 발견되었으며, 애플리케이션 설정이 점점 더 복잡하고 커스터마이징 가능해지면서 더욱 만연한 위협이 되고 있습니다.                                                                  |
| A03:2025 | 소프트웨어 공급망 실패  <br>(Software Supply Chain Failures)            | 주요 신규 카테고리입니다. 2021년의 ‘취약하고 오래된 구성 요소(A06:2021)’를 확장한 개념으로, 소프트웨어 의존성, 빌드 시스템, 배포 인프라 전반의 위협을 포함합니다. 발생 빈도는 낮지만 한 번 발생하면 심각한 피해를 초래할 수 있으며, 커뮤니티 설문조사에서 압도적인 1순위 우려 사항으로 선정되었습니다. |
| A04:2025 | 암호화 실패  <br>(Cryptographic Failures)                          | 2위에서 4위로 두 단계 하락했으나, 여전히 민감 데이터 노출이나 시스템 침해로 이어질 수 있는 중요한 위협입니다.                                                                                                                  |
| A05:2025 | 인젝션  <br>(Injection)                                          | 3위에서 5위로 두 단계 하락했습니다. XSS 인젝션부터 SQL 인젝션까지 다양한 인젝션 관련 취약점을 포함합니다.                                                                                                                  |
| A06:2025 | 안전하지 않은 설계  <br>(Insecure Design)                             | 4위에서 6위로 두 단계 하락했습니다. 2021년에 처음 도입된 이후 위협 모델링 및 보안 설계에 대한 업계의 개선이 눈에 띄게 나타나고 있습니다.                                                                                                |
| A07:2025 | 인증 실패  <br>(Authentication Failures)                          | 7위를 유지했으며 명칭이 약간 개선되었습니다. 표준화된 인증 프레임워크 사용이 증가하면서 관련 취약점 발생률이 감소하는 긍정적인 효과를 보이고 있습니다.                                                                                             |
| A08:2025 | 소프트웨어 및 데이터 무결성 실패  <br>(Software or Data Integrity Failures) | 8위를 유지했습니다. 소프트웨어 공급망 실패보다 낮은 수준에서 신뢰 경계(Trust Boundaries) 유지 실패 및 소프트웨어, 코드, 데이터 아티팩트의 무결성 검증 부재에 중점을 둡니다.                                                                       |
| A09:2025 | 로깅 및 경고 실패  <br>(Logging & Alerting Failures)                 | 9위를 유지했으며 명칭이 업데이트 되었습니다(이전: 보안 로깅 및 모니터링 실패). 단순히 로깅만이 아니라, 로깅 이벤트에 대한 적절한 조치를 유도하는 경고(Alerting) 기능의 중요성을 강조합니다. 데이터상으로는 항상 과소평가되지만, 커뮤니티 투표를 통해 순위를 유지했습니다.                     |
| A10:2025 | 예외적 조건의 오처리  <br>(Mishandling of Exceptional Conditions)      | 2025년 새롭게 추가된 카테고리입니다. 부적절한 오류 처리, 논리 오류, 비정상적인 조건에서 발생하는 정보 노출 등 24가지 CWE를 포함합니다.                                                                                                |

출처 : https://www.pentasecurity.co.kr/insight/owasp-top-10-2025-rc1-explained/