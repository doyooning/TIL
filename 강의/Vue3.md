#JS 
#Vue3

---
최초 Vue 프로젝트 생성
```
npm create vue@3.10.3
```

Vue 서버 실행
```
npm install
npm run dev
```

http://localhost:5173/ 클릭해서 확인
ctrl + c로 연결 해제

### script
```
export default{
	// data: 컴포넌트 전반에 걸쳐서 사용할 값
}
```
export default를 사용
options API 규칙 준수



### template
디렉티브들
v-html : 바인딩된 데이터가 html태그를 가지고 있으면 변환해줌
v-text : 문자 그대로 바인딩

v-pre : 해당 태그를 데이터바인딩을 하지 않고 그대로 출력함
v-bind : 속성과 데이터를 연결(축약 -> :)
```

```

v-if : 조건부 렌더링 처리(DOM에서 생성 자체를 안함)
v-show : 조건부 렌더링 처리(DOM에서 생성, display:none 처리)

v-cloak : 데이터 바인딩 완료 전 데이터 화면 노출 방지
v-for : 반복 렌더링
```

```

