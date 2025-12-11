#웹크롤링
#네트워크

---
### Http 요청으로 필요 정보 얻어오기

```
String url = "https://..."; // 요청 url 주소
HttpClient client = HttpClient.newHttpClient();  
HttpRequest request = HttpRequest.newBuilder()  
        .uri(URI.create(url))  
        .header("User-Agent", "Mozilla/5.0")  
        .build();
```
여기서 사용되는 URL은 Request URL이며, 홈페이지 URL과는 약간 다름
-> 필요한 정보에 대한 JSON 객체를 요청하는 주소임

`.newBuilder()` 로 빌더 형식으로 요청 조립(URI, 헤더)
!! 헤더 필수 !!

 *- 이 과정은 크롤링 사이트마다 다를 것 같지만 어떤 경우에도 헤더는 꼭 필요 -*

```
try {  
    HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());  
    String responseBody = response.body();  
  
    JSONObject json = new JSONObject(responseBody);  
    JSONArray list = json.getJSONArray("data");  
  
    System.out.println("=== 멜론 티켓 인기 공연 목록 ===\n");  
  
    for (int i = 0; i < list.length(); i++) {  
        JSONObject item = list.getJSONObject(i);  
        String title = item.optString("title", "제목 없음");  
        String place = item.optString("placeName", "장소 미정");  
        String period = item.optString("periodInfo", "기간 미정");  
        String region = item.optString("regionName", "-");  
  
        System.out.println((i + 1) + ". " + title);  
        System.out.println("장소: " + place + ", " + region);  
        System.out.println("기간: " + period);  
        System.out.println();  
    }  
  
} catch (IOException | InterruptedException e) {  
    e.printStackTrace();  
}
```
만든 request를 보내고, response의 body 부분을 받음
response 받음 -> JSON으로 변환 -> JSON 중 Array 타입 추출

멜론티켓의 구조는 JSON -> Array -> JSON 해야 콘서트 단일 정보에 접근 가능

*이 과정도 JSON에 어떤 내용을 담아서 송수신하느냐에 따라 추출 방식은 달라질 수 있음*

`item.optString("title", "제목 없음")`
optString(key, alt) : getString()의 optional 버전(Null 배제)
title : JSON 내에서의 키값




``




