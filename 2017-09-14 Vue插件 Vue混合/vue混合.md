# 混合

### 基础
- 介绍：
    - 混合(mixins)是一种分发Vue组件中可复用功能的非常灵活的方式。混合对象可以包含任意组件选项。以组件使用混合对象时，所有混合对象的选项将被混入该组件本身的选项.
    - 例子：
    ```
        let myMixin = {
            created() {
                this.hello()
            },
            methods: {
                hello() {
                    console.log('hello')
                }
            }
        }
        let Component = Vue.extend({
            mixins: [myMixin]
        })
        let component = new Component()
    ```

### 选项合并
- 介绍：
    - 当组件和混合对象含有同名属性时，这些选项将以恰当的方式混合。比如，同名函数钩子函数将混合为一个数组，然后都会被调用。另外，混合对象的钩子将在组件自身钩子**之前**调用:
    ```
        let myMixin = {
            created() {
                console.log('混合钩子被调用了')
            }
        }

        new Vue({
            mixins: [myMixin],
            created() {
                console.log('组件钩子被调用')
            }
        })
    ```
![image.png](http://upload-images.jianshu.io/upload_images/3360875-c7fffda3b1e605f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

值为对象的选项，例如`methods`,`components`,`directives`, 将被混合为同一个对象。两个对象键名冲突时，取**组件对象**的键值对。
```
        let mixin = {
            methods: {
                foo() {
                    console.log('foo')
                },
                bar() {
                    console.log('bar')
                }
            }
        }

        let vm = new Vue({
            mixins: [mixin],
            methods: {
                bar() {
                    console.log('组件bar')
                },
                baz() {
                    console.log('baz')
                }
            }
        })

        vm.foo()
        vm.bar()
        vm.baz()
```
![image.png](http://upload-images.jianshu.io/upload_images/3360875-a3595ecff6b9cdf9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

PS: `Vue.extend()`也是用同样的策略进行合并.


### 全局混合
- 介绍： 
    - 也可以全局祖册混合对象。注意：一旦使用全局混合对象，将会影响到所有之后创建的Vue实例。使用恰当时，可以为自定义对象注入处理逻辑。
    ```
        Vue.mixin({
            created() {
                let myOptions = this.$options.myOptions
                if (myOptions) {
                    console.log(myOptions)
                }
            }
        })

        new Vue({
            myOptions: 'hello!'
        })
    ```
![image.png](http://upload-images.jianshu.io/upload_images/3360875-a01afdb9f97ec9da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

ps: 谨慎使用全局混合对象， 因为会影响到每个单独创建的Vue实例（包括第三方模板）。大多数情况下，只应当应用于自定义选项，就像上面示例那样。也可以将其用作`Plugins`以避免产生重复应用.


### 自定义选项合并策略
- 介绍： 
    - 自定义选项将使用默认策略，即简单地覆盖已有值。如果想让自定义选项以自定义逻辑合并，可以向`Vue.config.optionMergeStrategies`添加一个函数：
    ```
    Vue.config.optionMergeStrategies.myOption = function(toVal, formVal){
        // return mergedVal
    }
    ```
    - 对于大多数对象选项，可以使用`methods`的合并策略：
    ```
    var strategies = Vue.config.optionMergeStrategies
    strategies.myOption = strategies.methods
    ```