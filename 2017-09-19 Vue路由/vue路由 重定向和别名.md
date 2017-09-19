# vue路由 重定向和别名

### 重定向
- 介绍：
    - 重定向也是通过`routes`配置来完成，下面例子是从`/a`重定向到`/b`:
    ```
    const router = new VueRouter({
        routes: [{
            path: '/a',
            redirect: '/b'
        }]
    })
    ```
    - 重定向的目标也可以是一个命名的路由：
    ```
    const router = new VueRouter({
        routes: [{
            path: '/a',
            redirect: {name: 'foo'}
        }]
    })
    ```
    - 或者是一个方法，动态返回重定向目标：
    ```
    const router = new VueRouter({
        routes: [{
            path: '/a',
            redirect: to => {
                // 方法接受 目标路由 作为参数
                // return 重定向的 字符串路径/路径对象
            }
        }]
    })
    ```
- 例子
```
export default [{
    path: '/',
    name: 'hello',
    component: Hello
}, {
    path: '/oxc',
    name: 'oxc',
    redirect: {
        name: 'ooxc'
    }
}, {
    path: '/ooxc',
    name: 'ooxc',
    component: OOxc
}]

<button @click="aaa">123</button>

aaa(){
    this.$router.push({path: '/oxc'})
}
```
![image.png](http://upload-images.jianshu.io/upload_images/3360875-4894990fc0cce7c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](http://upload-images.jianshu.io/upload_images/3360875-dcfb87630c801e44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 别名
- 介绍： 
    - 重定向的意思是，当用户访问`/a`时，URL将会被第换成`/b`，然后匹配路由为`/b`，而别名的意思就是：
    `/a`的别名是`/b`，意味着，当用户访问`/b`时，URL会保持为`/b`，但是路由匹配则为`/a`，就像用户访问`/a`一样
    ```
    const router = new VueRouter({
        routes: [{
            path:'/a',
            component: A,
            alias: '/b'
        }]
    })
    ```
- 上述例子变换：
```
export default [{
    path: '/',
    name: 'hello',
    component: Hello
}, {
    path: '/oxc',
    component: Oxc,
    alias: '/ooxc'
}]


aaa(){
    this.$router.push({path: '/ooxc'})
}
```
![image.png](http://upload-images.jianshu.io/upload_images/3360875-65364f47a779b93a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](http://upload-images.jianshu.io/upload_images/3360875-804b716c7858ed53.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
