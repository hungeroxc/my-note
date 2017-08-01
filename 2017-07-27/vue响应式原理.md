# Vue响应式原理

### 介绍
Vue最显著的特性之一就是响应式系统。
模型层(model)只是普通的JS对象，修改它则会更新视图层(view)。
这让状态管理变得非常简单并且直观。

### 如何追踪变化
- 现象介绍
    - 将一个普通的JS对象传递给Vue实例的date选项，Vue将遍历此对象所有的属性，并且使用`Object.defineProperty`把这些属性全部转变为`getter/setter`。`Object.defineProperty`只有ES5支持。所以Vue并不支持IE8及其以下版本浏览器。
- 原理
    - `getter/setter`让Vue在内部追踪变化，在属性被访问和修改时候通知变化。
    - 每个组件实例都有相应的`watcher`实例对象，它会在组件渲染的过程中把属性记录为依赖，之后当依赖项的`setter`被调用时候，会通知`watcher`重新计算，从而致使它关联的组件得以更新。
![image.png](http://upload-images.jianshu.io/upload_images/3360875-df243b1cd242dcfd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 关于`Object.defineProperty`
- 介绍
    - `Object.defineProperty`方法会直接在一个对象上定义一个新属性，或者修改一个对象的现在属性，并返回这个对象。
    - 简单来说，原本定义一个对象中的属性是通过**赋值**实现的，现在多了一个`Object.defineProperty()`方法
    ```
    // 以下操作两者结果相同
    let obj = {  }
    // 赋值定义
    obj.name = 'oxc'
    obj['name'] = 'oxc'
    // Object.defineProperty()定义
    Object.defineProperty(obj, 'name', {
        value: 'oxc'
    })
    ```
- 语法
    - `Object.defineProperty(obj, prop, descriptor)`
    - 参数：
        - obj: 要在其上定义属性的对象(必须)
        - prop: 要定义或者修改的属性的名称(必须)
        - descriptor: 将被定义或者修改的属性的描述符(必须)
    - 关于第三个参数descriptor，他可以接受的值如下：
        - value: 属性的值，默认为undefined；
        - writable: 传入的属性是否可以被改写，默认为true，若改为false则不能被改写（对该属性的任何操作都无效，但不会报错）；
        ```
        let obj = {
            time: ''
        }
        Object.defineProperty(obj, 'time', {
            writable: false,
            value: '789'
        })
        console.log(obj.time)   // 789
        obj.time = '1000000'
        console.log(obj.time)   // 789
        ```
        - configurable: 该属性决定属性是否可以被删除，默认为true，如果改为false则不能被删除，并且如果原来对象上如果没有这个属性，则该属性值不能被修改， 同时决定除writable特性外的其他特性是否可被修改;
        ```
        let obj = { }
        Object.defineProperty(obj, 'time', {
            configurable: false,
            value: '789'
        })
        console.log(obj.time)   //  789
        delete obj.time
        console.log(obj.time)   // 789
        obj.time = 'oxc'
        console.log(obj.time)   // 789
        ```
        - enumerable: 改属性决定是否能在`for...in`循环中遍历出来或者在`Object.keys()`中列举出来, 默认为true
        ```
        let obj = {
            name: '123'
        }
        Object.defineProperty(obj, 'time', {
            enumerable: false,
            value: '789'
        })
        for (k in obj) {
            console.log(obj[k])    // 只打印了一个123出来
        }
        ```
        - get（函数）：一旦目标属性被访问，就会调用这个函数;
        - set（函数）：一旦目标属性被设置，就会调用这个函数;
        ```
        let obj = {
            time: '123'
        }
        Object.defineProperty(obj, 'time', {
            get: function() {
                console.log('我被调用了')
            },
            set: function(value) {
                console.log('我被设置了')
            }
        })
        console.log(obj.time)
        obj.time = 100
        ```
        ![image.png](http://upload-images.jianshu.io/upload_images/3360875-9b5b52b697f8e0f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    - 注意事项： 在调用`Object.defineProperty()`的时候，`writable`, `configurable`, `enumerable`默认都是false, 也就是说，调用该方法创建的属性不可更改，不可删除，无法遍历或枚举出来；

- 数据描述符和存取描述符
    - 介绍：对象里目前存在的属性描述符有两种：数据描述符和属性存取符；
    - 数据描述符：是一个可写或不可写的属性；
    - 存储描述符：是由一对`getter-setter`函数功能来描述的属性。
    - 描述符只能是两者之一而不能同时为两者， 也不能混合使用， 例如：`get`和`value`不能同时使用；
    - 只有`value`，`writable`的属性是数据描述符；
    - 有`get`, `set`, `writable`, `configurable`, `enumerable`, 是存取描述符；
    - 定义属性时候，如果没有`get/set/value/writable`, 则这个属性被归类为数据描述符；

- 两种情况（对象本身拥有该属性，对象本身没有该属性）
    - 拥有该属性
        - 如果该属性已经存在，`Object.defineProperty()`将尝试根据描述符中的值以及对象当前的配置来修改这个属性。
        - 如果描述符的`configurable`特性为false，那么除了`writable`外，其他特性都不能被修改，并且数据和存储描述符也不能相互切换;
        - 如果一个属性的`configurable`为false，则其writable特性也只能修改为false；
    - 没有该属性
        - 如果对象中不存在该属性，`Object.defineProperty()`会先创建该属性。若描述符中省略了某些参数，则这些参数使用默认值。拥有布尔值的默认值都是false（`writable`, `configurable`, `enumerable`）, `get`和`set`则为`undefined`。    
    - 对比
    ```
    let o = {}
    o.a = 1
    // 等同于
    Object.defineProperty(o, "a", {
        Value: 1,
        writable: true,
        configurable: true,
        enumerable: true
    })

    // 使用Object.defineProperty定义的属性，默认是不可修改，不可删除和不可遍历的
    Object.defineProperty(o, "a", {Value: 1})
    // 等同于
    Object.defineProperty(o, "a", {
        Value: 1,
        writable: false,
        configurable: false,
        enumerable: false
    })
    ```
    ### 变化检测问题
    - Vue能够检测到变化的属性
        - Vue在初始化实例的时候会对data对象上的执行getter/setter转化，所以属性必须在初始化时存在于data对象上，初始化过后添加的属性不能被检测到；
        ```
        <div id="app">
            <div>{{ msg }}</div>       // 显示123， 但是通过修改vm.msg = 456并不能响应变化
        </div>
        <script>
            let vm = new Vue({
                el: '#app',
                created() {
                    this.msg = 123
                }
            })
        </script>
        ```
        - `$set(obj, key, value)`:
            - 介绍：
                - 虽然直接在实例创建后使用数据表示符对其data对象进行属性添加，添加的属性不能追踪变化，但是可以先在data对象中添加一个对象，然后使用`vm.$set(obj, key, value)`在这个对象中进行属性和值得添加，这样添加的属性是响应式的, 可以实现属性的动态添加;
            - 例子：
                ```
                <div id="app">
                    <div>{{ obj.msg }}</div>
                </div>
                <script>
                    let vm = new Vue({
                        el: '#app',
                        data: {
                            obj: {}
                        },
                        created() {
                            this.$set(this.obj, 'msg', 123)
                        }
                    })
                </script>
                ```  
                ![image.png](http://upload-images.jianshu.io/upload_images/3360875-882cac18e8503ae5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
                ![image.png](http://upload-images.jianshu.io/upload_images/3360875-b367f8530abfa244.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
                ![image.png](http://upload-images.jianshu.io/upload_images/3360875-2908f81b5f205a12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
            - 另一种写法
                - `Vue.set(vm.obj, 'msg', 123)`
        - 注意事项
            - 介绍：有时候在已有对象上会使用`Object.assign(target, ...sources)`进行新属性的添加，这样添加的新属性并不是响应式的，例如:
            ```
            <div id="app">
                <div>{{ obj.a }}</div>           // 显示123
            </div>
            <script>
            let vm = new Vue({
                el: '#app',
                data: {
                    obj: {

                    }
                },
                created() {
                    Object.assign(this.obj, {
                        a: 123
                    })
                }
            })
            vm.obj.a = 345                       // 还是显示123, 并没有改变
            </script>
            ```
            - 解决方案
                - 应该创建一个新的对象，然后将已有对象的属性和新添加的属性都添加到新属性中，然后将新对象赋值给目标对象，或者在在原有对象中添加新属性后深拷贝一次；
                ```
                let vm = new Vue({
                    el: '#app',
                    data: {
                        obj: {

                        }
                    },
                    created() {
                        this.obj = Object.assign({}, this.obj, {
                            a: 123
                        })
                    }
                })
                ```
    - 声明响应式属性
        - 由于Vue不允许动态添加根级响应式元素， 所以最好在实例创建之前就声明根级响应式属性，没有值得可以先用空值代替；





