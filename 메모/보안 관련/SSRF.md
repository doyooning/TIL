#보안 

---
# SSRF
SSRF(Server-Side Request Forgery)
공격자가 서버를 속여서 원래 서버가 접근할 수 있는 다른 시스템으로 요청을 보내게 만드는 공격입니다.

"SSRF 보호 오류"는 보통 서비스가 이러한 공격을 막기 위해 요청을 차단했을 때 나타나는 메시지입니다.

### SSRF가 발생하는 예시

예를 들어 서버에 URL을 입력받아 해당 페이지를 크롤링하는 기능이 있다고 가정해보겠습니다.
```
POST /preview
{
  "url": "https://example.com"
}
```

서버는 사용자가 입력한 URL로 직접 요청을 보냅니다.
공격자가 다음과 같이 입력하면:
```
{
  "url": "http://localhost:8080/admin"
}
```

서버는 자기 자신에게 요청을 보내게 됩니다.
더 위험한 경우:
```
{
  "url": "http://169.254.169.254/latest/meta-data/"
}
```

AWS EC2 환경에서는 인스턴스 메타데이터에 접근하여 IAM 자격 증명 같은 민감한 정보를 탈취할 수도 있습니다.

### SSRF 보호 기능은 무엇을 검사할까?
일반적으로 다음 주소들에 대한 요청을 차단합니다.

|차단 대상|예시|
|---|---|
|localhost|127.0.0.1, localhost|
|사설망|10.x.x.x, 172.16~31.x.x, 192.168.x.x|
|Loopback|127.0.0.0/8|
|Link Local|169.254.x.x|
|내부 DNS|*.internal|
|Docker 네트워크|172.x.x.x|

예를 들어 Spring 애플리케이션에서 URL을 받아 처리할 때:
```java
URI uri = URI.create(url);
InetAddress address = InetAddress.getByName(uri.getHost());
if (address.isLoopbackAddress()    || address.isSiteLocalAddress()) {
    throw new SecurityException("SSRF detected");
}
```

이런 식으로 방어합니다.

### 실제로 언제 보게 되나?
#### 1. 이미지 URL 등록
```
{
  "imageUrl": "http://localhost:8080/image.png"
}
```
→ "SSRF protection error"

#### 2. 웹훅(Webhook) 등록
```
{
  "callbackUrl": "http://192.168.0.10/callback"
}
```
→ 내부망 접근 시도로 판단

#### 3. AI 서비스에서 URL 읽기
ChatGPT, Claude, Slack App, GitHub App 등의 URL 입력 기능에서
```
http://127.0.0.1
```

입력 시 SSRF 보호에 의해 차단될 수 있습니다.

### 개발자 입장에서 방어 방법
1. 허용된 도메인만 접근 허용(Whitelist)
2. localhost, private IP 차단
3. DNS Rebinding 방어
4. HTTP Redirect 추적 시 재검증
5. AWS Metadata 주소 차단

특히 AWS에서는 아래 주소를 반드시 차단합니다.
```
169.254.169.254
```

