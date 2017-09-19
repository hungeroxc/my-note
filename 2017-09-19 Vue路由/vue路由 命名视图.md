# 命名视图

### 介绍
    - 介绍：有时候想同时(同级)展示多个视图，而不是嵌套展示，例如创建一个布局，有`sidebar`(侧导航)和`main`(主内容)两个视图，这个时候命名视图就派上用场了。可以在界面中拥有多个单独命名的视图，而不是只有一个单独出口。如果`router-view`没哟适名字，那么默认为default.
    ```
    <router-view class="view one"></router-view>
    <router-view class="view two" name="a"></router-view>
    <router-view class="view three" name="b"></router-view>
    ```
    一个视图使用一个组件渲染，因此对于同个路由，多个视图就需要多个组件。确保正确使用`components`配置（带上s是因为可能有多个）：
    ```
    const router = new VueRouter({
        routes: [{
            path: '/',
            components: {
                default: Foo,
                a: bar,
                b: foo
            }
        }]
    })
    ```