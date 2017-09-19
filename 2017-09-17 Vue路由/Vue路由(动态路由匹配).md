# 动态路由匹配

### 介绍
- 介绍
    - 我们经常需要将某种模式匹配到所有路由，全都映射到同一个组件。例如：我们有一个User组件，对于所有ID各不相同的用户，都要使用这个组件来渲染。那么，可以在`vue-router`的路由路径中使用[动态路径参数]来达到这个效果：
    ```
        const User = {
            template: '<div>User</div>'
        }

        const router = new VueRouter({
            routes: [
                // 动态路径参数，以冒号开头
                {path: '/user/:id', component: User}
            ]
        })
    ```
    现在，像`/user/foo`和`/user/bar`都将映射到相同的路由。
    一个[路径参数]使用冒号：标记。当匹配到一个路由时，参数会被设置到`this.$route.params`，可以在每个组件内使用。于是，可以更新`user`的模板，输出当前用户ID:
    ```
    const = {
        template: '<div>User{{ $route.params.id }}</div>'
    }
    ```
    可以在一个路由设置中设置多段[路径参数]，对应的值都会设置到`$doute.params`中。例如：
![image.png](http://upload-images.jianshu.io/upload_images/3360875-9ce75d91a6a081b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    除了`$route.params`外，`$route`对象还提供了其它有用的信息，例如，`$route.query`（如果URL中有查询参数）、`$route.hash`等等。
- 例子：
```
    <div id="app">
        <div id="app">
            <p>
                <router-link to="/user/foo">/user/foo</router-link>
                <router-link to="/user/bar">/user/bar</router-link>
            </p>
            <router-view></router-view>
        </div>
    </div>
    <script>
        const User = {
            template: `<div>User {{ $route.params.id }}</div>`
        }

        const router = new VueRouter({
            routes: [{
                path: '/user/:id',
                component: User
            }]
        })

        const app = new Vue({
            router
        }).$mount('#app')
    </script>
```
![image.png](http://upload-images.jianshu.io/upload_images/3360875-3a6f3362b6eb3b41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 响应路由参数的变化
- 介绍：
    - 当使用路由参数时，例如从`/user/foo`导航到`user/bar`, **原来的组件实例会被服用。**因为两个路由都渲染同一个组件，比起销毁再创建，复用则效果更加高效。**不过，这也意味着组件的生命钩子不会再被调用**。
    服用组件时，想对路由参数的变化做出相应的话，可以简单地watch（检测变化）`$route`对象：
    ```
    const User = {
        template: '...',
        watch: {
            '$route'(to, from){
                // 对路由变化做出相应....
            }
        }
    }
    ```

### 高级匹配模式
- 介绍：
    - `vue-router`使用`path-to-regexp`作为路径匹配引擎，所以支持很多高级的匹配模式，例如：可选的动态路径参数、匹配0到多个，甚至是自定义正则匹配。

### 匹配优先级
- 介绍： 
    - 有时候，同一个路径可以匹配多个路由，此时，匹配优先级就按照定义顺序，谁先定义，谁的优先级就更高