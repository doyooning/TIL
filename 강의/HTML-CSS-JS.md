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

### 클래스명  작명 방법 - BEM
부모의 클래스 이름을 이용하여 이름짓기
+해당 클래스의 역할

```
<form class="search-form">
  <input class="search-form_input">
  <button class="search-form_btn"></button>
</form>
```
