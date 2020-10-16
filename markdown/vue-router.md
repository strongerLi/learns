不同路由指向同一个页面（组件），组件不会重新渲染，可以通过：
```
<router-view :key="activeDate"></router-view>
```
只要key的值发生变化就会重新渲染路由下的组件
