#Kotlin

---
## 기초 문법
끝에 세미콜론 안 붙인다
fun main() { ... } : 메인 함수
print(), println() 다 있음

##### **변수 선언**
var를 사용
`var i = 10`

코틀린은 타입을 추론하는 기능을 가지고 있기 때문에 별도 타입 지정 불필요
타입을 명시적으로 지정해줄 때에는 콜론 찍고 타입 명시
`var i : Int = 10`

코틀린에서는 모든 타입들을 Wrapper 클래스 타입으로 제공

상수 선언은 val을 사용
`val v = 20`

-> Java에서의 final과 동일


##### **컴파일 타임 상수**
코틀린에서는 클래스 바깥에(컴파일 순서 상 더 먼저) 값들을 선언하거나 저장 가능
=> 탑 레벨 상수

탑 레벨 상수에 const 키워드를 붙여 컴파일 타임 상수를 만들 수 있음
(메인보다 우선적으로 컴파일됨, 성능상 우위)


##### **타입 캐스팅**
코틀린에서는 타입이 다르면 대입이 불가능(암묵적 형변환 안됨)
-> .to---() 를 사용

```kotlin
val i = 10
val name = "10"
i = name.toInt()
name = i.toString()
```


##### **String Interpolation**
문자열 출력 시 다음과 같은 형태 가능:

```kotlin
fun main() {
	var name = "이름"
	print("제 이름은 $name 입니다") // 변수 앞에 $만 붙여도 가능
	print("제 이름은 ${name + 10}입니다") // 변수 + 수식 가능
}
```


유용한 기능들은 kotlin 패키지에서 import해주는 것이 좋음
Math같은 경우 kotlin에도 math가 있어 이런 거 사용하도록 IDE가 유도함

Random
```kotlin
import kotlin.random.Random

fun main() {
	val randomNumber = Random.nextInt(0, 100) // 0 ~ 99
	print(randomNumber)
}
```
손쉽게 정수 랜덤 뽑기도 가능
범위는 From과 Until을 잘 숙지하기
From : Inclusive -> 0 포함
Until : Exclusive -> 100 제외
=> 0부터 99까지


##### **키보드 입력**
Scanner 사용
```kotlin
val reader = Scanner(System.`in`)
reader.nextInt()
reader.next()
```
Scanner : new를 사용하지 않고 생성 
in은 코틀린에서 사용할 수 없는 키워드
이런 키워드들은 백틱으로 감싸주는 방식으로 사용 가능


##### **조건문**
if문은 when문으로 그대로 치환 가능
switch문과 유사하나 훨씬 더 강력한 조건들을 지정 가능
```kotlin
when {
	i > 10 -> {
		print("10보다 크다)
	}
	i > 5 -> {
		print("5보다 크다")
	}
	else -> {
		print("")
	}
}
```

코틀린에서는 조건문을 식으로 취급
= 리턴값을 받아서 쓸 수 있다
```kotlin
var result = when {
	i > 10 -> {
		print("10보다 크다)
	}
	i > 5 -> {
		print("5보다 크다")
	}
	else -> {
		print("!!")
	}
}
print(result) // !!
```

삼항 연산
```kotlin
val result = if (i > 10) true else false
```


##### **반복문**
리스트 정의
`listOf(...)`

리스트 반복하기
```kotlin
val items = listOf(1, 2, 3, 4, 5)
items.forEach { items ->
	print(item)
}
```

`for i in items.length ... ` 표현식
```kotlin
for (i in 0..(items.size - 1)) {
	print(items[i])
}
// 이건 자바가 더 쉬운듯
```

while문은 자바와 동일
break, continue 동일


