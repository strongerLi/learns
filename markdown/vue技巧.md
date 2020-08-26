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
          // 其实这里不用$emit,因为子组件已经$emit了，会自动传给父组件的
          // this.$emit("update:visible");
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

### 监听生命周期

> 在Vue组件中，可以用过$on,$once去监听所有的生命周期钩子函数，如监听组件的updated钩子函数可以写成 this.$on('hook:updated', () => {})

```JavaScript
// 监听窗口发生变化，resize组件
window.addEventListener('resize', this.$_handleResizeChart)
// 通过hook监听组件销毁钩子函数，并取消监听事件
this.$once('hook:beforeDestroy', () => {
  window.removeEventListener('resize', this.$_handleResizeChart)
})
```

> 外部监听生命周期

``` vue
<template>
  <!--通过@hook:updated监听组件的updated生命钩子函数-->
  <!--组件的所有生命周期钩子都可以通过@hook:钩子函数名 来监听触发-->
  <custom-select @hook:updated="$_handleSelectUpdated" />
</template>
```

### vue.extend

```vue
<template>
  <transition name="custom-loading-fade">
    <!--loading蒙版-->
    <div v-show="visible" class="custom-loading-mask">
      <!--loading中间的图标-->
      <div class="custom-loading-spinner">
        <i class="custom-spinner-icon"></i>
        <!--loading上面显示的文字-->
        <p class="custom-loading-text">{{ text }}</p>
      </div>
    </div>
  </transition>
</template>
<script>
export default {
  data: {
    text: 'loading...',
    visible: false,
  }
}
</script>
```

```JavaScript
// loading/index.js
import Vue from 'vue'
import LoadingComponent from './loading.vue'

// 通过Vue.extend将组件包装成一个子类
const LoadingConstructor = Vue.extend(LoadingComponent)

let loading = undefined

LoadingConstructor.prototype.close = function() {
  // 如果loading 有引用，则去掉引用
  if (loading) {
    loading = undefined
  }
  // 先将组件隐藏
  this.visible = false
  // 延迟300毫秒，等待loading关闭动画执行完之后销毁组件
  setTimeout(() => {
    // 移除挂载的dom元素
    if (this.$el && this.$el.parentNode) {
      this.$el.parentNode.removeChild(this.$el)
    }
    // 调用组件的$destroy方法进行组件销毁
    this.$destroy()
  }, 300)
}

const Loading = (options = {}) => {
  // 如果组件已渲染，则返回即可
  if (loading) {
    return loading
  }
  // 要挂载的元素
  const parent = document.body
  // 组件属性
  const opts = {
    text: '',
    ...options
  }
  // 通过构造函数初始化组件 相当于 new Vue()
  const instance = new LoadingConstructor({
    el: document.createElement('div'),
    data: opts
  })
  // 将loading元素挂在到parent上面
  parent.appendChild(instance.$el)
  // 显示loading
  Vue.nextTick(() => {
    instance.visible = true
  })
  // 将组件实例赋值给loading
  loading = instance
  return instance
}

export default Loading
```

> 在页面使用

```vue
import Loading from './loading/index.js'
export default {
  created() {
    const loading = Loading({ text: '正在加载。。。' })
    // 三秒钟后关闭
    setTimeout(() => {
      loading.close()
    }, 3000)
  }
}
```

> 做成指令在元素上使用

```vue
import Vue from 'vue'
import LoadingComponent from './loading'
// 使用 Vue.extend构造组件子类
const LoadingContructor = Vue.extend(LoadingComponent)

// 定义一个名为loading的指令
Vue.directive('loading', {
  /**
   * 只调用一次，在指令第一次绑定到元素时调用，可以在这里做一些初始化的设置
   * @param {*} el 指令要绑定的元素
   * @param {*} binding 指令传入的信息，包括 {name:'指令名称', value: '指令绑定的值',arg: '指令参数 v-bind:text 对应 text'}
   */
  bind(el, binding) {
    const instance = new LoadingContructor({
      el: document.createElement('div'),
      data: {}
    })
    el.appendChild(instance.$el)
    el.instance = instance
    Vue.nextTick(() => {
      el.instance.visible = binding.value
    })
  },
  /**
   * 所在组件的 VNode 更新时调用
   * @param {*} el
   * @param {*} binding
   */
  update(el, binding) {
    // 通过对比值的变化判断loading是否显示
    if (binding.oldValue !== binding.value) {
      el.instance.visible = binding.value
    }
  },
  /**
   * 只调用一次，在 指令与元素解绑时调用
   * @param {*} el
   */
  unbind(el) {
    const mask = el.instance.$el
    if (mask.parentNode) {
      mask.parentNode.removeChild(mask)
    }
    el.instance.$destroy()
    el.instance = undefined
  }
})
```
