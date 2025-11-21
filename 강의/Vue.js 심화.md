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



