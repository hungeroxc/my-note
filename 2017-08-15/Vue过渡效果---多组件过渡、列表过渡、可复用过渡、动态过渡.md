# Vue过渡效果---多组件过渡、列表过渡、可复用过渡、动态过渡


### 多组件过渡
- 介绍：
    - 多组件过渡可以使用**动态组件**
        - 动态组件利用组件的`is`特性，用以动态切换组件
    - 例子：
    ```
    <transition name="component-fade" mode="out-in">
        <component v-bind:is="view"></component>
    </transition>


    new Vue({
        el: '#transition-components-demo',
        data: {
            view: 'v-a'
        },
        components: {
            'v-a': {
            template: '<div>Component A</div>'
            },
            'v-b': {
            template: '<div>Component B</div>'
            }
        }
    })

    .component-fade-enter-active, .component-fade-leave-active {
        transition: opacity .3s ease;
    }
    .component-fade-enter, .component-fade-leave-to {
        opacity: 0;
    }
    ```

### 列表过渡
- 介绍：
    - 渲染整组列表可以使用`<transition-group>`组件
    - `<transition-group>`组件特点：
        - 该组件会以一个真实元素呈现： 默认是一个`<span>`。也可以通过`tag`属性更换成其他元素；
        - 内部元素**总是需要**提供唯一的`key`属性；
- 列表的进入和离开过渡例子（该例子存在一个问题：移动和删除元素的时候，周围元素会瞬间移动到他们的新布局的位置，而不是平滑过渡，解决这个问题可以使用`v-move`）
```
<div id="list-demo" class="demo">
  <button v-on:click="add">Add</button>
  <button v-on:click="remove">Remove</button>
  <transition-group name="list" tag="p">
    <span v-for="item in items" v-bind:key="item" class="list-item">
      {{ item }}
    </span>
  </transition-group>
</div>



.list-item {
  display: inline-block;
  margin-right: 10px;
}
.list-enter-active, .list-leave-active {
  transition: all 1s;
}
.list-enter, .list-leave-to
/* .list-leave-active for below version 2.1.8 */ {
  opacity: 0;
  transform: translateY(30px);
}



new Vue({
  el: '#list-demo',
  data: {
    items: [1,2,3,4,5,6,7,8,9],
    nextNum: 10
  },
  methods: {
    randomIndex: function () {
      return Math.floor(Math.random() * this.items.length)
    },
    add: function () {
      this.items.splice(this.randomIndex(), 0, this.nextNum++)
    },
    remove: function () {
      this.items.splice(this.randomIndex(), 1)
    },
  }
})
```
PS：`<transition-group>`组件有一个特殊之处，可以改变定位。使用这个功能可以使用`v-move`
- `v-move`
    - 介绍：
        - `v-move`会在元素的改变定位过程中应用。和之前的类名一样，可以通过`name`属性来自定义前缀，也可以通过`move-class`属性手动设置；
    - 用法：
    ```
    // 假定transition的name为list
    .list-move {
        transition: transform 1s;
    }
    ```
- 列表的渐进过渡
    - 介绍：
        - 通过data属性和JS通信，可以实现列表的渐进过渡
        ```
        <transition-group
            name="staggered-fade"
            tag="ul"
            v-bind:css="false"
            v-on:before-enter="beforeEnter"
            v-on:enter="enter"
            v-on:leave="leave"
        >
            <li
            v-for="(item, index) in computedList"
            v-bind:key="item.msg"
            v-bind:data-index="index"
            >{{ item.msg }}</li>
        </transition-group>
        ```
    - `v-bind:css="false"`：
        - 该属性可以使Vue不监测css（用style做过渡的属性设置时（或者使用其他的动画库时）可以使用）


### 可复用的过渡
    
### 动态过渡
- 介绍： 
    - 可以使用对`transition`组件的`name`属性进行动态切换来实现动态的过渡

