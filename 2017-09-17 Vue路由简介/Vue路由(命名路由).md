# 命名路由

### 介绍：
    - 有时候，通过一个名称来标识一个路由显得方便些，特别是在链接一个路由，或者是执行一些跳转的时候。可以在创建Router实例的时候，在`routes`配置中给某个路由设置名称；
    ```
    const router = new VueRouter({
        routes: [{
            path: '/user/:userId',
            name: 'user',
            component: User
        }]
    })
    ```
    要链接到一个命名路由，可以给`<router-link>`的`to`属性传递一个对象：
    ```
    <router-link :to="{name: 'user', params: {userId: 123}}"></router-link>
    ```
    使用`router.push()`是一样的
    ```
    router.push({name: 'user', params: {userId: 123}})
    ```
    这两种方式都会把路由导航到/user/123路径