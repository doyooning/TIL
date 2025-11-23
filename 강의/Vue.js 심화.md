#JS 
#Vue3

---
### 실전 페이지 구성

index.js 설계
```js
import { createRouter, createWebHistory } from "vue-router";
import HomeView from "../views/TheHomeView.vue";

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: "/",
      name: "home",
      component: HomeView,
    },
    {
      path: "/board",
      name: "board",
      component: () => import("../views/AppBoardView.vue"),
      redirect: { name: "article-list" },
      children: [
        {
          path: "list",
          name: "article-list",
          component: () => import("@/components/board/BoardList.vue"),
        },
		...
        {
          path: "modify/:articleno",
          name: "article-modify",
          component: () => import("@/components/board/BoardModify.vue"),
        },
      ],
    },
  ],
});
  
export default router;
```
vue-router로 각 컴포넌트들을 라우팅해줌
`createRouter` -> history, routes 지정
`routes` -> path, component 지정


main.js 설계
```js
import { createApp } from "vue";
import App from "./App.vue";
import router from "./router";
  
import "bootstrap/dist/css/bootstrap.min.css";
import "bootstrap";
  
const app = createApp(App);
  
app.use(router);
  
app.mount("#app");
```
createApp (기본 설정)
부트스트랩 import


App.vue 설계
```js
<script setup>
import Navbar from "./components/layout/TheHeadingNavbar.vue";
</script>
  
<template>
  <div>
    <Navbar />
    <router-view></router-view>
  </div>
</template>
  
<style scoped></style>
```
Navbar로 사용할 컴포넌트 import
메인 구성 : 네비게이션 바 + 라우팅한 각 컴포넌트들


TheHeadingNavbar.vue 설계
```js
<template>
  <nav class="navbar navbar-expand-lg bg-body-tertiary sticky-top">
    <div class="container-fluid">
      <router-link class="navbar-brand" :to="{ name: 'home' }">
        <img src="@/assets/mmcafe.png" class="rounded mx-auto d-block" id="logo" alt="..." />
      </router-link>
      <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarScroll"
        aria-controls="navbarScroll" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="navbarScroll">
        <ul class="navbar-nav me-auto ms-5 my-2 my-lg-0 navbar-nav-scroll" style="--bs-scroll-height: 100px">
          <li class="nav-item">
            <router-link :to="{ name: 'board' }" class="nav-link">커뮤니티공간</router-link>
            <!-- <a class="nav-link" href="#">커뮤니티공간</a> -->
          </li>
          <li class="nav-item dropdown">
            <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
              HELP DESK
            </a>
            <ul class="dropdown-menu">
              <li><a class="dropdown-item" href="#">공지사항</a></li>
              <li><a class="dropdown-item" href="#">FAQ</a></li>
              <li>
                <hr class="dropdown-divider" />
              </li>
              <li><a class="dropdown-item" href="#">매장이용안내</a></li>
            </ul>
          </li>
          <li class="nav-item">
            <a class="nav-link" href="#">단골매장소식</a>
          </li>
        </ul>
        <form class="d-flex" role="search">
          <input class="form-control me-2" type="search" placeholder="검색..." aria-label="Search" />
          <button class="btn btn-outline-success" type="button">search</button>
        </form>
      </div>
    </div>
  </nav>
</template>
```
화면 상단 네비게이션 바 구성


```js
<router-link class="navbar-brand" :to="{ name: 'home' }">
	<img src="@/assets/mmcafe.png" class="..." id="logo" alt="..." />
</router-link>
```
Composition API에서는 router-link 태그 사용해서 페이지 이동 구현
이동할 주소를 `:to="{ name: 'home' }"` 형식으로 지정

---
### 실전 페이지 구성 2

드롭다운 메뉴 구성
```vue
<template>
  <div>
    <slot name="icon"></slot>
    <slot name="label"></slot>
  </div>
</template>
```
드롭다운 메뉴를 구현한 템플릿을 가진 vue를 만들어준다.
구조는 slot 형식으로 구성
-> 메뉴 하나의 구성이 < 아이콘 + 메뉴이름 > 으로 이루어짐

드롭다운 메뉴는 상단 네비게이션 바에서 import해서 사용함

상단 네비게이션 바 - 드롭다운 메뉴 활용
```vue
<li>
	<router-link to="/r01/file" class="dropdown-item">
	    <DropDownIconMenuSlot>
		    <template v-slot:icon>
		        <IconFile></IconFile>
		    </template>
		    <template v-slot:label> 파일공유 </template>
	    </DropDownIconMenuSlot>
	</router-link>
</li>
```
router-link로 해당하는 vue 라우팅
드롭다운메뉴 태그 안에서 템플릿 -> v-slot으로 해당하는 태그 지정
여기서는 IconFile이라는 vue에 파일 모양 아이콘을 불러오는 템플릿을 담음(font-awesome)

아이콘 vue
```vue
<template>
  <svg xmlns="http://www.w3.org/2000/svg" height="1em" viewBox="0 0 384 512">
    <!--! Font Awesome Free 6.4.2 by @fontawesome - https://fontawesome.com License - https://fontawesome.com/license (Commercial License) Copyright 2023 Fonticons, Inc. -->
    <path
      d="M320 464c8.8 0 16-7.2 16-16V160H256c-17.7 0-32-14.3-32-32V48H64c-8.8 0-16 7.2-16 16V448c0 8.8 7.2 16 16 16H320zM0 64C0 28.7 28.7 0 64 0H229.5c17 0 33.3 6.7 45.3 18.7l90.5 90.5c12 12 18.7 28.3 18.7 45.3V448c0 35.3-28.7 64-64 64H64c-35.3 0-64-28.7-64-64V64z"
    />
  </svg>
</template>
```
웹 폰트 등을 import하는 방식 중에서 html로 가져오기하면 이런 식으로 태그의 속성을 통해서 import 가능


