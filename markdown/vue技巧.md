### 使用$attrs与$listeners

> $attrs: 当组件在调用时传入的属性没有在props里面定义时，传入的属性将被绑定到
> $attrs属性内（class与style除外，他们会挂载到组件最外层元素上）。
> 并可通过v-bind="$attrs"传入到内部组件中

> $listeners: 当组件被调用时，外部监听的这个组件的所有事件都可以通过$listeners获取到。并可通过v-on="$listeners"传入到内部组件中。


```vue
<!---使用了v-bind与v-on监听属性与事件-->
<template>
    <el-dialog :visible.sync="visibleDialog" v-bind="$attrs" v-on="$listeners">
    <!--其他代码不变-->
    </el-dialog>
</template>
<script>
  export default {
   props: {
      // 对外暴露visible属性，用于显示隐藏弹框
      visible: {
        type: Boolean,
        default: false
      }
    },
    computed: {
      // 通过计算属性，对.sync进行转换，外部也可以直接使用visible.sync
      visibleDialog: {
        get() {
          return this.visible;
        },
        set() {
          this.$emit("update:visible");
        }
      }
    },
    //默认情况下父作用域的不被认作 props 的 attribute 绑定 (attribute bindings) 
    //将会“回退”且作为普通的 HTML attribute 应用在子组件的根元素上。
    //通过设置 inheritAttrs 到 false，这些默认行为将会被去掉
    inheritAttrs: false
 }
</script>

<!---外部使用方式-->
<custom-dialog
  :visible.sync="visibleDialog"
  title="测试弹框"
  @opened="$_handleOpened"
>
  这是一段内容
</custom-dialog>
```
> 对于$attrs，我们也可以使用$props来代替,实现代码如下
```vue
<template>
  <el-dialog :visible.sync="visibleDialog" v-bind="$props" v-on="$listeners">
    <!--其他代码不变-->
  </el-dialog>
</template>
<script>
import { Dialog } from 'element-ui'
export default {
  props: {
    // 将Dialog的props通过扩展运算符展开到props属性里面
    ...Dialog.props
  }
}
</script>
```
> 但上面的代码存在一定的缺陷，有些组件存在非props的属性，比如对于一些封装的表单组件，我们可能需要给组件传入原生属性，但实际原生属性并没有在组件的props上面定义，这时候，如果通过上面的方式去包装组件，那么这些原生组件将无法传递到内部组件里面。

### 
