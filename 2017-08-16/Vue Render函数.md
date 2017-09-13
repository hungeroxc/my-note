# Vue Render函数


### 基础
- 介绍：
    - Vue推荐在大多数情况下使用`template`来创建HTML。但是在一些场景下，还是需要JS的完全编程能力，这时候可以使用`render`函数
- 简单例子：(该例子中，在不适用`slot`属性向组件中传递内容时，例如里面的`Hello World`，这些子元素被存储在组件事例中的`#slots.default`中)
```
    <div id="demo">
        <anchored-heading :level="level">
            Hello World
        </anchored-heading>
    </div>
    <script>
        Vue.component('anchored-heading', {
            render: function(createElement) {
                return createElement(
                    // 构建标签name
                    'h' + this.level,
                    // 子组件中的阵列
                    this.$slots.default
                )
            },
            props: {
                level: {
                    type: Number,
                    required: true
                }
            }
        })
        let vm = new Vue({
            el: '#demo',
            data: {
                level: 1
            }
        })
    </script>
```

### `createElement`参数
- 介绍： 
    - `createElement`函数专门用来生末班
- 参数：
    - HTML标签字符串： 例如：`<div>123</div>`；
    - data.object: 一个包含模板相关属性的数据对象，可以在`template`中使用这些属性，可选参数；
    - 子节点：由`createElement()`构建而成，或者在HTML标签字符串中生成，可选参数
    ```
            render: function(createElement) {
                return createElement(
                    'h' + this.level, [
                        '先写一些文字',
                        createElement('h1', '一则头条')
                    ]
                )
            },
    ```
![](http://upload-images.jianshu.io/upload_images/3360875-5bb98921c1cf51f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 深入`data.object`
    - 介绍： 在模板语法中，`v-bind: class`和`v-bind: style`会被特殊对待，在VNode数据对象中，下列属性名是**级别最高**的字段。该对象也允许你绑定普通的HTML属性，就像DOM属性一样，例如`innerHTML`(这会取代`v-html`指令)。
    ```
    {
        // 和`v-bind:class`一样的API
        `class`: {
            foo: true,
            bar: false
        },
        // 和`v-bind:style`一样的api
        `style`: {
            color: 'red',
            fontSize: '14px'
        },
        // 正常的HTML特性
        `attrs`: {
            id: 'foo'
        },
        // 组件props
        `props`: {
            myProp: 'bar'
        },
        // DOM属性
        `domPrps`: {
            innerHTML: 'baz'
        },
        // 事件监听器基于`on`,所以不再支持例如`v-on:keyup.enter`修饰器，需要手动匹配`keyCode`
        `on`: {
            click: this.clickHandler
        },
        // 仅对于组件，用于监听原生时间，而不是组件内部使用`vm.$emit`触发的事件
        `nativeOn`: {
            click: this.nativeClickHandler
        },
        // 自定义指令，注意事项：不能对绑定的旧值设定新值
        // Vue会自动追踪
        `directives`: [
            {
                name: 'my-custom-directive',
                value: '2',
                expression: '1 + 1',
                arg: 'foo',
                modifiers: {
                    bar: true
                }
            }
        ],
        // 表单插槽作用域
        scopedSlots: {
            default: props => createElement('span', props.text)
        },
        // 如果组件是其他组件的子组件，需要为slot制定名称
        slot: 'name-of-slot',
        key: 'myKey',
        ref: 'myRef'
    }
    ```

- 完整实例
```
    <div id="demo">
        <anchored-heading :level="level">
            hello world
        </anchored-heading>
    </div>
    <script>
        function getChildrenTextContent(children) {
            // 遍历VNode类数组对象，查看是否有子组件，有则递归，没有则返回VNode中的text字符串
            return children.map((node) => {
                return node.children ? getChildrenTextContent(node.children) : node.text
            }).join('')
        }
        Vue.component('anchored-heading', {
            render: function(createElement) {
                let headingId = getChildrenTextContent(this.$slots.default)
                    // 将得到的headingId字符串变为小写
                    .toLowerCase()
                    // 中间添加-
                    .replace(/\W+/g, '-')
                    .replace(/(^\-|-$)/g, '')

                // 创建元素
                return createElement(
                    'h' + this.level, [
                        createElement('a', {
                            attrs: {
                                name: headingId,
                                href: '#' + headingId
                            }
                        }, this.$slots.default)
                    ]
                )
            },
            props: {
                level: {
                    type: Number,
                    required: true
                }
            }
        })
        let vm = new Vue({
            el: '#demo',
            data: {
                level: 1
            }
        })
    </script>
```
![image.png](http://upload-images.jianshu.io/upload_images/3360875-ee38fe7dde968028.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 约束
    - `VNode`必须是唯一不能重复的，所以下面的`render`函数是无效的
    ```
    render: function (createElement) {
        var myParagraphVNode = createElement('p', 'hi')
        return createElement('div', [
            // 错误-重复的VNodes
            myParagraphVNode, myParagraphVNode
        ])
    }
    ```
    - 如果真的需要重复很多次的元素/组件，可以用工厂函数来实现。例如：
    ```
    render: function (createElement) {
        return createElement('div',
            Array.apply(null, { length: 20 }).map(function () {
            return createElement('p', 'hi')
            })
        )
    }
    ```

### 使用JS代替模板功能
- `v-if`和`v-for`:
    - 由于使用原生的JS来实现某些功能很简单，Vue的render函数没有提供专用的API。比如，tenplate中的`v-if`和`v-for`：
    ```
    // 使用了v-if和v-for
    <ul v-if="item.length">
        <li v-for="item in items">{{ item.name }}</li>
    </ul>
    <p v-else>No Item Found</p>
    ```
- 在render函数中：
    - 上述例子都会在render函数中被JS的`if/else`和`map`重写：
    ```
        render: function(createElement) {
            if (this.items.length) {
                return createElement('ul', this.items.map((item) => {
                    return createElement('li', item.name)
                }))
            } else {
                return createElement('p', "no items found.")
            }
        }
    ```
- `v-model`：
    - render函数中没有与`v-model`相对应的api,必须自己实现相应逻辑；
    ```
        render: function (createElement) {
            var self = this
            return createElement('input', {
                domProps: {
                    value: self.value
                },
                on: {
                    input: function (event) {
                        self.value = event.target.value
                        self.$emit('input', event.target.value)
                    }
                }
            })
        }
    ```
    ps: 这需要深入底层，尽管麻烦一些，但对于`v-model`来说，可以更加灵活的控制;

- 时间&修饰符
    - 介绍：
        - 对于`.passive`、`.captrue`、`.once`时间修饰符，Vue提供了响应的前缀可以用于`on`:

![image.png](http://upload-images.jianshu.io/upload_images/3360875-017933e6f1dd3272.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

    - 例子：

    ```
    on: {
        '!click': this.doThisInCapturingMode,
        '~keyup': this.doThisOnce,
        '~!mouseover': this.doThisOnceInCapturingMode
    }
    ```

    - 其他修饰符：
        - 介绍： 对于其他修饰符，前缀不是很重要，因为可以直接在时间处理函数中使用时间方法：
![image.png](http://upload-images.jianshu.io/upload_images/3360875-7eb946461062f062.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
    - 使用了所有修饰符的例子：

    ```
    on: {
        keyup(event){
            // 如果触发时间的元素不是事件绑定的元素
            // 则返回
            if(event.target !== event.currentTarget) return
            // 如果按下去的不是enter键或者
            // 没有同事按下shift键
            // 则返回
            if(!event.shiftkey || ebent.keyCode !== 13) return
            // 阻止事件冒泡
            event.stopPropagation()
            // 阻止钙元素默认的keyup事件
            event.preventDefault()
        }
    }
    ```
- 插槽
    - 介绍:
        - 你可以从`this.$slots`获取VNode列表中的静态内容：
        ```
        render(createElement){
            // <div><slot></slot></div>
            return createElement('div', this.$slots.default)
        }
        ```
        还可以从`this.$scopedSlots`中获取能用作函数的作用域插槽，这个函数返回VNode:
        ```
        render(createElement){
            return createElement('div', [
                this.$scopeSlots.default({
                    text: this.msg
                })
            ])
        }
        ```
        如果要用渲染函数向子组件传递作用域插槽，可以利用VNode数据中的`scopedSlots`域：
        ```
        render(createElement){
            return createElement('div', [
                createElement('child', {
                    scopedSlots: {
                        default(props){
                            return createElement('span', props.text)
                        }
                    }
                })
            ])
        }
        ```

### JSX
- 介绍：
    - 如果你写了很多`render`函数，可能会觉得很痛苦：
    ```
    createElement(
        'conpA', {
            props: {
                level: 1
            }
        }, [
            createElement('span', 'hello'),
            'world'
        ]
    )
    ```
    特别是在模板如此简单的情况下：
    ```
    <component :level="1">
        <span>hello</span>world
    </component>
    ```
    这时候可以使用vue JSX的Babel插件，可以让我们回到更接近于模板的语法上。
    ```
    new Vue({
        el: '#demo',
        render (h) {
            return (
                <AnchoredHeading level={1}>
                    <span>Hello</span> world!
                </AnchoredHeading>
            )
        }
    })
    ```

### 函数式组件
- 介绍：
    - 之前创建的锚点标题组件是比较简单的，没有状态管理或者监听任何传递给他的状态，也没有生命周期方法。只是一个接受参数的函数。
    在这个例子中，我们标记组件为`function`，这意味着它是无状态(没有data)，无实例（没有this上下文）.
    一个函数式组件就像这样：
    ```
    Vue.component('my-component', {
        functional: true,
        // 为了弥补缺少的实例
        // 提供第二个参数作为上下文
        render(createElement, context){
            // ...
        },
        // props可选
        props: {
            ../
        }
    })
    ```
    组件需要的一切都是通过上下文传递，包括：
        - `props`: 提供props的对象
        - `children`: VNode子节点的数组
        - `slots`: slots对象
        - `data`: 传递给组件的data对象
        - `parent`: 对父组件的引用
        - `listenses`: (2.3.0+)一个包含了组件上所注册的`v-on`监听器的对象。这只是一个指向`data.on`的别名
        - `injections`: (2.3.0+)如果使用了`inject`选项，则该对象包含了应当被注入的属性
    在添加`functional: true`之后，锚点标题组件的render函数之间简单更新增加`context`参数，`this.$slots.default`更新为`context.children`,之后`this.level`更新为`context.props.level`.

    因为函数式组件只是一个函数，所以渲染开销也低很多。然而，对持久化实例的缺乏也意味着函数式组件不会出现在`Vue.devtools`的组件树种。

    在作为包装组件时他们也同样非常有用，比如，当需要做这些时：
        - 程序化地在多个组件中选择一个；
        - 在将`children`,`props`,`data`传递给子组件之前操作它们；
    
    下面是一个依赖传入props的值得`smart-list`组件例子，他可以代表更多具体的组件：
    ```
        var EmptyList = { /* ... */ }
        var TableList = { /* ... */ }
        var OrderedList = { /* ... */ }
        var UnorderedList = { /* ... */ }

        Vue.component('smart-list', {
            functional: true,
            render(createElement, context){
                function appropriateListComponent(){
                    var items = Context.props.items

                    if(items.length === 0) return EmptyList
                    if(typeof items[0] === 'object') return TableList
                    if(context.props.isOrdered) return OrderedList

                    return UnorderedList
                } 

                return createElement(
                    appropriateListComponent(),
                    context.data,
                    context.children
                )
            },
            props: {
                items: {
                    type: Array,
                    required: true
                }
            }
        })
    ```

- `slots()`和`children`对比
    - 介绍：
        - 你可能想知道为什么同时需要`slots()`和`children`.`slots().default`不是和`children`类似的吗？在一些场景中，是这样，但是如果是函数式组件和下面这样的children呢？
        ```
            <my-functional-component>
                <p slot="foo">
                    first
                </p>
                <p>second</p>
            </my-functional-component>
        ```
        对于这个组件，`children`会给你两个标签段落，而`slots().default`只会传递第二个匿名段落标签，`slots().foo`会传递第一个具名段落标签。同事拥有`children`和`slots()`，因此可以选择让组件通过`slot()`系统分发或者简单地通过`children`接受，让其他组件去处理.

### 模板编译
- Vue的模板实际上是编译成了render函数。这是一个实现细节，但是非常有趣。下面是一个使用`vue.compile`来实现编译模板字符串的简单demo：

```
<div>
    <header>
        <h1>I'm a template!</h1>
    </header>
    <p v-if="message">
        {{ message }}
    </p>
    <p v-else>
        No message.
    </p>
</div>   
```

render: 
```
function anonymous(){
    with(this){
        return _c('div', [
            _m(0),
            (message) 
            ? 
            _c('p', [
                _v(_s(message)) 
            ])
            :
            _c('p', [_v("No Message.")])
        ])
    }
}
```

staticRenderFns:
```
_m(0): function anonymous(){
    with(this){
        return _c('header', [
            _c('h1', [_v("I'm a template!")])
        ])
    }
}
```