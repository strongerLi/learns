不同路由指向同一个页面（组件），组件不会重新渲染，可以通过：
```
<router-view :key="activeDate"></router-view>
```
只要key的值发生变化就会重新渲染路由下的组件

- 同一个组件在不同的路由中被引用，并且这个组件中还要进行跳转

优化前
```
this.$router.push({
  path: '/devlops/uplinkedit',
})
// 想要适配
this.$router.push({
  path: '/somePaht/uplinkedit',
})
```
缺点： 这个组件应用到另一个路由时，点击编辑按钮跳转到错的的地址。

优化后
```
this.$router.push({
  path: 'uplinkedit',
})
this.$router.push({
  name: 'uniqeName',
})
```

