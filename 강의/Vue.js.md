#JS 
#Vue3

---
### Vue 개요
**Vue란?**
사용자 인터페이스(UI, User Interface)를 만드는 데 사용하는 자바스크립트 기반 오픈소스 프로그레시브 프레임워크

대부분 오픈소스 프레임워크는 모두 특정 소프트웨어 아키텍처 패턴에 기반을 둠

• 뷰는 MVVM(Model-View-ViewModel) 아키텍처 패턴을 따름

• MVVM 아키텍처 패턴:
데이터를 보여주는 뷰(view)와 데이터를 처리하는 모델(model) 사이에 데이터를 중개하는 뷰 모델(view model)을 두어 UI와 데이터 처리 로직의 상호 의존성을 줄이는 아키텍처 패턴

**Vue의 특징**
• 가상 DOM:
실제 DOM에서 발생할 수 있는 불필요한 렌더링을 최소화

• 양방향 데이터 바인딩:
화면 업데이트를 실시간으로 처리 가능




최초 Vue 프로젝트 생성
```bash
npm create vue@3.10.3
```

Vue 서버 실행
```bash
npm install
npm run dev
```

http://localhost:5173/ 클릭해서 확인
ctrl + c로 연결 해제

### script
```vue
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
```vue
<h1 v-bind:class="className">안녕하세요</h1>
<h1 :class="className">안녕하세요</h1>
```

v-if : 조건부 렌더링 처리(DOM에서 생성 자체를 안함)
v-show : 조건부 렌더링 처리(DOM에서 생성, display:none 처리)

v-cloak : 데이터 바인딩 완료 전 데이터 화면 노출 방지
v-for : 반복 렌더링
```vue
<script>
export default{
	data(){
		return {
			user: {
	        name: 'john',
	        age: 20,
	        gender: 'male',
		    },
		};
	},
};
</script>
<template>
	<ul>
		<li v-for="(value, key, index) in user" :key="index">
		  {{ index }}, key: {{ key }}, index: {{ value }}
		</li>
	</ul>
</template>	
```


### 이벤트 
v-on 디렉티브와 methods 옵션 속성으로 이벤트 연결하기
이벤트 핸들러 : 이벤트 타입과 일치하는 이벤트 발생되면 실행

수식어(modifier)
: 이벤트 처리 방식을 제어하는 데 사용하는 기능
```vue
methods: {
    onkeyUpHandler(event) {
      if (event.keyCode === 13) {
        console.log('Enter Key');
      }
    },
...
  <input type="text" @keyup.enter="onkeyUpHandler" />
```

반응성(reactivity)
: 데이터의 변화를 감지하고 자동으로 화면 UI를 업데이트하는 기능

반응성 시스템(reactivity system)
: 데이터 변화를 감지하고 업데이트하는 기능을 제공하는 시스템

v-once : 딱 한번만 렌더링하고 그 이후 접근은 막음(화면 업데이트를 막음)

메모이제이션(memoization)
: 이전에 계산한 결과를 저장해 중복계산을 피하고 실행 속도를 향상시키는 프로그래밍 기술
```vue
  <div v-memo="[name, gender]"></div>

  <p>이름: {{ name }}</p>
  <p>성별: {{ gender }}</p>
  <p>나이: {{ age }}</p>

  <button @click="name = 'ssg1'">이름변경</button>
  <button @click="gender = 'female'">성별변경</button>
  <button @click="age = 30">나이변경</button>
```


form 처리
v-model 디렉티브
양방향 데이터 바인딩

입력한 요소의 값을 가져올 때 v-model 디렉티브를 사용

