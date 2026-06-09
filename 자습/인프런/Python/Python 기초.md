#Python

---
Python은 Camel Case 기본으로 사용
ex) myVar

변수 선언할 때 타입 명시가 필요없어 매우 직관적이다

# 1. 기본
### 변수
: 데이터와 메모리
메모리의 복잡한 주소 대신, 데이터에 접근하기 위해 부여하는 이름

변수 정의(선언 및 초기화)
그냥 `변수명 = 값` 구조

주석
한 줄 주석: `#`
여러 줄 주석: `''' '''`

### 데이터 타입
정수형(Integer, int)
메모리가 허용하는 범위 내에서 무한히 큰 정수를 다룰 수 있음

실수형(Float, float)
8바이트 메모리 내에서 데이터를 처리, 정수와 달리 매우 큰 수를 저장할 때 데이터 손실(반올림/절삭)이 발생할 수 있음을 주의해야 함
ex) `1.2345678901234567890123` 
 -> `1.2345678901234567`

문자열(String, str)
파이썬은 문자와 문자열을 통합하여 `str` 타입으로 관리하며, 작은따옴표(' ')나 큰따옴표(" ")를 구분 없이 사용할 수 있음

불리언(Boolean, bool)
참(True)과 거짓(False)을 나타내며, 반드시 첫 글자를 대문자로 표기해야 함

형변환(Type Casting) 
데이터 타입이 서로 다른 경우 연산이 불가능하므로, `int()`, `str()` 등의 함수를 사용하여 타입을 변환 가능

### 컨테이너 자료형
데이터를 효율적으로 관리하기 위한 자료 구조

리스트(List, `[]`)
배열과 유사한 구조로, **인덱스(0부터 시작)를 사용**하여 데이터에 접근
핵심 기능:
- 데이터 추가(`append`), 삽입(`insert`), 삭제(`pop`, `remove`), 정렬(`sort`), 슬라이싱 등을 지원
- 다양한 데이터 타입(정수, 실수, 문자열, 불리언 등)을 혼합하여 저장

튜플 (Tuple, `()`)
리스트와 구조는 비슷하지만, **데이터 수정이 불가능**
핵심 기능: 
- 데이터 참조 및 슬라이싱은 가능하나, 추가/삭제/변경 등의 수정 작업은 제공되지 않음

딕셔너리 (Dictionary, `{}`)
인덱스 대신 **키(Key)와 값(Value)** 의 쌍으로 데이터를 저장하는 사전형 구조
핵심 기능:
- 키는 유일해야 하며, 키를 통해 값을 빠르게 찾을 수 있음
- 특정 데이터를 삭제할 때는 `del` 키워드 사용

### 연산자
산술연산자
프로그래밍 언어 사칙연산과 동일
문자열 간의 더하기(+) 연산으로 문자열 결합 기능

비교연산자
프로그래밍 언어 비교 연산과 동일
문자열 비교 시 아스키(ASCII) 코드 값을 기준으로 연산이 이루어짐

논리연산자
부정: `not`
논리곱: `and`
논리합: `or`

### 조건문
if문, if-else문, elif문
콜론(:) 사용과 들여쓰기 필수

파이썬은 타 언어와 달리 `switch`문을 제공하지 않지만, 
`if`, `elif`, `else`문을 조합하여 모든 논리적 분기 처리가 가능

### 반복문
for문
컨테이너 자료형을 이용한 반복문
- 리스트, 튜플, 딕셔너리

`range()`를 이용한 반복문
- `range(시작, 끝, 단계)`
시작: 시작할 숫자
끝: 숫자가 몇에 도달하였을 때 반복을 끝낼 것인지
ex) 9까지(포함) 반복하고 싶다 -> 10
단계: 반복할 때의 단위

### 함수
함수 정의
`def` 키워드 사용
```python
def fun(x, y):
	result = x + y
	print('result: {0}'.format(result))
	return result
```

함수 호출,
매개변수와 데이터 반환,
전역 변수/지역 변수
-> 프로그래밍 언어 동일

# 객체 지향 프로그래밍
OOP(Object-Oriented Programming)

### 객체와 클래스
###### 객체
속성(상태)과 기능(동작)을 하나로 묶은 프로그램의 단위

###### 클래스
객체를 만들기 위한 설계도(틀)
하나의 클래스를 통해 여러 개의 객체를 독립적으로 생성

###### 클래스 정의 및 객체 생성
`class` 키워드를 사용하여 클래스를 정의
관례상 클래스 이름의 첫 글자는 대문자로 시작

`__init__` 메서드(생성자)를 통해 객체가 생성될 때 초기 속성을 설정
`self`를 사용하여 클래스 내부의 속성과 메서드에 접근

```python
class Bike:
	def __init__(self):
		self.color = 'black'
		self.weight = 3
		
	def drive(self):
		print('drive')
		
	...
```

###### 객체 독립성
클래스로부터 생성된 객체들은 각각 독립적인 메모리 공간을 가지므로,
하나의 객체 속성을 변경해도 다른 객체에는 영향을 주지 않음

###### 접근자
퍼블릭(Public)
외부에서 자유롭게 접근 가능한 속성과 메서드

프라이빗(Private)
변수명 앞에 언더바 2개(`__`)를 붙여 외부 직접 접근을 제한하는 방식
데이터 보호와 보안을 위해 중요
private 네이밍 규칙:
- 접두사: `__` 사용
- 접미사: `_` 까지 허용

###### self 매개변수
`self`는 객체 자신을 가리키며, 파이썬 인터프리터가 자동으로 객체를 메서드에 전달
객체 내부의 속성을 참조하거나 수정할 때 필수적으로 사용

```python
class PayGildong:
	def __init__(self):
		self.day = 25
		self.__money = 1000000
		
	def changeMoney(self, money):
		self.__money = money
		
	def getMoney(self):
		return self.__money

```

###### 객체 생성 및 초기화 메서드
객체 생성은 `__name__()` 메서드가 담당,
초기화는 `__init__()` 메서드가 담당

클래스 메서드: 객체 생성
인스턴스 메서드: 객체 초기화

###### 정적 메서드와 클래스 메서드
정적 메서드(`@staticmethod`)
객체 생성 없이 클래스 이름을 통해 바로 호출 가능한 메서드

```python
class Earth:
	
	@staticmethod
	def getRadius():
		return 6400
	
	...
# Earth.getRadius()
```

클래스 메서드(`@classmethod`)
첫 번째 인자로 `cls`를 받아 클래스 자체의 속성에 접근하는 메서드
객체들 간의 데이터 공유가 필요할 때 유용함

```python
class PopulationStatistics:
	
	population = 0
	
	def plusPopulation(self):
		PopulationStatistics.population += 1
	
	@classmethod
	def getPopulation(cls):
		return cls.population
	
	...
# PopulationStatistics.getPopulation()
```

### 상속
접근 제어자 중 public은 상속되지만 private은 상속되지 않음

###### 생성자 호출과 속성 상속
Python에서 자식 클래스가 부모 클래스의 속성을 사용하려면
`__init__`메서드 내에서 부모의 `__init__`을 강제로 호출해주어야 함

###### 오버라이딩
부모 클래스에 정의된 메서드를 자식 클래스에서 상황에 맞게 재정의하여 사용

###### 다중 상속
Python은 다중 상속을 지원함
하나의 자식 클래스가 여러 부모 클래스를 상속받을 수 있음

###### 추상 클래스
자식 클래스에서 반드시 특정 메서드를 구현하도록 강제하는 클래스
```python
from abc import ABCMeta
from abc import abstractmethod

class Calculator(metaclass=ABCMeta):

	def __init__(self):
		pass
		
	@abstractmethod
	def add(self):
		pass
	
	@abstactmethod
	def sub(self):
		pass
	
```

abs 모듈과 데코레이터를 사용해 구현

###### super()
부모 클래스 객체를 가리키는 키워드
`__init__` 등 부모의 메서드를 더 간결하고 안전하게 호출

