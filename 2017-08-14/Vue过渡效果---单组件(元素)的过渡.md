# Vue过渡效果

### 介绍
- 概述
Vue在插入、更新或者移除DOM时，提供多种不同的应用过渡效果，包括：
    - 1.在CSS过渡和动画中自动应用class（也就是写好过渡效果的css，然后对其进行动态切换）；
    - 2.可以使用第三方CSS动画库，比如Animate.css；
    - 3.在过渡钩子函数中使用JS直接操作DOM；
    - 4.可以配合使用第三方JS动画库，比如Velocity.js；

### 单元素/组件的过渡
- 介绍
    - Vue提供了`transition`的封装组件，在下列情况中，可以给人艺元素和组件添加`entering/leaving`过渡；
        - 1.条件渲染（使用`v-if`）;
        - 2.条件展示(使用`v-show`)；
        - 3.动态组件；
        - 4.组件根节点；
    - 例子：
    ```
    // 点击p元素淡入或淡出
    <style>
        .fade-enter-active,
        .fade-leave-active {
            transition: opacity .5s
        }
        
        .fade-enter,
        .fade-leave-to {
            opacity: 0
        }
    </style>
    <div id="demo">
        <button @click="show = !show">toggle</button>
        <transition name="fade">
            <p v-if="show">hello</p>
        </transition>
    </div>
    <script>
        let vm = new Vue({
            el: '#demo',
            data: {
                show: true
            }
        })
    </script>
    ```
    - 在插入和删除包含在`transition`组件中的元素时，Vue将会做以下处理：
        - 1. 自动嗅探目标元素是否应用了CSS过渡或动画，如果是，在恰当时机添加/删除CSS类名；
        - 2. 如果过渡组件提供了JS钩子函数，这些钩子函数将在恰当时机被调用。
        - 3. 如果没有找到JS钩子也没有检测到CSS过渡/动画，DOM操作（插入/删除）在下一帧立即执行

- 过渡的CSS类名与CSS过渡
    - 六种状态（会有6个css类名在enter/leave的过渡中切换）
        - 1.`v-enter`：定义进入过渡的开始状态。在元素被插入时生效，在下一帧移除；
        - 2.`v-enter-active`：定义过渡状态。在元素整个过渡过程中作用，在元素被插入时生效，在`transition/animation`完成之后移除。这个类可以被用来定义过渡的过程时间，延迟和曲线函数；
        - 3.`v-enter-to`：定义进入过渡的结束状态。在元素被插入一阵后生效（与此同时`v-enter`被删除），在`transition/annimation`完成之后移除；
        - 4.`v-leave`：定义离开过渡的开始状态。在离开过渡触发时生效，在下一帧移除。
        - 5.`v-leave-active`: 定义过渡的状态。在元素整个过程中作用，在离开过渡被触发后生效，在`transition/animation`完成之后移除。这个类可以被用来定义过渡的过程时间，延迟和取消函数。
        - 6.`v-leave-to`：定义离开过渡的结束状态。在离开过渡被触发一帧后生效（与此同时`v-leave`被删除），在`transition/animation`完成后被移除；
        - 注意：上述例子的`v-`是这些类名的前缀，使用`<transition name="XXX">`可以重置前缀，例如`XXX-enter`
![image.png](http://upload-images.jianshu.io/upload_images/3360875-2f35da020017a29f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
        - 例子
        ```
        // 渲染时的初始状态
        .fade-enter {
            opacity: 0;
        }
        // 渲染时的过渡状态
        .fade-enter-active {
            transition: opacity 10s;
        }
        // 渲染完成后的状态
        .fade-enter-to {
            opacity: 1
        }
        // 离开时的初始状态
        .fade-leave {
            opacity: 1
        }
        // 离开时的过渡状态
        .fade-leave-active {
            transition: opacity 3s;
        }
        // 离开后的状态
        .fade-leave-to {
            opacity: 0
        }
        ```

- CSS动画
    - 介绍：
        - CSS动画用法和CSS过渡，区别在于动画中`v-enter`类名在节点插入DOM后不会立即删除，而是在`animationend`时间触发时删除；
        - 用法：在CSS中定义transition的name类名的`enter/leave`状态，并在其中定义`keyframe`动画过程
        - 例子：
        ```
        .bounce-enter-active {
            animation: bounce-in 1s;
        }
        
        .bounce-leave-active {
            animation: bounce-in 1s reverse;
        }
        
        @keyframes bounce-in {
            0% {
                transform: scale(0);
            }
            50% {
                transform: scale(1.5);
            }
            100% {
                transform: scale(1);
            }
        }
        ```
- JS钩子
    - 介绍：
        - 可以在`transition`组件的属性中声明JS钩子
        ```
        <transition
        // 进入过渡前回调
        @before-enter="beforeEnter"
        // 初始化过渡状态回调
        @enter="enter"
        // 过渡结束后状态回调
        @after-enter="afterEnter"
        // 取消过渡状态回调
        @enter-cancelled="enter"
        // leave状态同上
        @before-leave="beforeLeave"
        @leave="leave"
        @after-leave="afterLeave"
        // leave-cancelled只用于v-show中
        @leave-cancelled="leaveCancelled">
        </transition>

        methods: {
            beforeEnter(el){
                ....
            },
            enter(el, done){
                .....
                done()
            }
            .....
            leave(el, done){
                .....
                done()
            }
        }
        ```
    - 注意事项：
        - 当只使用JS钩子进行过渡时，done回调函数是必须的，否则，他们会被同步调用，过渡会立即完成;


### 初始渲染的过渡
- 介绍：
    - 可以通过`appear`特性设置节点的在初始渲染的过渡

### 多个元素的过渡
- 介绍：
    - 对于原生标签可以使用`v-if/v-else`（相当于为每个标签分别写过渡效果）。最常见的多标签过渡是一个列表和描述这个列表为空消息的元素；
- 例子：
    ```
    <div id="demo">
        <transition>
            <table v-if="item.length > 0">
                .....
            </table>
            <p v-else>Sorry , no items found.</p>
        </transition>
    </div>
    ```
- 相同标签名：
    - 相同标签名的情况下，需要通过设置`key`特性的唯一值标记来让Vue区分它们，否则Vue为了效率只会替换标签内部的内容。
    - 例子：
    ```
        <transition>
            <button v-if="isEditing" key="save">Save</button>
            <button v-else key="edit">Edit</button>
        </transition>
    ```
- 过渡模式
    - 问题：
        - 在`transition`组件中，有一个默认行为是`enter`和`leave`状态是同时开始进入过渡的。那么就有可能造成DOM结构的崩溃
    - 解决方案：过渡模式（制定新元素与当前元素进入过渡状态的顺序）
        - 1.`in-out`：新元素先进行过渡，完成后当前元素过渡离开；
        - 2.`out-in`：当前元素先进行过渡，完成后新元素进入；
        ```
        用法
        <transition name="fade" mode="out-in"></transition>
        ```