#JS 
#Vue3

---
# Vue 개요
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
		  {{ index }}, key: {{ key }}, value: {{ value }}
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

### 계산된 속성과 감시자 속성
computed -> options API에서 제공하는 속성

==계산된 속성(computed)==
: computed 옵션 속성으로 정의해 사용 
컴포넌트에서 자주 사용하는 데이터를 캐시해 애플리케이션의 성능을 향상시킴 
코드의 가독성을 높임
(cache: 데이터를 메모리같은 곳에 임시 저장)

```vue
<script>
export default {
  data() {
    return {
      firstName: 'Minsu',
      lastName: 'Kim',
      numberArray: [1, 2, 3, 4, 5],
    };
  },
  computed: {
    fullName() {
      console.log('computed fullName');
      return `${this.lastName} ${this.firstName}`;
    },
  
    evenSum() {
      return this.numberArray
        .filter((v) => v % 2 == 0)
        .reduce((acc, cur) => acc + cur);
    },
  },
};
</script>
<template>
  <h1>{{ lastName }}-{{ firstName }}</h1>
  <h1>{{ firstName }} - {{ lastName }}</h1>
  <h1>{{ fullName }}</h1> // Kim Minsu (computed fullName)
  <h1>{{ fullName }}</h1>
  <h1>{{ evenSum }}</h1> // 6
</template>
```

==감시자 속성(watch)==
: watch 옵션 속성으로 정의해 사용 
데이터의 변경을 감시하고, 변경이 감지될 때마다 특정 동작을 수행할 수 있게 함

기본 사용법 
-> watch 옵션 속성에도 함수를 정의해야 함

```vue
...
  data() {
    return {
      inputStr: '',
  ...
  watch: {
    inputStr(newValue, oldValue) {
      console.log(`old value: ${oldValue}`);
      console.log(`new value: ${newValue}`);
    },
...

  <input type="text" v-model="inputStr" />

```

근데 기본 사용법으로는 객체나 배열의 변경을 추적할 때 적용이 안 된다
=> 깊은 사용법 적용

script
```vue
  watch: {
    inputStr(newValue, oldValue) {
      console.log(`old value: ${oldValue}`);
      console.log(`new value: ${newValue}`);
    },
    num(newValue, oldValue) {
      console.log(`old value: ${oldValue}`);
      console.log(`new value: ${newValue}`);
    },
    arr: {
      handler(newValue, oldValue) {
        console.log(`old value: ${oldValue}`);
        console.log(`new value: ${newValue}`);
      },
      deep: true,
    },
    obj: {
      handler(newValue, oldValue) {
        console.log(`old value: ${JSON.stringify(oldValue)}`);
        console.log(`new value: ${JSON.stringify(newValue)}`);
      },
      deep: true,
    },
  },
```

arr(배열), obj(객체)에 깊은 사용법 적용
handler -> 값 변동 감시, 콜백 함수
deep -> 초기 렌더링시 콜백 함수 호출 여부(기본: false)

---
# 컴포넌트
컴포넌트 기반 아키텍처

**컴포넌트**(component)
: 애플리케이션을 구성하고 관리하는 방식, 특정 기능이나 작업을 독립적으로 수행하기 위한, 논리적으로 구분된 코드단위 

컴포넌트 기반 아키텍처(CBA,Component-Based Architecture)
: 한 웹 애플리케이션을 여러 컴포넌트로 구분해, 페이지 단위가 아니라 컴포넌트 단위로 설계하는 방식

컴포넌트 기반 아키텍처 설계 방식의 장점 
• **재사용성**(reusability)
: 같은 기능이나 작업이 필요한 곳에서 재사용 가능 
• **독립성**(independence)
: 다른 컴포넌트에 영향을 주지 않고 특정 컴포넌트만 수정하거나 교체할 수 있음 
• **모듈성**(modularity)
: 복잡성이 줄어듦 
• **확장성**(extensible)
: 컴포넌트에서 구성요소를 추가하거나 수정해 새로운 동작이나 기능 제공 
• **캡슐화**(encapsulation)
: 내부의 복잡성을 숨기고, 단순하게 기능만 사용할 수 있는 인터페이스제공


컴포넌트 - 트리 구조로 형성, 각각의 부모 관계 엄격하게 구분되어 있음

컴포넌트의 생명주기는 매우 중요
컴포넌트가 생성되고 생명주기의 여러단계를 거치면서 컴포넌트가 최종적으로 웹브라우징

**전역 등록과 지역 등록**
전역(global) 등록
```

```

지역(local) 등록
```

```


컴포넌트 스타일의 범위 
• scoped속성:컴포넌트에적용한스타일이해당컴포넌트에만적용되도록함 
• App컴포넌트에서 < style >태그에scoped속성적용


**컴포넌트 생명주기**

라이프사이클훅 
• 생명주기(lifecycle):컴포넌트가생성되고파괴되기까지의여러단계 
• 훅(hook):특정기능과역할을하는함수 
• 라이프사이클훅(lifecyclehook):컴포넌트에서호출되는훅

[[화면 캡처 2025-11-18 155212.jpg]]
[[화면 캡처 2025-11-18 155644.jpg]]


