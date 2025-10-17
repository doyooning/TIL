#HTML
#CSS
#JS 

---
### 글꼴 단위
em, rem (상대단위)
: 글꼴의 크기를 지정하는 단위
크기가 사용자의 기기의 해상도에 따라 유동적으로 변하는 단위

px (절대단위)
: 페이지 크기를 줄이든 키우든 같은 크기를 가짐

** 1rem = 16px, 2rem = 32px

### overflow 속성
내용이 요소 크기를 벗어났을 때 어떻게 처리할지 정하는 속성
CSS는 기본값으로  visible 속성을 가짐
-> hidden 처리하면 넘치는 컨텐츠를 숨길 수 있음

### Transform, Transition 속성
```
.profile {
  /* 크기 변형 */
  transform: translateY(50%);
  transition: transform 200ms cubic-bezier(0.18, 0.89, 0.32, 1.28);
}

.profile:hover {
  /* 프로필 애니메이션 느낌 추가 */
  transform: translateY(50%) scale(1.3);
}
```
transition : 대상을 / 얼마만큼 / 어떻게
ex) transform을 200ms동안 cubic-bezier(뽀짝하게 확대하는 그런 효과)

### 클래스명  작명 방법 - BEM
BEM = Block, Element, Modifier
부모의 클래스 이름을 이용하여 이름짓기
+해당 클래스의 역할

```
<form class="search-form">
  <input class="search-form_input">
  <button class="search-form_btn"></button>
</form>
```
부모 : 역할-태그
자식 : 부모의 클래스명_태그

### CSS 구조
html 페이지마다 각각의 css 파일 유지
+공통 서식 담은 css 파일(global.css) 유지
+브라우저 기본 서식 초기화 css 파일(reset.css)

html에 대응되는 각각의 css 파일들은 global.css를 import하고,
global.css 파일은 reset.css 파일 import

글꼴 적용 -> global.css에 import 형식으로 가져옴

사용자 지정 속성 -> variable.css로 관리 (:root {})

### CSS 셀렉트
특정 속성을 가진 태그 선택
: `[type=""]`

특정 요소 제외하고 선택
: `:not(...)`

### CSS 팁들
요소가 웹 화면을 벗어나지 않게
: `box-sizing: border-box;`

##### 위치 고정 관련
절대 위치 지정
: 부모 태그에 `position: relative;` 후 자식 태그에 `position: absolute;`
추가로 `left:` 나 `right: ` 등 지정해서 위치 조정

처음 만들 때 지정하면 좋은 것들
```
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
```
-> 크기 지정해줄 때 유용하다.

### 사용자 지정 색상 설정 및 사용
```
:root {
  /* html 어디에서나 사용자 지정 속성에 접근  */
  --purple: #6644b8;
  --darkpurple: #37274b;
  --background-purple: #120522;
  --border: rbga(255, 255, 255, 0.5);
}
```
사용할 때는 색상 입력 부분에 `var(--purple)` 처럼 사용

### HTML ~ JS 작업
Script언어는 순차적으로 작업하므로,
태그에 속성 등 작업시 script 태그 위치가 중요하다
(script 속성 지정을 먼저 해버리면 null에 속성을 지정하는 등의 문제가 발생)

좋은 방법은 head에 script 작성하고
```
window.onload = function() {
	// script 내용
}
```
이벤트 핸들러를 이용한 방법이 있다

표준 - script는 밖으로 빼낸다(script.js)
`<script src="script.js"><script>` 를 head 안에 추가


