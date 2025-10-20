#JS 

---
# 동기, 비동기 프로그래밍
동기 프로그래밍
: 작업이 차례로 실행되며 이전 작업이 완료될 때까지 다음 작업이 진행될 수 없음

비동기 프로그래밍
: 임의의 순서로 또는 동시에 작업이 실행되는 형태

자바스크립트는 런타임(브라우저나 NodeJs)에서 싱글 스레드로 동작
싱글 스레드로 동작하지만 Callback, Promise, Async, Await 방법으로 비동기 구현

###### Promise 객체
: 콜백 대신 사용하는 방법
비동기 작업이 완료되면 결과를 반환하는 객체

###### Async, Await
: Promise 객체를 사용하는 비동기 작업을 동기적으로 처리하는 키워드

# JSON 서버 생성하기
Node.js : 브라우저 기반의 JS를 로컬에서 실행
Npm : Node.js 패키지를 모아서 관리하는 저장소

Git Bash를 열고 입력:

```
mkdir json-server-exam && cd json-server-exam
```
-> json-server-exam 디렉토리를 만들고 열기

```
npm init -y
```
-> npm 시작, package.json 생성됨

```
npm install json-server --save-dev
```
-> json-server 설치

설치 디렉토리에 db.json 생성:
```
{
  "todos": [
    {
      "id": "1",
      "content": "HTML",
      "completed": true
    },
    {
      "id": "2",
      "content": "React",
      "completed": false
    },
    {
      "id": "3",
      "content": "JAVA",
      "completed": true
    },
    {
      "id": 5,
      "content": "Spring",
      "complete": false
    }
  ],
  "users": [
    {
      "id": "1",
      "name": "Lee",
      "role": "developer"
    },
    {
      "id": "2",
      "name": "Kim",
      "role": "designer"
    }
  ]
}
```
-> json 객체 만들어줌

package.json 열어서 scripts에 추가:
`"start": "json-server --watch db.json -p 3000",`
-> 서버 시작시 db.json 보여주고 포트는 3000번

```
npm run start
```
-> 서버 시작

localhost:3000/todos에서 확인 가능

# XHR과 HTTP 요청 실습
서버 설치 디렉토리에 public 디렉토리 추가

get_index.html
```
<body>
    <pre></pre>
    <script>
      const xhr = new XMLHttpRequest();
      // HTTP 요청 초기화
      // todos 리소스에서 모든 todo를 취득(index)
  
      xhr.open('GET', '/todos');
  
      // HTTP 요청 전송
      xhr.send();
  
      //xhr.onload 이벤트 이용해서 요청이 성공적으로 완료된 경우 발생
      xhr.onload = () => {
        if (xhr.status === 200) {
          document.querySelector('pre').textContent = xhr.response;
        } else {
          console.error('Error', xhr.status, xhr.statusText);
        }
      };
    </script>
  </body>
```
-> localhost:3000/todos를 찾으면 모든 todo를 보여줌

`xhr.open('GET', '/todos/1');` : todos의 1번 항목을 보여줌( "id": "1")

post.html
```
<body>
    <pre></pre>
    <script>
      const xhr = new XMLHttpRequest();
      // HTTP 요청 초기화
      // todos 리소스에서 새로운 todo 생성

      xhr.open('POST', '/todos');

      // 요청 담을 MIME 타입을 지정
      xhr.setRequestHeader('content-type', 'application/json');
  
      // HTTP 요청 전송
      xhr.send(JSON.stringify({ id: '6', content: 'CSS', complete: false }));
  
      //xhr.onload 이벤트 이용해서 요청이 성공적으로 완료된 경우 발생
      xhr.onload = () => {
        if (xhr.status === 200 || xhr.status === 201) {
          document.querySelector('pre').textContent = xhr.response;
        } else {
          console.error('Error', xhr.status, xhr.statusText);
        }
      };
    </script>
  </body>
```
-> 새로운 데이터 추가
`(id: '6')` : 숫자에 따옴표 붙여야 URI로서 검색 가능(/todos/6)

put.html
```
  <body>
    <pre></pre>
    <script>
      const xhr = new XMLHttpRequest();
      // HTTP 요청 초기화
      // todos 리소스에서 todos의 todo를 id를 제외하고 수정
  
      xhr.open('PUT', '/todos/5');
  
      // 요청 담을 MIME 타입을 지정
      xhr.setRequestHeader('content-type', 'application/json');
  
      // HTTP 요청 전송
      xhr.send(JSON.stringify({ id: 5, content: 'Python', complete: true }));
  
      //xhr.onload 이벤트 이용해서 요청이 성공적으로 완료된 경우 발생
      xhr.onload = () => {
        if (xhr.status === 200) {
          document.querySelector('pre').textContent = xhr.response;
        } else {
          console.error('Error', xhr.status, xhr.statusText);
        }
      };
    </script>
  </body>
```
-> id 제외 내용 수정

patch.html
```
  <body>
    <pre></pre>
    <script>
      const xhr = new XMLHttpRequest();
      // HTTP 요청 초기화
      // todos 리소스에서 id로 todo를 정하여 complete 속성만 수정
  
      xhr.open('PATCH', '/todos/5');
  
      // 요청 담을 MIME 타입을 지정
      xhr.setRequestHeader('content-type', 'application/json');
  
      // HTTP 요청 전송
      xhr.send(JSON.stringify({ id: 5, content: 'React', complete: false }));
  
      //xhr.onload 이벤트 이용해서 요청이 성공적으로 완료된 경우 발생
      xhr.onload = () => {
        if (xhr.status === 200 || xhr.status === 201) {
          document.querySelector('pre').textContent = xhr.response;
        } else {
          console.error('Error', xhr.status, xhr.statusText);
        }
      };
    </script>
  </body>
```
-> 5번 id의 리소스를 complete만 수정

delete.html
```
<body>
    <pre></pre>
    <script>
      const xhr = new XMLHttpRequest();
      // HTTP 요청 초기화
      // todos 리소스에서 id로 todo를 정하여 complete 속성만 수정
  
      xhr.open('DELETE', '/todos/6');
  
      // HTTP 요청 전송
      xhr.send();
  
      //xhr.onload 이벤트 이용해서 요청이 성공적으로 완료된 경우 발생
      xhr.onload = () => {
        if (xhr.status === 200 || xhr.status === 201) {
          document.querySelector('pre').textContent = xhr.response;
        } else {
          console.error('Error', xhr.status, xhr.statusText);
        }
      };
    </script>
  </body>
```
-> todos의 6번 항목을 삭제

