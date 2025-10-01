#Java 

---
#### Java TPC = Thinking, Presentation, Coding
생각하고, 표현하고, 코딩한다

#### JSON 파일 불러오기

```
String src = "jsonEx.json";    
InputStream is = jsonFileLoadEx.class.getResourceAsStream(src);  
if(is == null) {  
    throw new NullPointerException("Can't find resource " + src);  
}
```
해당 클래스가 만들어진 곳에서 src와 연결, 연결 객체를 스트림으로 넘겨줌
Stream is는 문자열 데이터를 받은 것이라서, 받은 문자열 데이터를 JSON 형태로 토큰화

`JSONTokener tokener = new JSONTokener(is);`
토큰화 객체 선언
`JSONObject object = new JSONObject(tokener);`
object로 다시 JSON 객체로 만들어줌
`JSONArray jsonArray = object.getJSONArray("jsonArray");`
object 안에 있던 배열 가져옴

```
for (int i = 0; i < jsonArray.length(); i++) {  
    JSONObject jsonObject = (JSONObject) jsonArray.get(i);  
    System.out.print(jsonObject.get("name") + "\t");  
    System.out.print(jsonObject.get("phone") + "\t");  
    System.out.println(jsonObject.get("address"));  
}
```
반복문으로 배열 돌면서 배열 안의 JSON 객체의 value 출력

