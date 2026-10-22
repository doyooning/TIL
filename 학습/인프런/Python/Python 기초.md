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

# 2. 프로그래밍
### 객체 지향 프로그래밍
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

### 모듈 (Module)
관련된 함수나 변수들을 하나의 파이썬 파일(.py)로 묶어 부품처럼 사용하는 것
`import [파일명]`을 통해 모듈 전체를 가져오거나, 
`from [파일명] import [함수명]`을 사용하여 필요한 기능만 선택적으로 가져올 수 있음
이미 작성된 검증된 코드를 재사용하여 효율적인 프로그래밍이 가능

```python
# cooking.py에 함수들이 구현되어 있을 때
# module
import cooking
cooking.makeJjajang()
cooking.makePasta()
```

```python
# module
from cooking import makeJjajang
from cooking import makePasta
```

### 패키지(Package)
여러 개의 모듈을 디렉토리(폴더) 단위로 체계적으로 관리하는 방법
패키지로 인식되려면 디렉토리 안에 반드시 `__init__.py`라는 이름의 초기화 파일이 있어야 함
규모가 큰 프로젝트에서 파일들을 계층적으로 정리할 수 있음

#### 패키지 추가 및 활용
###### 경로 관리
기본적으로 같은 경로에 있는 모듈과 패키지만 불러올 수 있지만, 
파이썬이 설치된 경로의 `site-packages` 폴더에 패키지를 넣으면 프로젝트 경로에 상관없이 어디서든 자유롭게 불러와 사용할 수 있음

###### 핵심 요약
`sys` 모듈의 `path`를 통해 파이썬이 모듈을 찾는 경로를 확인할 수 있으며, 이를 활용해 개발 환경을 유연하게 설정할 수 있음

### 예외 처리
#### 예외(Exception)
프로그램 실행 중에 발생하는 오류로, 개발자가 사전에 예측하여 대응할 수 있는 상황을 의미
에러(Error)와는 다름

###### 기본적인 예외 처리
```python
try:
	userData = int(input())
	result = int(10 / userData)
	
except ZeroDivisionError as e:
	print('0으로 나눌 수 없음: {0}'.format(e))

except Exception as e:
	print('예외 발생: {0}'.format(e))

else:
	print('완료')
	
finally:
	print('프로그램 종료')	
```
`try`, `except` 구문을 사용하여 처리
`else`: 예외가 발생하지 않았을 때 실행
`finally`: 예외 발생 여부와 상관없이 항상 실행, 주로 외부 자원 해제(DB)에 사용됨

###### Exception 클래스
모든 예외의 최상위 클래스

###### 사용자 정의 예외 클래스
`Exception` 클래스를 상속받아 개발자가 직접 자신만의 예외를 정의하고, 
`raise` 키워드를 통해 필요한 상황에서 예외를 발생시킴

```python
class MyException(Exception):
	def __init__(self, e):
		super().__init__('{0}으로 나눌 수 없습니다.'.format(e))
		
def division(x, y):
	if y != 0:
		return x / y
	else:
		raise MyException(y)

try:
	numX = int(input())
	numY = int(input())
	result = division(numX, numY)
	print('result : {0}'.format(result))
	
except MyException as e:
	print('예외 발생: {0}'.format(e))
	
else:
	print('정상 실행')
	
finally:
	print('자원 해제')
```

### 데이터 입출력
#### 파일 입출력의 기초
###### 기본 절차
파일을 다룰 때는 `열기(open) → 읽기(read) 또는 쓰기(write) → 닫기(close)` 과정을 반드시 거쳐야 함

###### 외부 자원 관리
파일은 파이썬 내부 자원이 아닌 외부 자원이므로, 작업 후에는 `close()` 함수를 통해 메모리 누수를 방지하고 자원을 반납해야 함

#### 파일 모드
- `r` (Read): 읽기 전용 모드
- `w` (Write): 쓰기 전용 모드 (파일이 존재하면 기존 내용을 덮어씀)
- `a` (Append): 추가 모드 (기존 내용 뒤에 새로운 데이터를 덧붙임)
- `x`: 파일이 이미 존재하면 예외(IOError) 발생

```python
# 파일 쓰기(열기 -> 쓰기 -> 닫기)
f = open('C:/...testFile.txt', 'w')
f.write('hello python')
f.close()

# 파일 읽기(열기 -> 읽기 -> 닫기)
f = open('C:/...testFile.txt', 'r')
print('f.read(): {0}'.format(f.read()))
f.close()

# 쓰기 전용
f = open('C:/...testFile.txt', 'a')
f.write('HELLO PYTHON')
f.close()

# 이미 존재하면 예외(IOError) 발생
f = open('C:/...testFile.txt', 'x')
f.write('HELLO PYTHON')
f.close()
```

#### 텍스트 파일 읽기 및 쓰기 함수
- `with open(...) as f:` 구문: 
  `close()`를 명시적으로 호출할 필요 없이 파이썬이 **알아서 파일을 닫아주어** 매우 편리

- 리스트 데이터 처리:
    - `write(lines)`: 리스트 형태의 데이터를 파일에 기록할 때 유용함
    - `readlines()`: 파일의 모든 라인을 읽어 리스트로 반환
    - `readline()`: 파일에서 한 행씩 읽어오기

파이썬 실행 파일을 **관리자 권한**으로 실행해야 특정 경로의 파일을 정싱적으로 실행하거나 읽기 가능

### 네트워크 통신
#### 네트워크 통신 기초

###### 클라이언트와 서버 구조
- 서버(Server): 
  서비스를 제공하는 쪽, 여러 클라이언트 요청을 처리
- 클라이언트(Client): 
  서버에 요청을 보내고 응답을 받는 쪽
- 리스너(Listener): 
  서버에서 클라이언트의 접속 요청을 기다리는 역할 (길목을 지키는 문지기 비유 가능)
- 소켓(Socket): 
  클라이언트와 서버 양쪽 모두에서 반드시 생성되어야 하는 통신 장치

###### 통신 과정 요약
1. 서버는 소켓을 바인딩(binding)하여 IP와 포트 번호에 연결
2. 서버는 리스너로 클라이언트 접속 요청 대기
3. 클라이언트가 서버 IP와 포트로 접속 시도 (connect)
4. 서버는 accept 메서드로 클라이언트 접속 수락
5. 데이터 송수신 (send, recv 메서드 사용)
6. 작업 완료 후 자원 해제 (소켓 닫기)

#### 메시지 송수신 구현
###### 메시지 송수신 실습 개념 (파이썬 예시 기반)

| 단계          | 설명                                            |
| ----------- | --------------------------------------------- |
| 소켓 임포트      | `import socket`으로 소켓 라이브러리 호출                 |
| 서버 소켓 생성    | `socket.socket()`으로 소켓 생성, IP와 포트 바인딩, 리스너 대기 |
| 클라이언트 소켓 생성 | 서버 IP, 포트로 연결 시도 (`connect()`)                |
| 메시지 송신      | 클라이언트가 `send()`로 텍스트 메시지 전송 (바이너리 인코딩 필요)     |
| 메시지 수신      | 서버가 `recv()`로 메시지 수신 후 출력                     |
| 응답 송신       | 서버가 다시 `send()`로 응답 메시지 전송                    |
| 클라이언트 수신    | 클라이언트가 응답 메시지 수신 후 출력                         |
| 소켓 종료       | 양쪽 모두 작업 완료 후 소켓 자원 해제 (`close()`)            |

- 메시지는 바이너리 형태로 변환하여 전송 (`b'Hello'`)
- 작은 따옴표나 특수문자는 이스케이프 문자(`\`)로 처리

###### 파일 송수신 방법 (이미지, mp3 등 바이너리 파일)

| 역할    | 주요 작업                                                                             |
| ----- | --------------------------------------------------------------------------------- |
| 클라이언트 | 1. 서버 IP와 포트로 연결  <br>2. 전송할 파일을 바이너리(`rb`) 모드로 열기  <br>3. 파일 데이터를 소켓을 통해 서버로 전송  |
| 서버    | 1. 클라이언트 접속 수락  <br>2. 파일 받을 준비 (`wb` 바이너리 쓰기 모드)  <br>3. 클라이언트로부터 받은 데이터를 파일로 저장 |

- 파일 송수신 시 텍스트가 아닌 **바이너리 모드로 열고 읽기/쓰기** 해야 함
- 여러 클라이언트가 접속 가능 (서버 리스너 백로그 숫자 설정, 예: 5명)
- 클라이언트가 보낸 파일은 서버 지정 경로에 `파일명_버전번호` 형식으로 저장하여 중복 방지
- 전송 완료 후 소켓 종료

### DB와 SQLite
Python 기본 내장 경량 DB인 SQLite
SQLite3 모듈을 import하면 바로 사용 가능

```
Connection Open
	  |
  Cursor Open
	  |
   DB Work
      |
  Cursor Close
	  |
Connection Close
```

```python
import sqlite3
conn = sqlite3.connect('C:/.../database.db')
cursor = conn.cursor()
cursor.execute("INSERT INTO T_STU_INFO(ST_NAME, ST_CODE, ST_MAJ) VALUES('홍길동', 'ST001', '디자인')")

id = cursor.lastrowid
print(id)
conn.commit() # 변경 사항 저장 필요(트랜잭션 관련)

cursor.execute("SELECT * FROM T_STU_INFO")
rows = cursor.fetchAll()
for r in rows:
	print('ST_NAME: {0}, ST_CODE: {1}, ST_MAJ: {2}'.format(r[0], r[1], r[2]))

cursor.close()
conn.close()
```

###### MySQL과 연동하기
```python
import pymysql
conn = pymysql.connect(host='localhost', port=3306, user='root', password='password', db='student', charset='utf8')
cursor = conn.cursor()

cursor.execute("CREATE TABLE T_STU_INFO(ST_NAME CHAR(32), ST_CODE CHAR(32), ST_MAJ CHAR(32))")

cursor.close()
conn.close()
```

`pymysql` 모듈 설치 방법
`pip list` 입력, `pymysql` 없으면 install 가능
`pip3 install PyMySQL` -> `pip3 list` 확인



