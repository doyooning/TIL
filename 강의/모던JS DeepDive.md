#JS 

---
# 문자열 타입

> 자바스크립트에서 문자열은 `원시타입` 이며, `변경 불가능한 값(immutable value)` 이다. 즉, 문자열이 한번 생성되면 그 문자열은 변경할 수 없다는 것을 의미

- `작은따옴표('')` , `큰 따옴표("")`, `백틱(``)` 으로 텍스트를 감싼다.


# 템플릿 리터럴

> `ES6`부터 템플릿 리터럴(template literal)이라고 하는 `새로운 문자열 표기법` 이 도입되었다.

- `멀티 라인 문자열(muti-line string)`, `표현식 삽입(expression interpolation)`, `태그드 템플릿(tagged template)` 등 편리한 문자열 처리 기능을 제공
- 템플릿 리터럴은 `런타임(runtime) 에 일반 문자열로 변환`되어 처리
- 템플릿 리터럴은 `백틱(``)` 을 사용해 표현


### 멀티라인 문자열

```js
// 일반 문자열 내에서는 줄바꿈(개행)이 허용되지 않는다.
var str = "Hello
world";
// SyntaxError: Invaild or unexpected token

// 1️⃣ 그래서 HTML 문자열을 작성시 기존 문자열 방식으로는 이스케이프 시퀀스(escape sequence)를 사용해야 한다.
var template = '<ul>\n\t<li><a href="#">Home</a></li>\n</ul>';
console.log(template);
```

```html
<!-- 1️⃣ - 결과 -->
<ul>
  <li><a href="#">Home</a></li>
</ul>
```

```js
// 🔎 2️⃣ - 멀티라인 문자열 사용시
var template = `<ul>
	<li><a href="#">Home</a></li>
</ul>`;
console.log(template);
```

```html
<!-- 2️⃣ - 결과 -->
<ul>
  <li><a href="#">Home</a></li>
</ul>
```

  
### 표현식 삽입

> 자바스크립트에서 문자열 연결은 `+` 사용해 연결할 수 있다. 하지만 `표현식 삽입(expression interpolation)` 을 통해 더욱 깔끔하고 쉽게 문자열을 삽입할 수 있다.

```js
var first = "YOUNG";
var last = "MIN";

// ES5: 문자열 연결
console.log("My name is " + first + " " + last + "."); // My name is YOUNG MIN.

// ES6: 표현식 삽입
console.log(`My name is ${first} ${last}.`); // My name is YOUNG MIN.
```

