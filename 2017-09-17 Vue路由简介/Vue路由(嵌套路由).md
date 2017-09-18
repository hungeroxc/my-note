# Vue路由之嵌套路由

实际生活中的应用界面，通常由多层嵌套的组件组合而成。同样地，URL中各段动态路径也按照某种结构对应嵌套的各层组件，例如：
![image.png](http://upload-images.jianshu.io/upload_images/3360875-40c6608ff5bb9cec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
借助`vue-router`，使用嵌套路由配置，就可以很清楚地表达这种关系。
```
<div id="app">
    <router-view></router-view>
</div>

const User = {
    template: '<div>User {{ $route.params.id }}</div>'
}

const router = new VueRouter({
    routes: [{
        path: '/user/:id', 
        component: User
    }]
})
```
这里的`<router-view></router-view>`是最顶层的出口，渲染最高级路由匹配到的组件。同样第，一个被渲染组件同样可以包含自己的嵌套`<router-view>`.例如，在`User`组件的模板添加一个`<router-view>`
```
const User = {
    template: `
        <div class="user">
            <h2>User {{ $route.params.id }}</h2>
            <router-view></router-view>
        </div>
    `
}
```
要在嵌套的出口中渲染组件，需要在`VueRouter`的参数中使用`children`配置：
```
const router = new VueRouter({
    routes: [{
            path: '/user/:id', 
            component: User,
            children: [{
                // 当/user/:id/profile 匹配成功,
                // UserProfile会被渲染在User的<router-view></router-view>中
                path: 'profile',
                component: UserProfile
            },{
                // 当/user/:id/posts 匹配成功,
                // UserPosts会被渲染在User的<router-view></router-view>中
                path: 'posts',
                component: UserPosts
            }]
        ]
})
```
要注意，以/开头的嵌套路径会被当做根路径。这让你充分的使用嵌套组件而无需设置嵌套的路径。

你会发现，`children`配置就是像`routes`配置一样的路由配置数组，所以可以嵌套多层路由。

此时，基于上面的配置，当你访问`/user/foo`时，`User`的出口是不会渲染任何东西，这是因为没有匹配到合适的子路由。如果想要渲染点什么，可以提供一个空的自路由：

```
const router = new VueRouter({
    route: [{
        path: 'user/:id',
        component: User,
        children: [
            // 当/user/:id匹配成功,
            // UserHome会被渲染在User的<router-view></router-view>中
            {path: '', component: UserHome},

            // 其它子路由
        ]
    }]
})
```

完整例子：
```
    <div id="app">
        <p>
            <router-link to="/user/foo">/user/foo</router-link>
            <router-link to="/user/foo/profile">/user/foo/profile</router-link>
            <router-link to="/user/foo/posts">/user/foo/posts</router-link>
        </p>
        <router-view></router-view>
    </div>
    <script>
        const User = {
            template: `
            <div class="user">
                <h2>User {{ $route.params.id }}</h2>
                <router-view></router-view>
            </div>
        `
        }

        const UserHome = {
            template: '<div>Home</div>'
        }
        const UserProfile = {
            template: '<div>Profile</div>'
        }
        const UserPosts = {
            template: '<div>Posts</div>'
        }

        const router = new VueRouter({
            routes: [{
                path: '/user/:id',
                component: User,
                children: [{
                    path: '',
                    component: UserHome
                }, {
                    path: 'profile',
                    component: UserProfile
                }, {
                    path: 'posts',
                    component: UserPosts
                }]
            }]
        })

        const app = new Vue({
            router
        }).$mount('#app')
    </script>
```
![image.png](http://upload-images.jianshu.io/upload_images/3360875-1c822da33de5e229.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)