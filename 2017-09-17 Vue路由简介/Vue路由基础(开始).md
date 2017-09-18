### 简单实例
- 介绍：
    - 使用Vue.js + vue-router创建单页面应用，是非常简单的。使用Vue.js，我们已经可以通过组合组件来组成应用程序，当你要把vue-router添加进来，我们需要做的是：
        - 1.将组件(component)映射到路由(routes)
        - 2.告诉vue-router在哪里渲染他们
    - 例子：
    ```
    <div id="app">
        <h1>hello</h1>
        <p>
            <!--通过 router-link组件来导航  -->
            <!-- 通过to属性指定连接 -->
            <!-- <router-link>默认会被渲染成一个<a>标签 -->
            <router-link to="/foo">fo to foo</router-link>
            <router-link to="/bar">fo to bar</router-link>
        </p>
        <!-- 路由出口 -->
        <!-- 路由匹配到的组件将会渲染在这里 -->
        <router-view></router-view>
    </div>
    <script>
        // 0.如果使用模块化机制编程，导入vue和vue-router，要调用Vue.use(VueRouter)

        // 1.定义（路由）组件
        // 可以从其他文件import进来
        const Foo = {
            template: '<div>foo123123</div>'
        }
        const Bar = {
            template: '<div>bar123123</div>'
        }

        // 2.定义路由
        // 每个路由应该映射一个组件。其中"component"可以使通过Vue.extent()创建的组件构造器，
        // 或者，只是一个组件配置对象。

        const routes = [{
            path: '/foo',
            component: Foo
        }, {
            path: '/bar',
            component: Bar
        }]

        // 3.创建router实例，然后传'routes'配置
        // 还可以传别的配置参数
        const router = new VueRouter({
            routes // 缩写，相当于routes: routes
        })

        // 4.创建和挂在根实例。
        // 记得通过router配置参数注入路由，从而让整个应用都有路由功能。
        const app = new Vue({
            router
        }).$mount('#app')
    </script>
    ```

![image.png](http://upload-images.jianshu.io/upload_images/3360875-d2c3e219f1d16b1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/3360875-9f70a52c2b0aa3c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/3360875-39349c8cefe63153.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

PS: 要注意，当`<router-link></router-link>`对应的路由匹配成功，将自动设置class属性值。`router-link-active`.



