# Vue组件---slot分发内容

### slot解决的问题
- 介绍
    - 问题：在使用组件的时候，经常向下面这样组合他们：
    ```
    <app>
        <app-header></app-header>
        <app-footer></app-footer>
    </app>
    // 这时候会有以下两点需要注意：
    1. <app>组件不知道他会接受什么内容。这是由<app>的父组件决定的；
    2. <app>组件很可能有他自己的模板；
    ```
    这时候问题就出现了：为了让组件可以进行组合，需要一种方式混合父组件内容和子组件自己的模板。这个过程叫做内容分发。Vue使用了符合规范的`<slot>`作为原始内容的插槽;

- 编译作用域
    - 介绍：在使用`slot`之前我们需要先明确内容是在哪个作用域里编译的，如下:
    ```
    <child-comp>
        {{ msg }}
    </child-comp>
    // 在本例子中，msg应该是绑定在父组件的数据，而不是子组件的数据。
    ```
    所以组件作用域就是说：在父组件模板的内容在父组件中编译，子组件中的内容在子组件中编译，形成解耦。
    - 常见错误
    ```
    // 常见错误之一就是将一个指令绑定到子组件的属性/ 方法，这种情况下Vue会报错:显示没有找到相关数据
    <div id='app'>
        <oxc v-show='isShow'></oxc>
    </div>
    <script>
        Vue.component('oxc', {
            template: '<div v-show="isShow">大春春</div>',
            data() {
                return {
                    isShow: true
                }
            }
        })
        new Vue({
            el: '#app'
        })
    </script>
    // 原因是因为在父组件模板编译的时候并未读到相关数据（因为不知道子组件的状态），导致出现undefined
    // 改进方案：将数据放到父组件模板上即可，由父组件来进行数据绑定
    new Vue({
        el: '#app',
        data: {
            isShow: true
        }
    })
    ```   

    **与此一样，内容分发需要在父组件作用域内编译**

![image.png](http://upload-images.jianshu.io/upload_images/3360875-3da74cf9accb1190.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### slot的使用
- 单个slot
     - 使用方法：在子组件模板中写入一个`<slot>`标签，然后在父组件中引用的子组件标签中进行DOM的插入，子组件模板编译时会用父组件插入的DOM内容替换掉`<slot>`标签
     - 使用实例
     ```
    <div id='app'>
        <oxc>
            <div v-show='isShow'>大春春</div>
        </oxc>
    </div>
    <script>
        Vue.component('oxc', {
            template: `<div>
                            <slot></slot>
                        </div>`
        })
        new Vue({
            el: '#app',
            data: {
                isShow: true
            }
        })
    </script>
    // 最终渲染结果
    <div id='app'>
        <div>
            <div v-show='isShow'>大春春</div>
        </div>
    </div>
    // 在该例子中，可以在父组件作用域的数据中对子组件插槽中的内容进行显示控制
     ```

- 具名slot(slot的name属性)
    - 介绍：`<slot>`标签有一个name属性，可以使用其来配置如何分发内容，多个slot可以有不同的名字。具名slot将匹配内容片段中有对应的slot特性的元素；
    - 注意：依然可以有一个匿名`<slot>`，这是一个默认的slot，作为找不到匹配内容片段的备用插槽，如果没有这个默认的slot，那么找不到匹配的内容片段将会被丢弃
    - 使用实例
    ```
    <div id='app'>
        <oxc>
            <div slot='name'>大春春</div>
            <div slot='age'>24</div>
            <div slot='sex'>男</div>
            <div>我叫大春春</div>
        </oxc>
    </div>
    <script>
        Vue.component('oxc', {
            template: `<div>
                            <div>
                                <slot name='name'></slot>
                            </div>
                            <div>
                                <slot name='sex'></slot>
                            </div>            
                            <div>
                                <slot name='age'></slot>
                            </div>
                            <slot></slot>
                        </div>
                        `
        })
        new Vue({
            el: '#app'
        })
    </script>
    // 渲染结果是
    <div id="app">
        <div>
            <div>大春春</div>
        </div>
        <div>
            <div>男</div>
        </div>
        <div>
            <div>24</div>
        </div>
        <div>
            <div>我叫大春春</div>
        </div>
    </div>
    ```
![image.png](http://upload-images.jianshu.io/upload_images/3360875-0b500c54ab314800.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 作用域插槽
    - 介绍：作用域插槽是一种特殊类型的插槽，用作使用一个（能够传递数据到）可重用模板替换已渲染元素。(与上面的slot使用不同的是：模板编译的作用域由父组件切换到了子组件)
    - 使用注意事项：
        - 1. 具有特殊属性`scope`的`<template>`元素必须存在，表示它是作用域插槽的模板。`scope`的值对应一个临时变量名，此变量名就收从子组件中传递的`props`对象；
        - 2. 作用域插槽也可以是具名的
    - 使用例子
    ```
    // 将父组件的列表数据用props传递给子组件的插槽，并且该列表数据的作用域实在子组件中的
    <div id='app'>
        <oxc :items='items'>
            <template slot='item' scope='props'>
                <span>{{ props.text }}</span>
            </template>
        </oxc>
    </div>
    <script>
        Vue.component('oxc', {
            template: `<div>
                            <slot name='item' v-for='item in items' :text='item.text'></slot>
                        </div>
                        `,
            props: ['items']
        })
        new Vue({
            el: '#app',
            data: {
                items: [{
                    text: '大春春'
                }, {
                    text: 'oxc'
                }, {
                    text: 'age'
                }]
            }
        })
    </script>
    ```
![image.png](http://upload-images.jianshu.io/upload_images/3360875-f3212db7109fb05f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





    