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
