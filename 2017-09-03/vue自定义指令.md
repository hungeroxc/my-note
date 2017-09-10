# Vue自定义指令

### 简介：
- 除了设置默认的核心指令(`v-model`和`v-show`等)，Vue也允许注册自定义指令。注意，在Vue2.0里面，代码服用的主要形式和抽象是组件--然而，有的情况下，你仍然需要对纯DOM元素进行底层操作，这时候就会用到自定义指令。
- 例子：当聚焦一个input元素的时候
```
// 注册一个全局自定义指令v-focus
Vue.directive('focus', {
    inserted: function(el) {
        el.focus()
    }
})
```

### 钩子函数
- 介绍：指令的定义函数提供了几个钩子函数（可选）：
    - bind: 之调用一次，指令的第一次绑定到元素时调用，用这个钩子函数可以定义一个在绑定时执行一次的初始化动作；
    - inserted：被绑定元素插入父节点时电泳（父节点存在即可调用，不必存在于document中）；
    - update：所在组件的VNode更新时调用，**但是可能发生在其孩子的VNode更新之前。**指令的值可能发生了改变也可能没有。但是你可以通过比较更新前后的值来忽略不必要的模板更新（详细钩子函数参数见下）；
    - componentUpdated：所在组件的VNode及其孩子的VNode全部更新时调用。
    - unbind：只调用一次，在指令和元素解绑时调用；

- 钩子函数参数
    - bind: 只调用一次，指令第一次绑定到元素时调用，用这个钩子函数可以定义一个在绑定时执行一次的初始化动作；
    - inserted: 被绑定元素插入父节点时调用（父节点存在即可调用，不必存在于document中）；
    - update: 所在组件的VNode更新时调用，但是可能发生在其孩子的VNode更新之前。指令的值可能发生了改变也可能没有。但是可以通过比较更新前后的值来忽略不必要的模板更新
    - componentUpdated: 所在组件的VNode及其孩子的VNode全部更新时调用。
    - unbind: 只调用一次，指令与元素解绑时调用。

### 钩子函数参数
- 钩子函数被赋予了一下参数：
    - el: 指令所绑定的元素，可以用来直接操作DOM；
    - binding: 一个对象，包含以下属性：
        - name:  指令名，不包括`v-`前缀;
        - value: 指令的绑定值，例如：`v-my-directive="1+1"`，value的值为2；
        - oldValue: 指令绑定的前一个值，仅在`update`和`componentUpdated`钩子中可用。无论是否改变都可能；
        - expression: 传给指令的参数。例如：`v-my-directive="1 + 1"`，expression的值为`1 + 1`;
        - arg: 传给指令的参数。例如：`v-my-directive: foo`，arg的值为`foo`；
        - modifiers: 一个包含修饰符的对象。例如：`v-my-directive.foo.bar`，修饰符对象`modifiers`的值是`{foo: true, bar: true}`
    - vnode: Vue编译生成的虚拟节点，可通过查阅VNode API了解;
    - oldVnode: 上一个虚拟节点，仅在`update`和`componentUpdated`钩子中可用；

- PS：除了`el`之外，其它参数都应该是制度的，尽量不要修改他们。如果需要在钩子中共享数据，建议通过元素的`dataset`来进行。

- 一个使用了这些参数的自定义钩子例子：
```
<div id="hook" v-demo:foo.a.b="message"></div>
```
### 函数简写
- 介绍：
    - 大多数情况下，我们可能想在`bind`和`update`钩子上做重复动作，并不关心其它的钩子函数。可以这样写:
    ```
    Vue.directive('my-directive', (el, binding) => {
        el.style.backgroundColor = binding.value
    })
    ```

### 对象字面量
- 介绍： 
    - 如果指令需要多个值，可以传入一个JS对象字面量。指令可以接受所有合法类型的JS表达式。
    ```
    <div v-demo="{color: 'white', text: 'hello!'}"></div>

    Vue.directive('demo', (el, binding) => {
        console.log(binding.value.color)
        console.log(binding.value.text)
    })
    ```











