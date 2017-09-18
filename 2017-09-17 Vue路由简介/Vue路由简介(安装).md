# Vue路由简介(安装)

### 直接下载/CDN
```
<script src="/path/to/vue.js"></script>
<script src="/path/to/vue-router.js"></script>
```

### NPM
```
npm install vue-router
```
如果在一个模块化工程中使用它，必须要通过`vue.use()`明确地安装路由功能:
```
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
```
如果使用全局的script标签，则无需如此（手动安装）

### 构建开发版
如果想使用最新的开发版本，就得从github上直接clone，然后自己build一个`vue-fouter`。
```
git clone https://github.com/vuejs/vue-router.git node_modules/vue-router
cd node_modules/vue-router
npm install
npm run build
```