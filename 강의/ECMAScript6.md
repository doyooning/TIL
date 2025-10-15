#JS 

---
### ES6
= JavaScript 표준
ECMAScript 5가 모든 브라우저 100% 호환 찍고 ES6로 표준 전환중

### 객체 생성



### Map과 Set
기능은 Java의 것과 동일(키-값, 중복불가)

주요 메서드는 객체의 prototype에서 확인

**Map**
```
const map = new Map([['k1', 'v1'], ['k2', 'v2'], ['k3', 'v3']]);

// forEach 사용
map.forEach((value, key) => {console.log(value + " : " + key)});
// v1 : k1, v2 : k2, ...

// 반복문 사용
for(let key of map.keys()) {
	console.log("key : " + key);
}
// key : k1 ...
```


**Set**
```
const set = new Set();
set.add('javascript').add('java').add('react');
// Set은 add를 통해 추가

// forEach 사용
set.forEach((val, val2, set) => {
	console.log(val, val2, set);
}); 
// javascript javascript Set ...

// 반복문 사용
for(const value of set) {
	console.log(value);
}
// javascript, java, react, ... 
```

***For ... in ... vs For ... of ...***

==For ... in ...==
: 객체의 속성 이름(Key)을 순회할 때 사용
```
const user = { name: "도윤", age: 20, city: "Seoul" };

for (let key in user) {
  console.log(key);          // name, age, city
  console.log(user[key]);    // 도윤, 20, Seoul
}
```

✅ 특징
- key(속성 이름)를 순회함
- 객체 뿐 아니라 배열에도 쓸 수 있지만, **배열 인덱스**(문자열로 변환된 index)를 반환함
- **순서가 보장되지 않음** (객체는 원래 순서가 없는 구조)

⚠️ *주의:*  
배열에 `for...in`을 쓰면 index 말고도 프로토타입 속성까지 순회될 수 있음 → **배열에는 쓰지 않는 게 좋음**

==For ... of ...==
: 반복 가능한(iterable) 객체의 값(value)을 순회할 때 사용
```
const arr = ["a", "b", "c"];

for (let value of arr) {
  console.log(value);  // a, b, c
}
```

✅ 특징
- 값을 직접 꺼내옴
- 순서가 보장됨
- **객체에는 사용할 수 없음** (객체는 iterable이 아님)

!! 예시 비교
```
const arr = ["a", "b", "c"];

for (let i in arr) {
  console.log(i); // 0, 1, 2  ← 인덱스
}

for (let v of arr) {
  console.log(v); // a, b, c  ← 값
}
```

**결론**
for in -> 키를 순회, for of -> 값을 순회
배열에서 for in 하면 인덱스를 반환 (배열은 각 인덱스에 요소가 매핑되어있는 형태이므로)


### Array
여러가지 API

`push()` : 배열의 가장 끝에 데이터를 추가 
`slice(시작, 끝)` : 배열에서 시작(인덱스 번호) 데이터부터 끝(인덱스 번호 -1) 데이터까지를 가져옴
`splice(시작, 개수, 데이터)` : 시작(인덱스 번호) 데이터부터 개수만큼의 데이터를 데이터로 변경함
`pop()` : 배열의 가장 끝 데이터를 추출(배열에서 제거됨)
`shift()` : 배열의 가장 처음 데이터를 추출(배열에서 제거됨)


