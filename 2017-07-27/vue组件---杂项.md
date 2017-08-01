# Vue 组件---杂项

### 动态组件
- 作用介绍：用以动态切换组件的显示；
- 用法：使用`is`关键字指定组件，实现其在同一挂载点上动态切换
```
    // 该例中，对select_component的切换可以控制组件的显示，使其公用一个挂载点
    <div id='app'>
        <child :is='select_component'></child>
    </div>
    <script>
        let vm = new Vue({
            el: '#app',
            data: {
                select_component: 'oxc'
            },
            components: {
                oxc: {
                    template: `<div>OXC</div>`
                },
                aaa: {
                    template: `<div>aaa</div>`
                }
            }
        })
    </script>
```
![image.png](http://upload-images.jianshu.io/upload_images/3360875-027da0c45229380e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- `keep-alive`简介
    - 作用：将切换出去的组件保留在内存中，保存其状态避免重新渲染，从而提升性能；
    - 实例
    ```
    // 本例中，当从oxc切换到aaa再切换回来时候，oxc组件渲染的是第一次渲染时保存在内存中的组件，并未重新渲染；
    <div id='app'>
        <keep-alive>
            <child :is='select_component'></child>
        </keep-alive>
    </div>
    <script>
        let vm = new Vue({
            el: '#app',
            data: {
                select_component: 'oxc'
            },
            components: {
                oxc: {
                    template: `<input>`
                },
                aaa: {
                    template: `<input>`
                }
            }
        })
    </script>
    ```

### 编写可复用的组件
- 方式
    - 1. 在编写组件时候需要留意该组件是否需要复用，若是可复用的组件则需要定义一个清晰的公开接口
    - 2. 要善于利用好`props`,`events`,`slots`三个api
    - 3. 利用好`v-bind`,`v-on`缩写
    - 4. 例子
    ```
    <my-component
        :foo='baz'
        :bar='qux'
        @event-a='dothis'
        @event-b='dothat'>
        <img slot='icon' src='...'>
        <p slot='main-text'></p>
    </my-component>
    ```

### 子组件索引
- 介绍
    - 作用：在JS中直接访问子组件。
    - 用法：可以使用`ref`为子组件指定一个索引
    - 实例
    ```
    <div id='app'>
        <oxc ref='oxc'></oxc>
    </div>
    <script>
        Vue.component('oxc', {
            template: `<div>大春春</div>`,
            data() {
                return {
                    msg: '大春春'
                }
            }
        })
        let app = new Vue({
            el: '#app'
        })
        let child = app.$refs.oxc
    </script>
    ```
![image.png](http://upload-images.jianshu.io/upload_images/3360875-47f3b78ffd8e4ce4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**PS：$refs不是响应式的，是在组件渲染完成之后才填充进去。只是作为一个访问子组件的应急方案，尽量避免在计算属性中使用它**

### 异步组件
- 介绍
    - 在大型应用中，需要将模块拆分成多个小模块，按需从服务器下载。因此，在Vue中允许将组件定义为一个工厂函数，动态解析组件定义。Vue只会在需要渲染的时候出发工厂函数，并且将结果缓存起来，用于后面的再次渲染。
- 用法
    - 工厂函数接受一个`resolve`回调，在收到从服务器下载的组件定义时调用。也可以在下载失败时调用`reject`提示加载失败
- 实例
```
// 本例子中的组件将会在3000ms后进行渲染
    <div id='app'>
        <oxc></oxc>
    </div>
    <script>
        Vue.component('oxc', (resolve, reject) => {
            setTimeout(function() {
                resolve({
                    template: '<div>大春春</div>'
                })
            }, 3000)
        })
        let app = new Vue({
            el: '#app'
        })
    </script>
```
- webpack的代码分割功能
    - 该章节将在webpack教程中进行介绍

### 高级异步组件
- 介绍
    - 从2.3.0开始，异步组件的工厂函数也可以返回一个如下对象：
- 实例
```
const AsyncComp = () => ({
  // 需要加载的组件. 应当是一个 Promise
  component: import('./MyComp.vue'),
  // loading 时应当渲染的组件
  loading: LoadingComp,
  // 出错时渲染的组件
  error: ErrorComp,
  // 渲染 loading 组件前的等待时间。默认：200ms.
  delay: 200,
  // 最长等待时间。超出此时间则渲染 error 组件。默认：Infinity
  timeout: 3000
})
```
- 注意：如果需要在vue-router的路由组件中使用，需要vue-router 2.4.0+版本


