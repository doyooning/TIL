#Java 

---
#### JSON 파일 불러오기
```
BufferedReader br = new BufferedReader(new InputStreamReader(System.in));  
        String clientId = "vtlypwpkhj";  
        String clientSecret = "myKRj3N9rPMhqYJxOZbMBnQFBNsYNFTpZf4SpfAU";  
  
        try {  
            System.out.print("주소를 입력하세요: ");  
            String address = br.readLine();  
            String addr = URLEncoder.encode(address, "UTF-8");  
            String apiURL = "https://maps.apigw.ntruss.com/map-geocode/v2/geocode?query=" + addr; 
    
            URL url = new URL(apiURL);  
            HttpURLConnection con = (HttpURLConnection) url.openConnection();  
            con.setRequestMethod("GET");
            con.setRequestProperty("x-ncp-apigw-api-key-id",  clientId); // REQUIRED  
            con.setRequestProperty("x-ncp-apigw-api-key",  clientSecret); // REQUIRED  
            int responseCode = con.getResponseCode();  
            if (responseCode == 200) {                  
            br = new BufferedReader(new InputStreamReader(con.getInputStream(), "UTF-8"));  
            } else {  
                br = new BufferedReader(new InputStreamReader(con.getErrorStream()));  
            }  
            String inputLine;  
            StringBuffer sb = new StringBuffer();  
            // inputLine으로 읽어들인 한 줄 한 줄을 sb로 append            
            while ((inputLine = br.readLine()) != null) {  
                sb.append(inputLine);  
            }  
            br.close();  
  
            // sb를 JSON 객체로 변환  
            JSONTokener tokener = new JSONTokener(sb.toString());  
            JSONObject object = new JSONObject(tokener);  
//            System.out.println(object);  
            JSONArray arr = object.getJSONArray("addresses");  
            for (int i = 0; i < arr.length(); i++) {  
                JSONObject obj = (JSONObject) arr.get(i);  
                System.out.println("address: " + obj.get("roadAddress"));  
                System.out.println("jibunAddress: " + obj.get("jibunAddress"));  
                System.out.println("경도: " + obj.get("x"));  
                System.out.println("위도: " + obj.get("y"));  
            }  
  
        } catch (Exception e) {  
            System.out.println(e.getMessage());  
        }
```

`String clientId = "vtlypwpkhj";`
`String clientSecret = "myKRj3N9rPMhqYJxOZbMBnQFBNsYNFTpZf4SpfAU";`
네이버 클라우드 API에서 발급받은 ID, Secret 값 입력
(개인 고유 번호라 비밀)

`String addr = URLEncoder.encode(address, "UTF-8");`
입력 공백도 문자처리 해주기 위함

`String apiURL = "https://maps.apigw.ntruss.com/map-geocode/v2/geocode?query=" + addr;`
API 요청 URL
- 'query=' 다음에는 요청할 정보를 넣어주어야 한다.
- URL이나 사용법은 API 공식문서에 다 나와있으니 참조하면 된다.

`URL url = new URL(apiURL);`
`HttpURLConnection con = (HttpURLConnection) url.openConnection();`
요청 전송을 위한 세팅

`con.setRequestMethod("GET");`
요청 방식 : GET

`con.setRequestProperty("x-ncp-apigw-api-key-id",  clientId);`
`con.setRequestProperty("x-ncp-apigw-api-key",  clientSecret);`
요청 속성에 내 고유번호 정보를 넣어줌

`int responseCode = con.getResponseCode();`
응답 코드 반환 (200 : 정상호출)

`br = new BufferedReader(new InputStreamReader(con.getInputStream(), "UTF-8"));`
`br = new BufferedReader(new InputStreamReader(con.getErrorStream()));`
정상호출시 getInputStream으로 응답 내용을  Stream으로 가져온다.
- 읽어들일 때 한글이면 UTF-8
오류시 getErrorStream

`String inputLine;`
`StringBuffer sb = new StringBuffer();`
`while ~ br.close();`
inputLine으로 읽어들인 한 줄 한 줄을 sb로 append



