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

• scoped 속성
: 컴포넌트에 적용한 스타일이 해당 컴포넌트에만 적용되도록 함 

• App 컴포넌트에서 < style >태그에 scoped 속성 적용

**컴포넌트 생명주기**

라이프사이클 훅 
• 생명주기(lifecycle)
: 컴포넌트가 생성되고 파괴되기까지의 여러 단계 
• 훅(hook)
: 특정 기능과 역할을 하는 함수 
• 라이프사이클 훅(lifecycle hook)
: 컴포넌트에서 호출되는 훅

[[화면 캡처 2025-11-18 155212.jpg]]
[[화면 캡처 2025-11-18 155644.jpg]]

### 컴포지션 API
**반응형 데이터 정의하기** 
: `ref(), reactive()` 

1. ref() : 기본 자료형, 참조 자료형 구분없이 사용 가능
	 .value 속성으로 접근하기 때문에 참조형타입에 반응성이 전달되지 않는다
2. reactive() : 참조 자료형(객체나 배열)에 반응성을 부여할 때 사용

==템플릿 ref==
ref() 함수로 생성한 반응형 데이터를 HTML 태그에서 검색할 때 사용

• ref속성으로 다른 컴포넌트를 참조할 수도 있음 
• 접근할 수 있는 데이터를 `defineExpose()` 함수로 지정해야 함


• 래핑(wrapping)
: 어떤 값을 다른 객체로 감싸는 것 
• 언래핑(unwrapping)
: 어떤 값을 다른 객체가 감싸고 있을 때, 감싸진 상태를 해제하고 원래 값에 직접 접근하는것

객체는 reactive, 나머지는 ref로 관리하면 좋다

==computed==
• `computed()` 함수의 데이터를 직접 변경하는 방법
: `computed()` 함수를 `get()`과 `set()` 함수로 세분해서 사용


함수 정의 
• 실행 가능한 코드 블록을 함수(메서드)로 정의할 수 있음

감시자 속성 
`watch(반응형데이터, (현재값, 이전값) => {}, 옵션 : immediate/deep/once/post | sync)`
 - 반응형 데이터
	 ==ref()==
	 반응형 데이터는 반응형 데이터 자체를(변수의 값이 완전히 달라지는 것) 감시
	 (배열이나 객체의 속성 값 변경은 감시 x)

ref()로 생성한반응형데이터를감시하는경우 • 오직반응형변수의값이완전히달라지는지만감시 • deep속성을사용하면감지가능(단,currentValue와prevValue의값이같음)

reactive()로 생성한반응형데이터를감시하는경우 • 내부값이변경되면감지가능 • currentValue와 prevValue의 값이 같음 • 반응형데이터를콜백함수형태로사용하면이전값을알수있음

```vue
<script setup>
import { reactive, ref, watch } from 'vue';
const state = reactive({ count: 0 });
// 반응형 데이터를 콜백 함수 형태로 사용한 인자로 watch()에게 넘겨주기
watch(
  () => state.count,
  (curVal, preVal) => {
    console.log(curVal, preVal);
  }
);
</script>
<template>
  <h1>reactive : {{ state.count }}</h1>
  <button @click="state.count++">증가</button>
  <button @click="state.count--">감소</button>
</template>
```

`watchEffect()`
- 콜백 함수 내부에서 참조하는 모든 반응형 데이터를 감시
- optionsAPI에서는 deep, immediate 옵션이 true로 적용되어 있음
- 콜백 함수의 시점을 결정하는 flush 옵션이 있음

immediate와deep속성을사용하지않음 • 감시하는반응형데이터중하나라도값이변경되면콜백함수호출


`watchPostEffect()` 
• flush속성이결합된기능이라서 flush속성을지정하지않아도 DOM이갱신된후콜백함수실행


`props` 
컴포넌트에서 다른 컴포넌트로 데이터를 전달할 때 사용하는 속성
defineProps()함수를사용할때 ① prop의타입을단독으로지정 ② prop의타입을배열로지정 ③ type,default,required속성사용

DefineProps 컴포넌트에서는 부모에서 전달한 반응형 데이터를 defineProps()를 사용하여 받아서 출력할 수 있다.

`emits` 
emits로이벤트호출할때 ① 별도의함수를이벤트핸들러로연결해호출 ② 직접호출

optionsAPI : 컴포넌트에서 전달한 이벤트를 전달받을 때 emits를 사용
CompositionAPI : defineEmits() 함수를 사용

이벤트 등록할 때는 KebabCase로 작성하는 것을 권장(대소문자 인식을 못 함)
-> 연결할 때 알아서 인식해서 CamelCase로 바꿔줌

props는 데이터를, emits는 이벤트를 전달하는 것으로 이해하면 됨


### Vue 라우터

VueJS는 SPA 기반, 자체적으로 라우팅 기술 처리 방법을 제공하지 않음
-> 외부 라이브러리 설치

Vue 라우터 설치
```
$ npm install vue-router@4
```


라우터인스턴스생성및등록 
• 라우터인스턴스생성:vue-router패키지에서제공하는 createRouter()함수를 사용해라우트구성객체(routeconfiguration object)를 인수로전달

• 라우트구성객체 
§ history속성:뷰라우터의히스토리관리방식을정의 
§ routes속성:애플리케이션의경로와해당경로에렌더링될컴포넌 트를매핑하는라우트객체(routeobject)를포함

• 라우트객체의component속성값을지정할때 
① 정적임포트(staticimport)방식:화면에렌더링해야하는경로가아 니어도애플리케이션시작시점에서컴포넌트를메모리에로드 
② 동적임포트(dynamicimport)방식:해당컴포넌트가필요한순간에 로드 
• 일반적으로루트경로(/)에만정적임포트로컴포넌트를지정하고,나머 지는동적임포트로지정


뷰 라우터사용하기 
• RouterLink 컴포넌트:사용자가클릭할수있는링크를생성 
§ 전통적인HTML방식의웹애플리케이션에서는태그와href속 성사용 
§ 뷰라우터에서는RouterLink컴포넌트와to속성사용 
• RouterView 컴포넌트:라우터인스턴스를생성할때정의한라우트객 체정보중에서현재경로와일치하는경로의컴포넌트를렌더링

동적경로매칭 
• 동적경로매칭:URL의일부분이동적으로변경되는경로를처리 
• 동적세그먼트 
• 콜론(:)으로시작하는임의의식별자 
• URL에서동적으로변경되는부분을동적세그먼트로나타냄 
• 개수제한없음

매개변수가져오기 
• 쿼리스트링(querystring) 
• URL끝에?로매개변수를붙여요청하는경우?부터시작해뒤에오 는일련의문자 
• route인스턴스로쿼리스트링의값을쉽게가져올수있음 
• route인스턴스를참조하는방법 
• 옵션스API:this키워드로$route전역속성을참조 
• 컴포지션API:useRoute()함수로route인스턴스의값을변수에 할당해서참조

