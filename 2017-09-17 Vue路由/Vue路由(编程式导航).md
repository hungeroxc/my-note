# 编程式导航

除了使用`<router-link>`创建a标签来定义导航链接，我们还可以借助router的实例方法，通过编写代码来实现。

### `router.push(location)`
- 介绍：
    -想要导航到不同的url,则使用`router.push`方法。这个方法回想history栈添加一个新的记录，所以，房用户点击浏览器后退按钮时，则回到之前的URL.
    当你点击`<router-link>`时，这个方法会在内部调用，所以说，点击`<router-link :to="...">`等同于调用`router.push(...)`
    ![image.png](http://upload-images.jianshu.io/upload_images/3360875-78861625fa7a2d09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    该方法的参数可以是一个字符串路径，或者一个描述地址的对象。例如：
    ```
    // 字符串
    router.push('home')

    // 对象
    router.push({path: 'home'})

    // 命名的路由
    router.push({name: 'user', params: {userId: 123}})

    // 带查询参数，变成 /register?plan=private
    router.push({path: 'register', query: {plan: 'private'}})
    ```
- 例子
```
<button @click="router">123</button>
router(){
    this.$router.push({name: 'oxc', query: {id: 321}})
}
```
点击前
![image.png](http://upload-images.jianshu.io/upload_images/3360875-95a6c3c0255e09bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击后
![image.png](http://upload-images.jianshu.io/upload_images/3360875-f3a74d22b1b463e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击后退
![image.png](http://upload-images.jianshu.io/upload_images/3360875-06a36fd773157c8a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### `router.replace(location)`
与`router.push`很想，唯一不同的是，它不会向history添加新纪录，而是跟它的方法名一样--替换掉当前的`history`记录
![image.png](http://upload-images.jianshu.io/upload_images/3360875-02b5fda9837e4be2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### `router.go(n)`
这个方法的参数时一个整数，意思是在history记录中向前或者后退多少部，类似`window.history.go(n)`
例子：
```
// 在浏览器中前进一步，等同于history.forward()
router.go(1)
// 后退一步记录,等同于 history.back()
router.go(-1)
// 前进3步记录
router.go(3)
// 当history记录不够用时，该方法失效
router.go(100)
router.go(-100)
```

### 操作history
你也注意到`router.push`、`router.replace`、`router.go`和`window.history.pushState`、`window.history.replaceState`、`window.history.go`很想，实际上它们是消防`window.history API`的。




