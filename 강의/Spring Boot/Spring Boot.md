#Java 
#Spring 

---
### Spring Boot 프로젝트 생성
start.spring.io에서 생성


**개발 환경 설정**

1. 데이터베이스 설치
2. JDK 설치
3. 프로젝트 생성

- 스프링 부트 백엔드 프로젝트 생성
- Vite를 활용한 프론트엔드 프로젝트 생성

4. 데이터베이스 설정
5. 모듈셋팅
6. 이미지 셋팅
7. 파일 및 코드 정리

### 프론트엔드 프레임워크 설치

VueJS, Sass, Axios 설치
```bash
npm create vue@3.10.3
npm install
...

npm install -D sass@1.77.8

...

npm install axios@1.4.0
```

index.html
-> head에 추가
```vue
<script src="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.3/js/bootstrap.min.js"  
        integrity="sha512-ykZ1QQr0Jy/4ZkvKuqWn4iF3lqPZyij9iRv6sGqLRdTPkY69YX6+7wvVGmsdBbiIfN/8OdsI7HABjvEok6ZopQ=="  
        crossorigin="anonymous"  
        referrerpolicy="no-referrer"></script>
```

main.js
-> Pinia 관리, 마운트 시점 변경
```js
// 마운트 시점 변경(라우터의 초기 탐색 후) - Vue 라우터가 초기 작업을 완료한 시점으로 변경  
// 구현할 라우터 관련 기능이 중복 호출되지 않도록  
router.isReady().then(() => {  
    app.mount('#app');  
})
```

App.vue
-> 기본 세팅

index.js
-> router 설정
```js
import { createRouter, createWebHistory } from 'vue-router'  
  
const router = createRouter({  
  history: createWebHistory(import.meta.env.BASE_URL),  
  routes: [  
    {  
      path: '/',  
      name: 'home',  
      component: () => import('../views/Home.vue')  
    },  
  ]  
})  
  
export default router
```

stores/account.js
-> Pinia store 관리
```js
import { defineStore } from 'pinia'  
// 계정 데이터를 다루는 계정 스토어  
export const useAccountStore = defineStore('account', {});
```


---
### 프로젝트 구조
